# Peter Mind Playground

A single-page HTML/JS acting tool powered by Anthropic's API, with Google Doc script loader and three character tabs for interactive role-play and character development.

## 🎭 Overview

Peter Mind Playground is a web-based tool for actors, writers, and character developers to explore dialogue and character interactions through AI-assisted conversations. It features three distinct characters:

- **Peter** — A stateful character that maintains conversation history
- **Monkey Puppet** — A one-shot character for quick interactions
- **Human Puppet** — A one-shot character for human-like responses

## 📁 Project Structure

```
project-peter-mind/
├── index.html                          # Main entry point (GitHub Pages)
├── peter_mind_playground_V2.html       # Most up-to-date source
├── README.md                           # This file
├── .gitignore
└── docs/
    ├── SETUP.md                        # Setup instructions
    ├── GOOGLE_SCRIPT.md                # Google Apps Script documentation
    └── ARCHITECTURE.md                 # Technical architecture
```

## 🚀 Quick Start

### Option 1: GitHub Pages (Live)
Visit: https://ayatziv.github.io/peter-playgound/

### Option 2: Local Development
1. Clone this repository
2. Open `peter_mind_playground_V2.html` in your browser
3. Add your Anthropic API key in the Configuration panel
4. Load a beat from your Google Doc
5. Start acting!

## ⚙️ Configuration

### Required
- **Anthropic API Key**: Your Claude API key (embedded in the tool)
- **Google Doc ID**: Document containing your beats/prompts

### Optional
- **Apps Script URL**: Custom Google Apps Script deployment URL
- **Model Selection**: Choose Claude model variant
- **Temperature**: Adjust AI creativity (0.0–1.0)

## 📝 Google Doc Format

Structure your Google Doc with beats and prompts:

```
@@room-opening
Your opening beat description here

@@interrogation
Details for the interrogation scene

@@@@fixed_prompt
System prompt shared across all beats

%%keypoints
Key points shown in gold panel
%%

$$START_MESSAGE[Your opening message]$$
```

### Special Tags
- `@@tagname` — Defines a beat (tab title in UI)
- `@@fixed_prompt` — Loads into Fixed Prompt textarea
- `%%keypoints ... %%` — Extracted to Key Points panel (not sent to AI)
- `$$START_MESSAGE[...]$$` — Auto-fires when beat selected
- `@@END_dont_delete` — Skipped by parser

## 🔑 Features

### UI Components
- **Sidebar**: Collapsible configuration and character tabs
- **Beat Selector**: Dropdown populated from `@@` tags in Google Doc
- **Prompts**: Fixed + Dynamic textareas with resizable divider
- **Key Points Panel**: Gold-colored strip showing extracted keypoints
- **Character Tabs**: Peter, Monkey Puppet, Human Puppet
- **Event Log**: Full conversation transcript with download option
- **System Prompt Preview**: Collapsible view of combined prompts

### Smart Features
- Auto-send opening messages when beat changes
- Key points extraction (lines won't be sent to API)
- Combined system prompt generation
- Event log → transcript download (no timestamps)
- Responsive design for various screen sizes

## 🌐 Hosting

### GitHub Pages
- Repository: https://github.com/ayatziv/peter-playgound
- Hosted URL: https://ayatziv.github.io/peter-playgound/
- To update: Replace `index.html` with latest `peter_mind_playground_V2.html`

### Why GitHub Pages?
Wix iframe sandboxes block `fetch()` and JSONP `<script>` tags to external domains. GitHub Pages has no such restrictions.

## 📜 Google Apps Script

Current Apps Script URL:
```
https://script.google.com/macros/s/AKfycbwBdtP2kx-TnqVNIzh2lqNf_lxrbAHD1Xm5jI4muQlkDb6Isx-x7o_pzPIqdsdq3VgOUg/exec
```

**Associated Google Doc**: `1Z8BMGPUaJJoz05omBibE5Rh0VXcnZpiWOlb-n5cAx5A`

The Apps Script:
- Reads from Google Doc tabs
- Parses `@@` tags for beats
- Returns formatted JSON via JSONP
- Prepends tab title + newline to each beat body

## 🔐 Security Notes

- **API Key**: Currently embedded in HTML (for rapid prototyping)
- **CORS**: Bypassed via JSONP for Google Docs integration
- **Local Storage**: Used for UI state persistence
- **Production**: Consider backend API proxy for key management

## 🛠️ Development

### Tech Stack
- **Frontend**: HTML5, Vanilla JavaScript, CSS3
- **API**: Anthropic Claude API
- **Data Source**: Google Docs + Apps Script (JSONP)
- **Hosting**: GitHub Pages

### Key JavaScript Objects
- `config` — User settings (API key, Doc ID, etc.)
- `characters` — Peter, Monkey Puppet, Human Puppet definitions
- `currentBeat` — Active scene data
- `eventLog` — Transcript history
- `systemPrompt` — Combined prompt generation

### Customization
Edit character system prompts, adjust UI layout, or add new character types directly in the HTML file.

## 📚 Files Reference

| File | Purpose | Status |
|------|---------|--------|
| `peter_mind_playground_V2.html` | Latest source (canonical) | ✅ Active |
| `index.html` | GitHub Pages copy | ✅ Active |
| `peter_mind_playground_V1.html` | Old snapshot | ⚠️ Archived |

## 🤝 Contributing

This is a personal playground tool. To modify:
1. Edit `peter_mind_playground_V2.html`
2. Test locally
3. Commit to git
4. Update `index.html` for GitHub Pages
5. Push to GitHub

## 📄 License

Personal project. Use at your own discretion.

---

**Last Updated**: 2026-09-21  
**Author**: Amir Yatziv  
**Contact**: ayatziv@gmail.com
