# Setup Instructions

## Getting Started

### Prerequisites
- Modern web browser (Chrome, Firefox, Safari, Edge)
- Anthropic API key (from https://console.anthropic.com)
- Google Doc with beats/prompts (optional but recommended)

## Installation

### Option A: Use GitHub Pages (Recommended)
No installation needed! Just visit:
```
https://ayatziv.github.io/peter-playgound/
```

### Option B: Local File
1. Download or clone this repository
2. Open `peter_mind_playground_V2.html` in your browser
3. Add your configuration (see below)

### Option C: Local Server (if you want server features)
```bash
# Using Python 3
python -m http.server 8000

# Using Node.js (with http-server)
npx http-server

# Using PHP
php -S localhost:8000
```

Then navigate to `http://localhost:8000`

## Configuration

### Step 1: Add Your Anthropic API Key
1. Open the tool in your browser
2. Click **Configuration** in the sidebar
3. Paste your Claude API key in the **API Key** field
4. The key is stored in browser localStorage (not sent to any server except Anthropic)

**Get your key**:
- Visit https://console.anthropic.com/keys
- Create a new API key
- Copy and paste into the tool

### Step 2: Connect Your Google Doc (Optional)
1. In **Configuration**, enter your **Google Doc ID**
2. Find your Doc ID in the URL: `docs.google.com/document/d/{DOC_ID}/edit`
3. Make sure the Apps Script has access to your document

#### Create a New Google Doc
1. Go to https://docs.google.com
2. Create a new blank document
3. Paste the Doc ID into the tool
4. Add beats using the format below

### Step 3: Set Up Beats in Your Google Doc

Structure your document with beats (scenes/prompts):

```
@@room-opening
[Your first beat description goes here]

@@interrogation
[Description for the interrogation scene]

@@resolution
[Final scene description]

@@@@fixed_prompt
[This prompt applies to all beats]

%%keypoints
Points that should be highlighted
but not sent to Claude
%%

$$START_MESSAGE[Opening message to send automatically]$$
```

### Step 4: Select Model & Temperature
- **Model**: Claude 3.5 Sonnet (default), or choose another variant
- **Temperature**: 0.0 (deterministic) → 1.0 (creative)
- Adjust based on your use case

## Workflow

1. **Load Beat**: Select a beat from the dropdown
2. **Review Prompts**: Check Fixed + Dynamic prompts
3. **Check Key Points**: Read the gold panel above chat
4. **Send Message**: Type in the input box and press Enter
5. **Switch Characters**: Use tabs to switch between Peter, Monkey, Human

## Troubleshooting

### "Failed to load beats from Google Doc"
- [ ] Check Doc ID is correct
- [ ] Ensure Google Doc is readable (not in Trash)
- [ ] Try refreshing the page
- [ ] Check browser console for errors (F12)

### "API Error: 401 Unauthorized"
- [ ] Verify your API key is correct
- [ ] Check key hasn't expired
- [ ] Ensure key starts with `sk-ant-`

### CORS Error in Browser Console
- [ ] This is expected for direct fetch requests
- [ ] Tool uses JSONP to bypass CORS
- [ ] If JSONP fails, check Google Apps Script URL

### localStorage Issues
- [ ] Clear browser cache (Ctrl+Shift+Delete)
- [ ] Check if localStorage is enabled
- [ ] Try a different browser or private window

## Advanced Setup

### Custom Google Apps Script
If you want to use your own Google Apps Script:

1. Create a new Apps Script project
2. Use the provided script code (see `GOOGLE_SCRIPT.md`)
3. Deploy as web app
4. Copy deployment URL
5. Paste into **Google Apps Script URL** field

### Using with Wix
The tool works great embedded in Wix via iframe. Just:
1. Add **Custom Code** section
2. Paste an iframe pointing to GitHub Pages URL:
   ```html
   <iframe src="https://ayatziv.github.io/peter-playgound/" 
           width="100%" height="1200" frameborder="0"></iframe>
   ```

## Browser Storage

The tool stores configuration in browser localStorage:
- API Key
- Google Doc ID
- Custom Google Apps Script URL
- Last selected beat
- UI preferences

**Clear storage**: Open DevTools (F12) → Storage → Local Storage → Delete

## Security Considerations

### API Key Storage
Currently, API keys are stored in browser localStorage. This is suitable for:
- ✅ Personal use
- ✅ Testing & development
- ❌ Multi-user environments
- ❌ Sensitive production systems

For production, consider:
- Using a backend proxy server
- Environment variables
- API key rotation
- Rate limiting

### Google Docs Access
- Ensure only authorized users can edit your beat document
- Use Google Docs sharing settings
- Regenerate Apps Script deployment if compromised

## Performance Tips

1. **Large Google Docs**: Trim to only active beats
2. **Multiple Beats**: Use descriptive names for quick navigation
3. **Long Conversations**: Periodically download transcript and start fresh
4. **API Rate Limits**: Monitor usage on Anthropic console

## Getting Help

- Check browser console (F12 → Console tab)
- Review `ARCHITECTURE.md` for technical details
- Contact: ayatziv@gmail.com

---

**Last Updated**: 2026-09-21
