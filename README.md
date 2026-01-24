# User Storage - Open WebUI Tool

**Persistent file storage for your AI conversations**

Give your LLM the ability to store, manage, and version your files across conversations. No more losing work when a chat ends!

## 🎯 What is User Storage?

User Storage is an Open WebUI tool that provides three isolated storage zones:

| Zone | Purpose | Features |
|------|---------|----------|
| **Uploads/** | Import files from chat | Read-only + delete |
| **Storage/** | Your free workspace | All operations, including shell commands |
| **Documents/** | Version-controlled files | Automatic Git tracking |

## ⚡ Quick Start

### 1. Install the tool
Copy the tool code to Open WebUI: `Workspace > Tools > + > Paste code`

### 2. Important model options

#### Native Function Calling (required)
This tool requires Native mode to work properly:
- Go to `Admin Panel > Settings > Models`
- Select your model
- `Advanced Parameters > Function Calling > "Native"`

#### File Context (recommended: Off)
Open WebUI can automatically inject uploaded file contents into the conversation context. While useful for immediate analysis, this feature can cause issues with User Storage:
- **Double processing**: The model receives file content twice (in context + via tool)
- **Wasted tokens**: Large files consume context window unnecessarily
- **Model confusion**: The model may try to work from memory instead of using tools

**Recommendation**: Disable File Context when using User Storage for file management.
- Go to `Admin Panel > Settings > Models`
- Select your model
- `Capabilities > File Context > Off`

**When to keep it enabled**: If you primarily need the model to immediately analyze or discuss uploaded files without storing them, File Context provides faster responses.

### 3. Start using it!

**Upload a file:**
```
"Import this CSV file into my storage"
→ LLM calls: store_import(import_all=True)
→ LLM calls: store_uploads_to_storage(src="file.csv", dest="file.csv")
```

**Read a file:**
```
"Show me the contents of report.md"
→ LLM calls: store_storage_exec(cmd="cat", args=["report.md"])
```

**Create a file:**
```
"Create a markdown file with my meeting notes"
→ LLM calls: store_storage_write(path="notes.md", content="# Meeting Notes\n...")
```

**Version control:**
```
"Save this document with Git tracking"
→ LLM calls: store_documents_write(path="doc.md", content="...", message="Initial version")
```

## 💬 How to Talk to Your LLM

The LLM understands natural language. Here are some examples:

### File Operations
| You say | LLM understands |
|---------|-----------------|
| "Import this file" | `store_import(import_all=True)` |
| "Save this to my storage" | `store_storage_write(...)` |
| "Delete the old backup" | `store_storage_delete(path="old_backup.txt")` |
| "Rename report.md to final.md" | `store_storage_rename(...)` |
| "Move this to Documents for versioning" | `store_storage_to_documents(...)` |

### Reading & Searching
| You say | LLM understands |
|---------|-----------------|
| "Show me the file" | `store_storage_exec(cmd="cat", ...)` |
| "List my files" | `store_storage_exec(cmd="ls", args=["-la"])` |
| "Search for 'TODO' in all files" | `store_storage_exec(cmd="grep", args=["-r", "TODO", "."])` |
| "Count lines in data.csv" | `store_storage_exec(cmd="wc", args=["-l", "data.csv"])` |

### Safe Editing (with rollback)
| You say | LLM understands |
|---------|-----------------|
| "Open report.md for safe editing" | `store_storage_edit_open("report.md")` |
| "Replace all 'draft' with 'final'" | `store_storage_edit_exec(cmd="sed", ...)` |
| "Save the changes" | `store_storage_edit_save("report.md")` |
| "Cancel, keep the original" | `store_storage_edit_cancel("report.md")` |

### Version Control (Documents zone)
| You say | LLM understands |
|---------|-----------------|
| "Save with a commit message" | `store_documents_write(..., message="...")` |
| "Show Git history" | `store_documents_exec(cmd="git", args=["log"])` |
| "What changed since last version?" | `store_documents_exec(cmd="git", args=["diff", "HEAD~1"])` |

## 🗂️ The Three Zones Explained

### Uploads/ - Temporary Import Zone
- Files you upload to chat land here
- Isolated per conversation
- Read-only (can't modify uploaded files)
- Use bridges to move files to Storage or Documents

### Storage/ - Your Free Workspace
- Full read/write access
- Run shell commands (140+ whitelisted)
- Safe editing with rollback capability
- Git available if you have a repo
- Persistent across conversations

### Documents/ - Version-Controlled Space
- Automatic Git repository
- Every write creates a commit
- Full history tracking
- Safe editing with rollback
- Perfect for important documents

## 🔧 Available Commands

The tool includes 140+ whitelisted shell commands:

**Text processing:** `cat`, `head`, `tail`, `grep`, `sed`, `awk`, `sort`, `uniq`, `cut`, `tr`...

**File management:** `ls`, `cp`, `mv`, `rm`, `mkdir`, `touch`, `find`, `tree`...

**Archives:** `tar`, `zip`, `unzip`, `gzip`, `7z`...

**Data tools:** `jq` (JSON), `sqlite3`, `csvtool`...

**Media:** `ffmpeg`, `ffprobe`, `convert` (ImageMagick)...

**Version control:** `git` (status, log, diff, commit, etc.)

Run `store_allowed_commands()` to see all available commands.

## 🛡️ Security Features

- **Chroot isolation**: Can't escape your storage zone
- **Command whitelist**: Only safe commands allowed
- **No network**: wget, curl, ssh blocked
- **No shells**: bash, python, perl blocked
- **Argument sanitization**: Injection attempts blocked
- **File size limits**: Configurable quotas

## 📊 Utilities

| Command | What it does |
|---------|--------------|
| `store_help()` | Shows documentation |
| `store_stats()` | Storage usage statistics |
| `store_allowed_commands()` | Lists available commands |
| `store_maintenance()` | Cleans expired locks |
| `store_force_unlock(path, zone)` | Emergency unlock |

## 🎓 Example Workflows

### Data Analysis Workflow
```
1. Upload your CSV
2. "Import this file and show me the first 10 rows"
3. "Count how many unique values are in column 2"
4. "Filter rows where column 3 is greater than 100"
5. "Save the filtered data as filtered.csv"
```

### Document Writing Workflow
```
1. "Create a new document called thesis.md in Documents"
2. "Add the introduction section"
3. "Show me the Git history"
4. "Add the methodology section"
5. "What's changed since my first commit?"
```

### Safe Editing Workflow
```
1. "Open config.json for safe editing"
2. "Change the API endpoint to production"
3. "Show me the changes"
4. "Looks good, save it" OR "Cancel, revert to original"
```

## 💡 Tips

1. **Always import first**: When you upload a file, ask the LLM to import it before doing anything else.

2. **Use Documents for important files**: Automatic versioning means you can always recover previous versions.

3. **Safe editing for risky changes**: When modifying critical files, use the edit_open/edit_save workflow.

4. **Check your stats**: Run `store_stats()` periodically to monitor your storage usage.

5. **Be specific**: Tell the LLM exactly what you want - "save as markdown" or "delete the backup folder".

## 🔗 Requirements

- Open WebUI 0.4.0+
- Native Function Calling enabled
- Models with tool label

---

**Happy storing! 📁**
