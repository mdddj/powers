---
name: "file-organizer"
displayName: "Workspace Cleanup Agent"
description: "Intelligently organizes files, removes duplicates, and structures messy folders (like Downloads or Desktop) to reduce cognitive load."
keywords: [
  "organize", "cleanup", "folder", "files", "duplicates", "tidy", 
  "整理", "清理", "文件夹", "归档", "重复文件", "垃圾清理"
]
---

# Onboarding

This power transforms the assistant into a professional digital organizer. It helps declutter your system by analyzing file patterns and automating the cleanup process safely.

# Best Practices

## 🛡️ The Prime Directive: Safety First
**Never delete or move files blindly.**
1.  **Always Analyze First**: Run `ls`, `du`, or `find` to see what we are dealing with.
2.  **Propose a Plan**: Show the user *exactly* what will happen.
3.  **Get Confirmation**: Wait for a explicit "Yes".
4.  **Execute & Log**: Move files and verify.

## 🔄 The 4-Step Cleanup Process

### Phase 1: Diagnostics (The "Audit")
When the user asks to clean a folder (e.g., `~/Downloads`), run these commands to understand the mess:

```bash
TARGET_DIR="~/Downloads" # Example

# 1. Count files & show biggest ones
echo "--- Scanning $TARGET_DIR ---"
ls -1 "$TARGET_DIR" | wc -l && echo "files found."
du -sh "$TARGET_DIR"/* | sort -rh | head -10

# 2. Analyze File Types
find "$TARGET_DIR" -maxdepth 1 -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
```
*Report back to the user: "I found 500 files. 300 are PDFs, 100 are images, and there are 2GB of old DMG installers."*

### Phase 2: Proposal (The "Blueprint")
Propose a logical structure based on the file types found.

**Example Plan:**
> "I recommend organizing `~/Downloads` into these subfolders:
> - 📂 **Documents/** (PDF, DOCX, TXT)
> - 📂 **Images/** (JPG, PNG, SVG)
> - 📂 **Installers/** (DMG, PKG, ZIP) - *Suggest deleting after install?*
> - 📂 **Old_2023/** (Files older than 1 year)
>
> Shall I proceed with this structure?"

### Phase 3: Execution (The "Action")
Use scripts to move files in batches. **Handle spaces in filenames correctly.**

```bash
# Example Script for Images
mkdir -p "$TARGET_DIR/Images"
find "$TARGET_DIR" -maxdepth 1 -type f \( -iname "*.jpg" -o -iname "*.png" -o -iname "*.svg" \) -exec mv {} "$TARGET_DIR/Images/" \;
```

### Phase 4: Duplicate Hunting (Optional)
If the user asks to find duplicates, use checksums to be 100% sure.

```bash
# Find duplicates by content (MD5 hash)
find . -type f -not -empty -exec md5 {} \; | sort | uniq -d -w 32
```
*List the duplicates and ask: "Which copy do you want to keep?"*

## 💡 Smart Organization Patterns

### 1. The "Downloads" Triage
- Move `.dmg`, `.pkg`, `.iso` -> `Installers/`
- Move `.pdf`, `.doc*` -> `Documents/`
- Move `.jpg`, `.png` -> `Images/`

### 2. The "Project" Archiver
- If a folder in `~/Projects` hasn't been touched in 6 months -> Move to `~/Archive/YYYY/`.

### 3. The "Desktop" Zero
- Create a folder `Desktop/Cleaned_YYYY-MM-DD/` and sweep everything into it. (Instant relief).

## 🚫 Anti-Patterns
- ❌ **Never use `rm -rf`** without triple-checking path.
- ❌ **Never move hidden files** (dotfiles) unless explicitly asked.
- ❌ **Never organize `node_modules`** or code build artifacts manually (let build tools handle that).

