---
name: "article-extractor"
displayName: "Web Article Ripper"
description: "Extract clean, readable content from URLs (blogs, news, tutorials) without ads or clutter."
keywords: [
  "extract", "download article", "save blog", "rip", 
  "提取", "下载文章", "保存博客", "抓取", "去广告", "readability"
]
---

# Onboarding

This power allows you to strip away ads, popups, and clutter from web pages to save clean text.

## Dependency Check
Before running extractions, verify if the optimized tools are installed. Run this check silently:

1.  **Check for `reader`** (Mozilla Readability - Recommended):
    `command -v reader`
    *If missing, suggest:* `npm install -g @mozilla/readability-cli`
2.  **Check for `trafilatura`** (Python - Good for academic/news):
    `command -v trafilatura`
    *If missing, suggest:* `pip3 install trafilatura`

*Note: If no tools are found, you can use the `curl` fallback method, but quality will be lower.*

# Best Practices

## 🎯 Extraction Strategy

Always follow this priority order to ensure the best quality text:

1.  **Primary**: Use `reader` (Best formatting, preserves structure).
2.  **Secondary**: Use `trafilatura` (Best for complex layouts/multi-language).
3.  **Fallback**: Use `curl` + Python parsing (Emergency only).

## 💻 Execution Workflow

When the user asks to "extract [URL]" or "save this article", execute the following logic sequence in the terminal.

### Step 1: define variables & Logic

Construct a script block that handles the extraction, title cleaning, and saving automatically.

**Use this robust pattern:**

```bash
URL="<USER_PROVIDED_URL>"

# 1. Determine Tool & Extract
if command -v reader &> /dev/null; then
    echo "Using tool: Mozilla Reader..."
    # Reader puts title in the first line usually
    reader "$URL" > temp_raw.txt
    TITLE=$(head -n 1 temp_raw.txt | sed 's/^# //')
    
elif command -v trafilatura &> /dev/null; then
    echo "Using tool: Trafilatura..."
    # Get Metadata for title
    METADATA=$(trafilatura --URL "$URL" --json)
    TITLE=$(echo "$METADATA" | python3 -c "import json, sys; print(json.load(sys.stdin).get('title', 'Article'))")
    # Extract text
    trafilatura --URL "$URL" --output-format txt --no-comments > temp_raw.txt
    
else
    echo "Using fallback: Curl..."
    # Basic Title Grep
    TITLE=$(curl -s "$URL" | grep -oP '<title>\K[^<]+' | head -n 1 | sed 's/ - .*//')
    # Basic HTML Strip (Simple Python one-liner)
    curl -s "$URL" | python3 -c "import sys, re; h=sys.stdin.read(); t=re.sub('<script[^>]*>.*?</script>', '', h, flags=re.DOTALL); t=re.sub('<style[^>]*>.*?</style>', '', t, flags=re.DOTALL); t=re.sub('<[^>]+>', '', t); print('\n'.join([l.strip() for l in t.splitlines() if l.strip()]))" > temp_raw.txt
fi

# 2. Clean Filename (Critical for filesystem safety)
# Remove special chars, limit length, replace spaces
SAFE_TITLE=$(echo "$TITLE" | tr '/' '-' | tr ':' '-' | tr '?' '' | tr '"' '' | tr '<>' '' | tr '|' '-' | tr ' ' '_' | cut -c 1-80)
FILENAME="${SAFE_TITLE}.txt"

# 3. Save and Preview
mv temp_raw.txt "$FILENAME"
echo "✅ Saved as: $FILENAME"
echo "--- PREVIEW ---"
head -n 10 "$FILENAME"
```

## 🚨 Error Handling Rules

1.  **Paywalls**: If the extracted file is empty or contains "Subscribe to read", inform the user: "This article appears to be behind a paywall."
2.  **Empty Output**: If a tool fails, try the next one in the priority list.
3.  **Filenames**: Never blindly trust the web title. Always sanitize it (as shown in the script above) to avoid file system errors.

## 🧠 Post-Processing (Optional)

After extracting, ask the user:
> "Extraction complete. Would you like me to **summarize** this article or extract **key insights** for you?"
