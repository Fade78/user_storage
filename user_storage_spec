# SPECIFICATION - User Storage Tool v2.4

## Overview

User Storage is an Open WebUI tool that allows users to store, manipulate, and organize their files persistently between conversations. The tool offers three isolated zones with different security and functionality levels.

## Architecture

### Three isolated zones

```
{user_root}/
├── Uploads/           # Import zone (read-only + delete)
│   └── {conv_id}/     # Isolated per conversation
│       └── files...
├── Storage/           # Free zone (all operations)
│   ├── data/          # User files
│   ├── editzone/      # Temporary working copies
│   │   └── {conv_id}/
│   └── locks/         # Edit locks
└── Documents/         # Versioned zone (auto Git)
    ├── data/          # Git repository
    │   └── .git/
    ├── editzone/
    │   └── {conv_id}/
    └── locks/
```

### Paths

- **User base**: `/app/backend/data/user_files/users/{user_id}/`
- **Uploads**: `{user_root}/Uploads/{conv_id}/`
- **Storage**: `{user_root}/Storage/data/`
- **Documents**: `{user_root}/Documents/data/`

## Functions (30 total)

### Import (1 function)
| Function | Description |
|----------|-------------|
| `store_import(filename, import_all, dest_subdir)` | Imports uploaded files to Uploads/ |

### Uploads (2 functions)
| Function | Description |
|----------|-------------|
| `store_uploads_exec(cmd, args, timeout)` | Executes a command (read-only) |
| `store_uploads_delete(path)` | Deletes a file/folder |

### Storage - Normal (4 functions)
| Function | Description |
|----------|-------------|
| `store_storage_exec(cmd, args, timeout)` | Executes a command |
| `store_storage_write(path, content, append)` | Writes content |
| `store_storage_delete(path)` | Deletes a file/folder |
| `store_storage_rename(old_path, new_path)` | Renames/moves |

### Storage - Safe editing (5 functions)
| Function | Description |
|----------|-------------|
| `store_storage_edit_open(path)` | Locks and copies to editzone |
| `store_storage_edit_exec(cmd, args, timeout)` | Executes in editzone |
| `store_storage_edit_write(path, content, append)` | Writes in editzone |
| `store_storage_edit_save(path)` | Saves to data/ |
| `store_storage_edit_cancel(path)` | Cancels (rollback) |

### Documents - Normal (4 functions)
| Function | Description |
|----------|-------------|
| `store_documents_exec(cmd, args, timeout)` | Executes a command (includes Git) |
| `store_documents_write(path, content, append, message)` | Writes + Git commit |
| `store_documents_delete(path, message)` | Deletes + Git commit |
| `store_documents_rename(old_path, new_path, message)` | git mv + commit |

### Documents - Safe editing (5 functions)
| Function | Description |
|----------|-------------|
| `store_documents_edit_open(path)` | Locks and copies |
| `store_documents_edit_exec(cmd, args, timeout)` | Executes in editzone |
| `store_documents_edit_write(path, content, append)` | Writes in editzone |
| `store_documents_edit_save(path, message)` | Saves + commit |
| `store_documents_edit_cancel(path)` | Cancels (rollback) |

### Bridges (4 functions)
| Function | Description |
|----------|-------------|
| `store_uploads_to_storage(src, dest)` | Copies Uploads → Storage |
| `store_uploads_to_documents(src, dest, message)` | Copies Uploads → Documents + commit |
| `store_storage_to_documents(src, dest, message)` | Copies Storage → Documents + commit |
| `store_documents_to_storage(src, dest, message)` | Moves Documents → Storage + git rm |

### Utilities (5 functions)
| Function | Description |
|----------|-------------|
| `store_help()` | Full documentation |
| `store_stats()` | Usage statistics |
| `store_allowed_commands()` | List of available commands |
| `store_force_unlock(path, zone)` | Force unlock |
| `store_maintenance()` | Cleanup locks/editzones |

## Security

### Path validation
- **`_resolve_chroot_path()`**: Resolves and verifies path stays within chroot
- **`_validate_relative_path()`**: Validates relative paths (blocks escaping `../`)
- **`_validate_path_args()`**: Validates paths in command arguments

### Command whitelists

**WHITELIST_READONLY (76 commands)** - Uploads:
```
cat, head, tail, less, more, nl, wc, stat, file, du, tac,
ls, tree, find, grep, egrep, fgrep, rg, awk, sed,
sort, uniq, cut, paste, tr, fold, fmt, column, rev, shuf,
expand, unexpand, pr, join, diff, diff3, cmp, comm,
tar, unzip, zipinfo, 7z, zcat, bzcat, xzcat,
md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum,
base32, base64, basenc, strings, od, hexdump, xxd,
jq, xmllint, yq, iconv, bc, dc, expr, factor, numfmt,
basename, dirname, realpath, echo, printf,
ffprobe, identify, exiftool, sqlite3
```

**WHITELIST_READWRITE (139 commands)** - Storage, Documents:
- All READONLY commands +
```
df, locate, which, whereis, split, csplit,
sdiff, patch, colordiff, zip, 7za,
gzip, gunzip, bzip2, bunzip2, xz, unxz, lz4, zstd,
sum, uuencode, uudecode,
touch, mkdir, rm, rmdir, mv, cp, ln, truncate, mktemp, install, shred, rename,
chmod, pandoc, dos2unix, unix2dos, recode, seq, date, cal,
readlink, pathchk, pwd, uname, nproc, printenv, env,
timeout, sleep, yes, tee, xargs, envsubst, gettext, tsort, true, false,
ffmpeg, magick, convert, git
```

### Git subcommands

**Read**: `status, log, show, diff, branch, tag, blame, ls-files, ls-tree, shortlog, reflog, describe, rev-parse, rev-list, cat-file`

**Write**: `add, commit, reset, restore, checkout, rm, mv, revert, cherry-pick, stash, clean`

**Forbidden**: `push, pull, fetch, clone, remote, gc, prune, filter-branch`

### Forbidden commands (BLACKLIST)
```
bash, sh, zsh, fish, dash, csh, tcsh, ksh,
python, python3, perl, ruby, node, php, lua,
exec, eval, source,
nohup, disown, setsid, screen, tmux, at, batch, crontab,
sudo, su, doas, chown, chgrp,
wget, curl, fetch, ssh, scp, sftp, rsync,
nc, netcat, ncat, telnet, ftp, ping, traceroute,
dd, mount, umount, kill, killall, pkill,
reboot, shutdown, halt, poweroff,
systemctl, service, mkfs, fdisk, parted,
iptables, firewall-cmd
```

### Forbidden arguments
Pattern: `[;&|`$\n\r]|&&|\|\||>>|<<|>\s|<\s|\$\(|\$\{`

Blocks: `;`, `|`, `&`, `&&`, `||`, `>`, `>>`, `<`, `<<`, `$(`, `${`, `` ` ``, `\n`, `\r`

### Size validation
- Max content: configurable via `max_file_size_mb` (default: 300 MB)

## Configuration (Valves)

| Parameter | Default | Description |
|-----------|---------|-------------|
| `storage_base_path` | `/app/backend/data/user_files/users` | Storage root |
| `quota_per_user_mb` | 1000 | Quota per user (MB) |
| `max_file_size_mb` | 300 | Max file size (MB) |
| `lock_max_age_hours` | 24 | Max lock duration (hours) |
| `exec_timeout_default` | 30 | Default timeout (seconds) |
| `exec_timeout_max` | 300 | Maximum timeout (seconds) |

## Response format

### Success
```json
{
  "success": true,
  "data": { ... },
  "message": "Description"
}
```

### Error
```json
{
  "success": false,
  "error": "ERROR_CODE",
  "message": "Description",
  "details": { ... },
  "hint": "Suggestion"
}
```

### Error codes
| Code | Description |
|------|-------------|
| `PATH_ESCAPE` | Chroot escape attempt |
| `COMMAND_FORBIDDEN` | Forbidden command |
| `ARGUMENT_FORBIDDEN` | Dangerous argument |
| `FILE_NOT_FOUND` | File not found |
| `FILE_LOCKED` | File is locked |
| `QUOTA_EXCEEDED` | Quota exceeded |
| `TIMEOUT` | Command timeout |
| `COMMAND_NOT_FOUND` | Command not available |
| `EXEC_ERROR` | Execution error |
| `ZONE_FORBIDDEN` | Invalid zone |

## Typical workflows

### Importing an uploaded file
```
1. store_import(import_all=True)           → Copies to Uploads/
2. store_uploads_to_storage(src, dest)     → Copies to Storage/
3. store_storage_exec(cmd="cat", args=["file"])  → Read
```

### Safe editing
```
1. store_storage_edit_open("report.md")    → Locks + copies
2. store_storage_edit_exec(cmd="sed", args=["-i", "s/old/new/g", "report.md"])
3. store_storage_edit_save("report.md")    → Save
   OR
   store_storage_edit_cancel("report.md")  → Rollback
```

### Versioned documents
```
1. store_documents_write("notes.md", "# Notes", message="Initial")
2. store_documents_exec(cmd="git", args=["log", "--oneline"])
3. store_documents_exec(cmd="git", args=["diff", "HEAD~1"])
```

## Open WebUI requirements

For the tool to work properly, enable **Native Function Calling**:

**Option 1 - Per model** (recommended):
```
Admin Panel > Settings > Models > [Select model] > Advanced Parameters > Function Calling > "Native"
```

**Option 2 - Per chat**:
```
Chat Controls (⚙️) > Advanced Params > Function Calling > "Native"
```

## Changelog v2.4

- ✅ Full English translation
- ✅ PATH_ESCAPE vulnerability fix (7 functions)
- ✅ Fix `store_import` for Open WebUI format (`{id}_{name}`)
- ✅ Added `_validate_relative_path()`
- ✅ Secured `file_name` in import
- ✅ Removed debug code
- ✅ Git available in Storage and Documents
