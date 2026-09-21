# Google Apps Script Integration

## Overview

The Peter Mind Playground uses Google Apps Script to read beats and prompts from a Google Doc. The script:
- Parses special tags (`@@`, `%%`, `$$`)
- Returns formatted JSON via JSONP
- Handles CORS restrictions by using JSONP callback

## Current Deployment

**Script URL**: 
```
https://script.google.com/macros/s/AKfycbwBdtP2kx-TnqVNIzh2lqNf_lxrbAHD1Xm5jI4muQlkDb6Isx-x7o_pzPIqdsdq3VgOUg/exec
```

**Source Document**: `1Z8BMGPUaJJoz05omBibE5Rh0VXcnZpiWOlb-n5cAx5A`

## Script Code

```javascript
// Google Apps Script to load beats from Google Doc
// Deploy as web app with "Execute as" = your email, "Who has access" = Anyone

function doGet(e) {
  const docId = e.parameter.docId;
  const callback = e.parameter.callback || 'callback';
  
  if (!docId) {
    return buildJsonResponse(callback, { error: "Missing docId parameter" });
  }
  
  try {
    const doc = DocumentApp.openById(docId);
    const beats = parseBeats(doc);
    return buildJsonResponse(callback, beats);
  } catch (error) {
    return buildJsonResponse(callback, { 
      error: error.toString() 
    });
  }
}

function parseBeats(doc) {
  const body = doc.getBody();
  const tables = doc.getTables();
  const beats = {};
  let fixedPrompt = '';
  let currentBeat = null;
  
  // If using tables (each table = beat with columns: Name, Content, etc.)
  if (tables.length > 0) {
    for (let i = 0; i < tables.length; i++) {
      const table = tables[i];
      const numRows = table.getNumRows();
      
      for (let r = 0; r < numRows; r++) {
        const row = table.getRow(r);
        const cell0 = row.getCell(0).getText().trim();
        const cell1 = row.getCell(1).getText().trim();
        
        if (cell0.startsWith('@@')) {
          const beatName = cell0.replace(/^@@+/, '').trim();
          beats[beatName] = cell1;
        }
      }
    }
  }
  
  // Fallback: parse from text body (simpler approach)
  // Look for lines starting with @@ as beat markers
  const paragraphs = [];
  for (let i = 0; i < body.getNumChildren(); i++) {
    const elem = body.getChild(i);
    if (elem.getType() === DocumentApp.ElementType.PARAGRAPH) {
      paragraphs.push(elem.getText());
    }
  }
  
  let content = '';
  for (let i = 0; i < paragraphs.length; i++) {
    const para = paragraphs[i];
    
    if (para.startsWith('@@')) {
      // Save previous beat
      if (currentBeat) {
        beats[currentBeat] = content.trim();
      }
      
      // Start new beat
      currentBeat = para.replace(/^@@+/, '').trim();
      content = '';
      
      if (currentBeat === 'fixed_prompt') {
        fixedPrompt = '';
      }
    } else if (currentBeat === 'fixed_prompt') {
      fixedPrompt += para + '\n';
    } else if (currentBeat) {
      content += para + '\n';
    }
  }
  
  // Save final beat
  if (currentBeat) {
    beats[currentBeat] = content.trim();
  }
  
  return {
    beats: beats,
    fixedPrompt: fixedPrompt.trim(),
    lastUpdated: new Date().toISOString()
  };
}

function buildJsonResponse(callback, data) {
  const json = JSON.stringify(data);
  return HtmlService.createHtmlOutput(
    callback + '(' + json + ');'
  ).setMimeType(HtmlService.MimeType.JAVASCRIPT);
}
```

## Setup Instructions

### 1. Create a New Apps Script Project
1. Go to https://script.google.com
2. Click **+ New project**
3. Name it: "Peter Mind Beats Loader"

### 2. Copy Script Code
Replace the default `Code.gs` content with the script above.

### 3. Create Script Properties (Optional)
For production, store the Doc ID in Script Properties:

```javascript
function setDocIdProperty(docId) {
  const scriptProperties = PropertiesService.getScriptProperties();
  scriptProperties.setProperty('docId', docId);
}

function getDocIdProperty() {
  const scriptProperties = PropertiesService.getScriptProperties();
  return scriptProperties.getProperty('docId');
}
```

### 4. Deploy as Web App
1. Click **Deploy** → **New deployment**
2. Select type: **Web app**
3. Configure:
   - **Execute as**: Your email
   - **Who has access**: Anyone
4. Click **Deploy**
5. Copy the deployment URL
6. Paste into Peter Mind: **Configuration** → **Google Apps Script URL**

## Beat Format in Google Doc

### Basic Format
```
@@beat-name-one
Content for this beat goes here.
Multiple lines are supported.

@@beat-name-two
Content for second beat.

@@fixed_prompt
This prompt is shared across all beats.
```

### Special Tags

#### Beat Definition
```
@@room-opening
This is the opening scene description.
```
- Use `@@` prefix for any beat name
- Content continues until next `@@` tag
- Beat names appear in the dropdown

#### Fixed Prompt
```
@@fixed_prompt
You are a helpful assistant.
Always stay in character.
```
- Single `@@fixed_prompt` section per doc
- Contents shared with all beats
- Loads into "Fixed Prompt" textarea

#### Key Points
```
%%keypoints
- Never break character
- Focus on subtext
- Pause for reactions
%%
```
- Content between `%%` markers
- Displayed in gold panel above chat
- NOT sent to Claude
- Useful for reminders, stage directions

#### Start Message
```
$$START_MESSAGE[Hello, how are you today?]$$
```
- Auto-sends when beat is selected
- Great for scene initialization
- Optional per beat

#### Skip Section
```
@@END_dont_delete
This section is skipped by the parser.
Safe place to add notes or archives.
```

## Example Complete Document

```
@@@@fixed_prompt
You are Peter, a thoughtful and introspective character.
You remember previous conversations in this session.
Respond naturally and authentically.

%%keypoints
- Peter has been through a lot recently
- He's cautious but ultimately kind
- Current mood: contemplative
%%

@@scene-one-cafe
Peter is sitting in a quiet cafe, nursing a cold coffee.

$$START_MESSAGE[The coffee's gone cold. I didn't notice. How long have I been sitting here?]$$

@@scene-two-confrontation
An old friend walks through the door.

$$START_MESSAGE[I wasn't expecting to see them today.]$$

@@scene-three-resolution
The conversation reaches its natural conclusion.

%%keypoints
- Peter feels relieved
- There's hope for reconciliation
%%

@@END_dont_delete
V1 Scenes (archived)
- original-opening
- extended-flashback
```

## Why JSONP?

Wix (and some other platforms) block direct `fetch()` requests due to CORS. JSONP bypasses this by:
1. Creating a `<script>` tag dynamically
2. Pointing to the Apps Script URL
3. Apps Script returns `callback(data);`
4. Browser executes the callback immediately

The Peter Mind tool detects JSONP capability and uses it automatically.

## Troubleshooting

### "Failed to fetch beats"
- [ ] Check Doc ID is correct
- [ ] Verify document is shared/readable
- [ ] Confirm Apps Script deployment URL
- [ ] Check browser console for errors

### "Script error: Missing docId parameter"
- [ ] Ensure Google Doc ID is entered
- [ ] Check ID has no extra spaces
- [ ] ID should be 44 characters long

### "Permission denied"
- [ ] Open doc in Google Drive
- [ ] Click Share
- [ ] Set to "Anyone with the link can view"
- [ ] Or invite the Apps Script service account

### "Deployment not responding"
- [ ] Try redeploying the script
- [ ] Click **Deploy** → **Manage deployments**
- [ ] Delete old deployment
- [ ] Create new deployment

## Customization

### Parse Google Sheets Instead
To load beats from Google Sheets:

```javascript
function parseBeats(sheetId) {
  const sheet = SpreadsheetApp.openById(sheetId);
  const range = sheet.getActiveSheet().getDataRange();
  const values = range.getValues();
  
  const beats = {};
  for (let i = 0; i < values.length; i++) {
    const beatName = values[i][0];
    const content = values[i][1];
    if (beatName && beatName.startsWith('@@')) {
      beats[beatName.replace('@@', '')] = content;
    }
  }
  return { beats: beats };
}
```

### Add Metadata
Extend the response to include timestamps, authors, tags:

```javascript
return {
  beats: beats,
  fixedPrompt: fixedPrompt,
  metadata: {
    docTitle: doc.getName(),
    lastModified: doc.getLastUpdated(),
    authorEmail: Session.getActiveUser().getEmail()
  }
};
```

## Security

### Deployment Access Control
- **"Anyone"** (current): Public access, anyone can fetch beats
- **"Specific people"**: Restrict to certain users
- For public Google Docs, "Anyone" is fine

### API Rate Limiting
Google Apps Script has rate limits:
- 30 requests per minute per user
- 100 requests per minute per project
- For heavy use, consider backend caching

### Regenerate Deployment
If URL is compromised:
1. Go to script.google.com
2. Open the project
3. Click **Deploy** → **Manage deployments**
4. Delete the old deployment
5. Create a new deployment with new URL

## Testing

Test the script directly in Apps Script:
1. Click **Run** next to `doGet`
2. Check **Execution log** for output
3. View **Logs** for errors

Or test in browser:
```
https://script.google.com/macros/s/[YOUR_DEPLOYMENT_ID]/exec?docId=1Z8BMGPUaJJoz05omBibE5Rh0VXcnZpiWOlb-n5cAx5A
```

Replace with your deployment ID and Google Doc ID.

---

**Last Updated**: 2026-09-21
