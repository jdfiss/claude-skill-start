# /start：Claude Code 開新專案 skill

[English](#english) · [中文](#中文)

---

## 中文

在終端機叫出 Claude Code，打 `/start`（或說「開新專案」），它會先問你幾個問題，再一次把專案環境建好：

1. **問需求**：專案名稱與位置、用途、筆記放哪裡、要不要 git、要不要 Telegram、Python 環境、其他需求。每題都有建議預設值，一路按下去就好。
2. **建環境**：只做你選的項目
   - 專案資料夾、Python `.venv`、`requirements.txt`
   - `.env` 和 `.env.example`、`.gitignore`
   - 精簡版 `CLAUDE.md`，之後每次叫出 Claude 都知道專案背景
   - 筆記：自動偵測你的 Obsidian vault，可以選存進現有 vault、建新 vault、放專案的 `docs/`，或兩邊都要
   - `git init` 加第一個 commit（不會自動 push）
3. **連 Telegram**：用文字編輯器打開 `.env`，讓你自己填 Bot Token 和 Chat ID（token 不會出現在對話裡），填好後測試 `getMe` 並送一則測試訊息。

已經存在的檔案（`.git`、`.venv`、`CLAUDE.md`、`.env`）不會被覆蓋，只會補上缺的。

支援 Windows、macOS、Linux。

### 安裝

把 `skills/start` 資料夾複製到你的 Claude Code skills 目錄：

```bash
git clone https://github.com/jdfiss/claude-skill-start.git
# macOS / Linux
cp -r claude-skill-start/skills/start ~/.claude/skills/
# Windows (PowerShell)
Copy-Item -Recurse claude-skill-start\skills\start $HOME\.claude\skills\
```

只想在單一專案裡用的話，就改放到該專案的 `.claude/skills/`。

裝好後重開 Claude Code，打 `/start` 就會出現。

### 需求

- [Claude Code](https://claude.com/claude-code)
- Python 3（要建 venv 才需要）
- git（要用 git 才需要）
- Obsidian、Telegram bot 都是選配

---

## English

Open Claude Code in your terminal and type `/start` (or say "start a project"). It interviews you first, then sets up the whole project in one go:

1. **Interview**: project name and location, purpose, where notes go, git, Telegram, Python environment, anything else. Every question has a recommended default.
2. **Build**: only what you picked
   - Project folder, Python `.venv`, `requirements.txt`
   - `.env` + `.env.example`, `.gitignore`
   - A short `CLAUDE.md` so every future Claude session knows the project context
   - Notes: detects your Obsidian vaults; store in an existing vault, a new vault, the project's `docs/`, or both
   - `git init` plus a first commit (never pushes on its own)
3. **Telegram**: opens `.env` in a text editor so you fill in the Bot Token and Chat ID yourself (the token never goes through the chat), then checks `getMe` and sends a test message.

Existing files (`.git`, `.venv`, `CLAUDE.md`, `.env`) are never overwritten; only missing pieces are added.

Works on Windows, macOS and Linux. The skill's instructions are written in Traditional Chinese; Claude follows them fine and will talk to you in your own language.

### Install

Copy the `skills/start` folder into your Claude Code skills directory:

```bash
git clone https://github.com/jdfiss/claude-skill-start.git
# macOS / Linux
cp -r claude-skill-start/skills/start ~/.claude/skills/
# Windows (PowerShell)
Copy-Item -Recurse claude-skill-start\skills\start $HOME\.claude\skills\
```

To use it in one project only, put it in that project's `.claude/skills/` instead.

Restart Claude Code and type `/start`.

### Requirements

- [Claude Code](https://claude.com/claude-code)
- Python 3 (only for the venv step)
- git (only for the git step)
- Obsidian and a Telegram bot are optional

## License

MIT
