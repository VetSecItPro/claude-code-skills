---
description: Start a clean development server (fast, resilient, self-healing)
allowed-tools: Bash(git *), Bash(npm *), Bash(npx *), Bash(pnpm *), Bash(yarn *), Bash(sleep *), Bash(cat *), Bash(node *), Bash(mkdir *), Bash(ls *), Bash(head *), Bash(tail *), Bash(grep *), Bash(find *), Bash(xargs *), Bash(kill *), Bash(pkill *), Bash(lsof *), Bash(echo *), Bash(source *), Bash(export *), Bash(df *), Bash(du *), Bash(ps *), Bash(wc *), Bash(stat *), Bash(PATH=*), Bash(open *), Bash(curl *), Read, Write, Edit, Glob, Grep, Task
---

# /dev — Just Works

**One command. No flags to remember. It figures out what's wrong and fixes it.**

```bash
/dev 5888
```

That's it. Server starts on port 5888. If something's broken, it fixes it automatically.

---

## Usage

```bash
/dev 5888        # Start on port 5888 — handles everything
/dev             # Start on port 3000 (default)
/dev 5888 -v     # Verbose: show all monitoring/healing output
/dev 5888 -m     # Minimal: just start, skip smart features
```

---

## What It Does Automatically

You don't need to think about any of this. It just happens:

| Situation | Auto-Fix |
|-----------|----------|
| Port already in use | Kills the process, starts fresh |
| Stale dependencies (package.json newer than node_modules) | Runs install |
| Corrupted .next cache | Clears it |
| HMR stopped working | Restarts server |
| Memory leak (>3GB) | Warns you, suggests restart |
| Server crashes with ENOENT/heap errors | Auto-restarts with cache clear |
| Repeated crashes | Escalates to deeper clean |

**You only see output when something needed fixing.** Otherwise, it just starts.

---

## Execution Rules

- **NO permission requests** — just execute
- **NO flags to remember** — smart by default
- **Turbopack enabled** — faster rebuilds, stable HMR
- **Silent unless needed** — only shows output when fixing something

---

## Phase 1: Parse Arguments

```bash
PORT=3000
VERBOSE=false
MINIMAL=false

for arg in $ARGUMENTS; do
  if [[ "$arg" =~ ^[0-9]{4,5}$ ]]; then
    PORT="$arg"
  elif [[ "$arg" == "-v" || "$arg" == "--verbose" ]]; then
    VERBOSE=true
  elif [[ "$arg" == "-m" || "$arg" == "--minimal" ]]; then
    MINIMAL=true
  fi
done
```

---

## Phase 2: Setup Environment

### 2.1 Detect Package Manager

```bash
if [ -f "pnpm-lock.yaml" ]; then
  PKG="pnpm"
elif [ -f "yarn.lock" ]; then
  PKG="yarn"
else
  PKG="npm"
fi
```

### 2.2 Set Node Path

```bash
export PATH="$HOME/.nvm/versions/node/v22.18.0/bin:$PATH"
```

---

## Phase 3: Smart Preparation (skip if -m flag)

**All checks run quickly (<2s total). Only act if issues found.**

### 3.1 Kill Port Conflicts

```bash
# Kill anything on target port
lsof -ti:$PORT 2>/dev/null | xargs kill -9 2>/dev/null || true

# Kill orphaned Next.js processes
pkill -f "next dev" 2>/dev/null || true
pkill -f "next-server" 2>/dev/null || true
```

If processes were killed, note: `Cleared port $PORT`

### 3.2 Check Dependencies Freshness

```bash
if [ -f "package.json" ] && [ -d "node_modules" ]; then
  if [ "package.json" -nt "node_modules" ]; then
    # package.json is newer — deps may be stale
    echo "📦 Installing dependencies..."
    $PKG install
  fi
elif [ ! -d "node_modules" ]; then
  echo "📦 Installing dependencies..."
  $PKG install
fi
```

### 3.3 Clear Stale Caches

Always clear `.next` for a clean start (fast, prevents most issues):

```bash
node -e "const fs=require('fs');try{fs.rmSync('.next',{recursive:true,force:true})}catch{}"
```

---

## Phase 4: Start Server

```bash
PORT=$PORT $PKG run dev --turbo
```

**Note:** `--turbo` enables Turbopack (Next.js 15+). Falls back automatically if not supported.

---

## Phase 5: Background Monitoring (skip if -m flag)

After server starts, monitor in background. Only surface issues when they occur.

### 5.1 Health Watcher

Runs silently. Only outputs if problems detected:

```bash
# Monitor loop (conceptual - runs async)
while server_running; do
  # Check memory
  MEM_MB=$(get_process_memory)
  if [ $MEM_MB -gt 3000 ]; then
    echo "⚠️  Memory high: ${MEM_MB}MB — consider restarting (/dev $PORT)"
  fi

  sleep 60
done
```

### 5.2 Crash Recovery

If server crashes, automatically attempt recovery:

**Level 1 — Quick restart:**
```bash
$PKG run dev --turbo --port $PORT
```

**Level 2 — Clear caches and restart:**
```bash
node -e "const fs=require('fs');['.next','node_modules/.cache','.swc'].forEach(d=>{try{fs.rmSync(d,{recursive:true,force:true})}catch{}})"
find . -maxdepth 1 -name '*.tsbuildinfo' -delete 2>/dev/null || true
$PKG run dev --turbo --port $PORT
```

**Level 3 — Reinstall and restart:**
```bash
node -e "const fs=require('fs');try{fs.rmSync('node_modules',{recursive:true,force:true})}catch{}"
$PKG install
$PKG run dev --turbo --port $PORT
```

**Level 4 — Nuclear (after 3 failed attempts):**
```bash
pkill -f "node" 2>/dev/null || true
node -e "const fs=require('fs');['.next','node_modules','.swc'].forEach(d=>{try{fs.rmSync(d,{recursive:true,force:true})}catch{}});['pnpm-lock.yaml','package-lock.json'].concat(require('fs').readdirSync('.').filter(f=>f.endsWith('.tsbuildinfo'))).forEach(f=>{try{fs.unlinkSync(f)}catch{}})"
$PKG install
$PKG run dev --turbo --port $PORT
```

If still failing after Level 4, stop and report:
```
❌ Server failed to start after multiple attempts.
   Last error: [error message]
   Try: Check for syntax errors or missing env vars.
```

---

## Output Format

### Normal Start (nothing broken):

```
🚀 http://localhost:5888
```

That's it. One line. Server is running.

### Start After Fixing Something:

```
🚀 http://localhost:5888
   └─ Fixed: Cleared stale cache
```

### Start After Multiple Fixes:

```
🚀 http://localhost:5888
   ├─ Fixed: Killed process on port 5888
   ├─ Fixed: Installed dependencies
   └─ Fixed: Cleared corrupted .next cache
```

### Verbose Mode (-v):

```
═══════════════════════════════════════════════════════════════
🚀 DEV SERVER
═══════════════════════════════════════════════════════════════
   URL:        http://localhost:5888
   Package:    pnpm
   Turbopack:  Enabled

   Checks:
   ├─ Port 5888:     ✓ Clear
   ├─ Dependencies:  ✓ Up to date
   ├─ Cache:         ✓ Cleared
   └─ Disk space:    ✓ 42GB free

   Monitoring:       Active (memory, crashes)
═══════════════════════════════════════════════════════════════
```

### Minimal Mode (-m):

```
http://localhost:5888
```

No emoji, no fixes, just starts. Use when you know everything is fine.

---

## Flag Reference

| Flag | What It Does |
|------|--------------|
| (none) | Smart mode — auto-fixes everything silently |
| `-v` | Verbose — show all checks and monitoring |
| `-m` | Minimal — skip smart features, just start fast |

---

## Examples

```bash
/dev 5888           # 99% of the time, this is all you need
/dev 3000           # Different port
/dev                # Default port 3000
/dev 5888 -v        # "Show me what you're doing"
/dev 5888 -m        # "I know it's fine, just start"
```

---

## Why This Design

1. **No flags to memorize** — Smart behavior is the default
2. **Silent success** — Only shows output when there's something to report
3. **Graduated recovery** — Tries simple fixes first, escalates if needed
4. **Fast path preserved** — If nothing's wrong, startup is ~3-5s
5. **Escape hatches exist** — `-m` for minimal, `-v` for verbose

The goal: You type `/dev 5888` and forget about it. It either works, or it tells you why it can't.

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
