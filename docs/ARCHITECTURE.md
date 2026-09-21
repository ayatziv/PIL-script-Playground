# Architecture Overview

## System Design

```
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (Client)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Peter Mind Playground UI                    │   │
│  │  (Single HTML file: peter_mind_playground_V2.html)      │   │
│  │                                                          │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │ Sidebar (Config & Character Tabs)               │    │   │
│  │  │  - Configuration panel                          │    │   │
│  │  │  - API Key, Doc ID inputs                       │    │   │
│  │  │  - Character selection (Peter/Monkey/Human)     │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │ Main Content Area                                │    │   │
│  │  │  - Beat selector dropdown                        │    │   │
│  │  │  - Key Points panel (gold, dismissible)          │    │   │
│  │  │  - Fixed Prompt textarea                         │    │   │
│  │  │  - Dynamic Prompt textarea                       │    │   │
│  │  │  - Combined System Prompt preview                │    │   │
│  │  │  - Chat interface (messages + input)             │    │   │
│  │  │  - Event log / transcript                        │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
         │                                      │
         │                                      │
         ▼                                      ▼
    ┌─────────────────────┐          ┌──────────────────────┐
    │  Google Docs        │          │  Anthropic API       │
    │  + Apps Script      │          │  (Claude)            │
    │  (JSONP)            │          │                      │
    └─────────────────────┘          └──────────────────────┘
```

## Components

### 1. UI Layer
**File**: `peter_mind_playground_V2.html` (all-in-one)

**Sections**:
- **Header**: Logo, title, minimal controls
- **Sidebar**: Configuration and character tabs
- **Main Area**: Beats, prompts, chat, and logs

**Technologies**:
- HTML5 semantic markup
- CSS3 Grid & Flexbox for layout
- Vanilla JavaScript (no frameworks)
- Responsive design (desktop/tablet/mobile)

### 2. Data Management

#### Local Storage
Persists user preferences:
```javascript
{
  apiKey: "sk-ant-...",
  docId: "1Z8BMGP...",
  appsScriptUrl: "https://...",
  selectedBeat: "scene-opening",
  systemPrompt: "...",
  temperature: 0.8,
  selectedModel: "claude-3-5-sonnet-20241022"
}
```

#### State Objects
- `config` — User settings (API key, Doc ID, model, temperature)
- `characters` — Three character definitions (Peter, Monkey, Human)
- `currentBeat` — Active beat data {name, content, fixedPrompt, keyPoints, startMessage}
- `eventLog` — Array of all messages in session
- `systemPrompt` — Generated from beat + fixed prompt

### 3. Google Docs Integration

**JSONP Flow**:
1. User selects a beat from dropdown
2. JavaScript creates `<script>` tag dynamically
3. Script points to Google Apps Script URL with parameters:
   - `docId`: Google Doc ID
   - `callback`: JavaScript callback function name
4. Apps Script:
   - Opens Google Doc
   - Parses `@@`, `%%`, `$$` tags
   - Returns JSON wrapped in callback
5. Callback function executes, updates UI with new beat

**Why JSONP**:
- Bypasses CORS restrictions
- Works in Wix iframe sandboxes
- No backend proxy needed
- Direct browser-to-Google integration

**Parsing Logic**:
```javascript
// Pseudo-code
forEach line in GoogleDoc {
  if (line.startsWith('@@')) {
    beatName = line.replace('@@', '')
    startNewBeat()
  } else if (line.startsWith('%%')) {
    extractKeyPoints()
  } else if (line.startsWith('$$START_MESSAGE')) {
    extractStartMessage()
  } else {
    appendToCurrentBeat()
  }
}
```

### 4. Character System

Three character profiles:

#### Peter (Stateful)
```javascript
{
  name: "Peter",
  description: "A thoughtful, introspective character...",
  systemPrompt: "You are Peter. [full prompt]",
  keepMemory: true  // Maintains conversation history
}
```

#### Monkey Puppet (One-shot)
```javascript
{
  name: "Monkey Puppet",
  description: "A playful, irreverent character...",
  systemPrompt: "You are Monkey Puppet. [full prompt]",
  keepMemory: false  // Fresh start each message
}
```

#### Human Puppet (One-shot)
```javascript
{
  name: "Human Puppet",
  description: "A realistic, grounded character...",
  systemPrompt: "You are Human Puppet. [full prompt]",
  keepMemory: false  // Fresh start each message
}
```

**Implementation**:
- Character selection via tab click
- System prompt updated on switch
- Message history reset for one-shot characters
- Peter maintains full conversation thread

### 5. Claude API Integration

**Endpoint**: `https://api.anthropic.com/v1/messages`

**Request Flow**:
```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "x-api-key": apiKey,
    "anthropic-version": "2023-06-01",
    "content-type": "application/json"
  },
  body: JSON.stringify({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 1024,
    system: systemPrompt,  // Combined: beat + fixed
    messages: [
      { role: "user", content: userMessage },
      { role: "assistant", content: previousResponse },
      // ... history ...
    ],
    temperature: 0.8
  })
});
```

**Message History**:
- **Peter**: Keeps all messages in session
- **Monkey/Human**: Only current exchange

**Token Counting**:
- Approximate: ~4 chars = 1 token
- Monitor usage in Anthropic console

### 6. UI Interactions

#### Beat Selection
```javascript
beatSelect.addEventListener("change", async (e) => {
  currentBeat = beats[e.target.value];
  updatePrompts();
  displayKeyPoints();
  if (currentBeat.startMessage) {
    autoSendMessage(currentBeat.startMessage);
  }
});
```

#### Message Sending
```javascript
sendButton.addEventListener("click", async () => {
  const message = inputBox.value;
  addToLog(message, "user");
  
  const response = await fetchClaude({
    system: systemPrompt,
    messages: eventLog.map(m => ({
      role: m.role,
      content: m.text
    }))
  });
  
  addToLog(response.content[0].text, "assistant");
  inputBox.value = "";
});
```

#### Responsive Divider
- Resizable divider between Fixed and Dynamic prompts
- Mouse events: down → move → up
- Updates textarea heights dynamically

#### Transcript Download
```javascript
downloadButton.addEventListener("click", () => {
  const transcript = eventLog.map(m => 
    `[${m.role}] ${m.text}`
  ).join("\n\n");
  
  downloadAsFile(transcript, "transcript.txt");
});
```

## Data Flow

### Initial Load
```
Page Load
  ↓
Load from localStorage
  ↓
Initialize UI
  ↓
User selects beat
  ↓
Fetch from Google Docs (JSONP)
  ↓
Parse response
  ↓
Update prompts + key points
  ↓
Auto-send start message (optional)
```

### Message Cycle
```
User types message
  ↓
Click "Send" or press Enter
  ↓
Append to event log
  ↓
Display in chat
  ↓
Prepare system prompt
  ↓
Send to Claude API
  ↓
Receive response
  ↓
Append to event log
  ↓
Display response
  ↓
Wait for next input
```

### Character Switch
```
User clicks character tab
  ↓
Load character system prompt
  ↓
Update UI display
  ↓
Clear history (if one-shot)
  ↓
Keep history (if Peter)
  ↓
Ready for new message
```

## CSS Architecture

### Color Scheme
```css
:root {
  --bg: #0a0a0f;              /* Dark background */
  --surface: #111118;          /* UI elements */
  --surface2: #1a1a24;         /* Elevated surfaces */
  --border: #2a2a3a;           /* Dividers */
  --accent: #e8c547;           /* Gold (key points) */
  --accent2: #7b5ea7;          /* Purple (secondary) */
  --accent3: #4ade80;          /* Green (accent) */
  --text: #e8e8f0;             /* Primary text */
  --text-muted: #6b6b85;       /* Secondary text */
  --user-bg: #1e1e2e;          /* User messages */
  --agent-bg: #151520;         /* Agent messages */
}
```

### Layout Grid
```css
body {
  display: grid;
  grid-template-columns: 380px 4px 1fr;  /* sidebar | divider | content */
  grid-template-rows: 56px 1fr;          /* header | body */
  height: 100vh;
}
```

### Typography
- **Heading Font**: Syne (600, 800 weight)
- **Body Font**: Space Mono (400, 700 weight)
- **Sizes**: Scaled from base 14px

## Performance Considerations

### Optimizations
1. **Lazy Loading**: Key Points panel hidden by default
2. **Event Delegation**: Single listener for multiple elements
3. **Throttled Resizing**: Divider drag uses requestAnimationFrame
4. **Efficient DOM**: Minimal reflow/repaint on updates
5. **localStorage Caching**: Reduces API calls

### Bottlenecks
1. **Google Docs Load**: JSONP parsing can be slow for large docs
2. **API Response Time**: Claude typically 2-5 seconds
3. **Message History**: Peter's history grows with session length

### Limits
- **Max tokens per request**: 4096 (configurable)
- **Max conversation history**: Browser memory (typically unlimited)
- **Concurrent requests**: One at a time (sequential)

## Security

### API Key Handling
- **Storage**: Browser localStorage (not backend)
- **Transmission**: Only to Anthropic servers (HTTPS)
- **Risk**: Exposed if developer tools accessed
- **Mitigation**: Consider backend proxy for production

### Google Doc Access
- **Permission Model**: Inherited from Google Docs sharing
- **JSONP**: Script tag loads publicly
- **Authentication**: Apps Script executes with script owner's auth

### CORS
- **Current**: JSONP bypasses CORS
- **Alternative**: Backend proxy could enforce CORS
- **Trade-off**: Simplicity vs. security

## Browser Compatibility

### Supported
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

### Required APIs
- `fetch()` — For Claude API
- `localStorage` — For config persistence
- `requestAnimationFrame` — For smooth interactions
- `Blob` + `URL.createObjectURL()` — For downloads

### Fallbacks
- No graceful fallback for missing APIs
- Requires modern browser

## Deployment

### GitHub Pages
1. Push changes to `main` branch
2. `index.html` auto-deploys to https://ayatziv.github.io/peter-playgound/
3. No build step required
4. Global CDN (fast delivery)

### Local File
1. Save `.html` locally
2. Open in browser (file:// protocol)
3. Limited functionality (some CORS restrictions)

### Behind Firewall
1. Copy `.html` to internal server
2. Serve via HTTP/HTTPS
3. Use backend API proxy if needed

## Future Enhancements

- [ ] Backend proxy for API key security
- [ ] Database storage for conversation history
- [ ] Authentication system
- [ ] Multi-user sessions
- [ ] Export to Markdown/PDF
- [ ] Real-time collaboration
- [ ] Plugin system for custom characters
- [ ] Analytics/usage tracking

---

**Last Updated**: 2026-09-21
