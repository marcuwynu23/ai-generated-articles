
# 🧹 How to Completely Remove a File from Git History (and Keep It Ignored)

Sometimes secrets (like API keys or config files) get accidentally committed into a Git repository.  
Even if you delete the file in a later commit, it still exists in your **entire history** — which means anyone can recover it.  

This guide shows you how to **safely purge a file from all commits in history** and make sure it never comes back.

---

## ⚠️ Warning

- This process **rewrites Git history**.  
- You will need to **force push** the cleaned history to your remote.  
- All contributors must **re-clone** the repo afterward (not just pull).  

---

## 🔹 Step 1. Backup Your Repo

Before making destructive changes, make a mirror backup:

```bash
git clone --mirror git@github.com:your-org/your-repo.git repo-backup.git
```

This way, you always have the original history in case something goes wrong.

---

## 🔹 Step 2. Use a Purge Script

To make it easy, we provide **three scripts** depending on your system:

### 🐧 Bash (Linux/Mac/Git Bash) → `purge-file.sh`
```bash
#!/bin/bash
set -e

FILE_TO_REMOVE=$1

if [ -z "$FILE_TO_REMOVE" ]; then
  echo "❌ Usage: ./purge-file.sh <file-path>"
  exit 1
fi

ORIGIN_URL=$(git remote get-url origin 2>/dev/null || true)

if [ -n "$ORIGIN_URL" ]; then
  echo "📌 Saving origin remote: $ORIGIN_URL"
else
  echo "⚠️ No origin found!"
fi

echo "🧹 Purging $FILE_TO_REMOVE from history..."
git filter-repo --path "$FILE_TO_REMOVE" --invert-paths --force

if [ -n "$ORIGIN_URL" ]; then
  git remote add origin "$ORIGIN_URL"
  echo "✅ Re-added origin: $ORIGIN_URL"
fi

echo "👉 Run:"
echo "   git push origin --force --all"
echo "   git push origin --force --tags"
```

Run:
```bash
chmod +x purge-file.sh
./purge-file.sh sample/docs.config.yml
```

---

### 🪟 PowerShell (Windows) → `purge-file.ps1`
```powershell
param (
    [string]$FileToRemove
)

if (-not $FileToRemove) {
    Write-Error "❌ Usage: .\purge-file.ps1 <file-path>"
    exit 1
}

$OriginUrl = git remote get-url origin 2>$null
if ($LASTEXITCODE -eq 0) {
    Write-Output "📌 Saving origin remote: $OriginUrl"
} else {
    Write-Warning "⚠️ No origin found!"
    $OriginUrl = ""
}

Write-Output "🧹 Purging $FileToRemove from history..."
git filter-repo --path $FileToRemove --invert-paths --force

if ($OriginUrl) {
    git remote add origin $OriginUrl
    Write-Output "✅ Re-added origin: $OriginUrl"
}

Write-Output "👉 Run:"
Write-Output "   git push origin --force --all"
Write-Output "   git push origin --force --tags"
```

Run:
```powershell
.\purge-file.ps1 "sample/docs.config.yml"
```

---

### 🐍 Python (Cross-platform) → `purge_file.py`
```python
import subprocess
import sys

def run(cmd, capture=False):
    try:
        if capture:
            return subprocess.check_output(cmd, shell=True, text=True).strip()
        else:
            subprocess.check_call(cmd, shell=True)
    except subprocess.CalledProcessError:
        return None

if len(sys.argv) < 2:
    print("❌ Usage: python purge_file.py <file-path>")
    sys.exit(1)

file_to_remove = sys.argv[1]

origin_url = run("git remote get-url origin", capture=True)
if origin_url:
    print(f"📌 Saving origin remote: {origin_url}")
else:
    print("⚠️ No origin found!")
    origin_url = None

print(f"🧹 Purging {file_to_remove} from history...")
run(f"git filter-repo --path {file_to_remove} --invert-paths --force")

if origin_url:
    run(f"git remote add origin {origin_url}")
    print(f"✅ Re-added origin: {origin_url}")

print("👉 Run:")
print("   git push origin --force --all")
print("   git push origin --force --tags")
```

Run:
```bash
python purge_file.py sample/docs.config.yml
```

---

## 🔹 Step 3. Force Push the Clean Repo

After running any script:

```bash
git push origin --force --all
git push origin --force --tags
```

---

## 🔹 Step 4. Add to `.gitignore`

Prevent future accidents by adding the file to `.gitignore`:

```bash
echo "sample/docs.config.yml" >> .gitignore
git add .gitignore
git commit -m "chore: ignore sensitive config file"
git push origin main
```

---

## ✅ Done!

Now:
- The file is **completely removed from history**  
- The remote repo no longer has it  
- Future commits will ignore it  
