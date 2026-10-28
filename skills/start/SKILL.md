---
name: start
description: 開新專案的一條龍設定。使用者在終端機（例如 VS Code 內建終端機）叫出 Claude Code、準備開始一個新專案時使用：先訪談需求（專案名稱與位置、用途、Obsidian 筆記路徑、要不要 git、要不要 Telegram、Python 環境、其他需求），再依答案建好環境（資料夾、.venv、.env、.gitignore、CLAUDE.md、Obsidian 筆記、git），最後請使用者填 Telegram 的 Bot Token 和 Chat ID 並先把 Telegram 連上。使用者說「開新專案」「start a project」「建新專案」「新專案」「幫我開專案」「初始化專案」「/start」「/start_a_project」時觸發；在空資料夾裡說「我們要開始做 XXX」也應該主動建議使用。
---

# start（開新專案）

常見的開工方式是：打開終端機 → 叫出 Claude Code → 開始做新專案。接下來的步驟（建環境、指定 Obsidian 路徑、連 Telegram……）每次都一樣又很繁瑣，這個 skill 就是把它們一次做完，讓使用者只需要回答幾個問題。

整個流程分三段：**問 → 建 → 連 TG**。不要跳過「問」直接動手，因為每個專案需要的東西不一樣，建錯了要收拾比多問一句麻煩得多。

---

## 第一段：訪談需求

先看一下現在的工作目錄（`pwd`、`ls`），判斷是空資料夾、已有東西，還是一個放了很多不相關東西的大雜燴目錄（例如下載區、暫存區、家目錄）。已有的東西（`.git`、`.venv`、`CLAUDE.md`、`.env`）之後**不要覆蓋**，只補缺的。

同時確認作業系統（Windows / macOS / Linux），後面的路徑寫法和開檔方式都依系統而定。

接著用 AskUserQuestion 一次問完（最多 4 題一輪，不夠就分兩輪）。每題都給推薦的預設值，讓使用者可以一路按下去：

| 項目 | 要問什麼 | 預設建議 |
|---|---|---|
| 專案名稱與位置 | 資料夾名稱、放哪裡 | 目前目錄若是空的就直接用；若是大雜燴目錄，就在底下建 `<專案名>/` |
| 專案用途 | 一兩句話說明要做什麼（寫進 CLAUDE.md 和 Obsidian 首頁） | — |
| 筆記存放 | 專案筆記要放哪裡（見下方「筆記存放怎麼問」） | 存進最常用的現有 vault，子資料夾 `<專案名>/`；找不到任何 vault 就預設 `docs/` |
| Git | 要不要 `git init`？有沒有遠端 repo 網址？ | 要 init，遠端之後再說 |
| Telegram | 要不要連 TG？ | 要（剛建好就先連，之後要用再說） |
| Python 環境 | 要不要 `.venv`？用哪個 Python？先裝哪些套件？ | 要，用系統預設 Python，只裝 `python-dotenv` |
| 其他 | 還需要什麼？例如 hooks、時間戳、docs/ 資料夾、tests/ | 無 |

### 筆記存放怎麼問

先找出使用者現有的 Obsidian vault，再問問題，選項才會是真的路徑而不是空想：讀 Obsidian 的設定檔 `obsidian.json`，裡面的 `vaults` 列出所有 vault 的 `path`（`ts` 最新或標 `open: true` 的就是最常用的）：

- Windows：`%APPDATA%\obsidian\obsidian.json`
- macOS：`~/Library/Application Support/obsidian/obsidian.json`
- Linux：`~/.config/obsidian/obsidian.json`（Flatpak 版在 `~/.var/app/md.obsidian.Obsidian/config/obsidian/obsidian.json`）

讀不到的話，就在家目錄或常用磁碟底下找含有 `.obsidian` 子資料夾的目錄，但要限制搜尋深度，不要掃整顆硬碟。如果都找不到，代表使用者可能沒在用 Obsidian，這時選項以 `docs/` 為主。

接著用一題 AskUserQuestion 問，選項如下：

- **存進現有的 vault**：每個找到的 vault 都列一個選項（最常用的放第一個並標 Recommended）。選了之後，再確認子資料夾名稱，預設是 `<vault>\<專案名>\`。
- **建一個新的 vault**：問要建在哪裡（例如放在其他 vault 旁邊的 `<專案名>-notes/`）。
- **放在專案裡的 `docs/`**：筆記跟著程式碼一起進 git，不放 Obsidian。適合要交給別人、或之後要公開的專案。
- **都要**：在 Obsidian vault 裡另建一個專案資料夾，同時也建專案的 `docs/`。選了之後一樣要確認用哪個 vault、子資料夾叫什麼。分工方式是：`docs/` 放跟程式碼綁在一起的文件（規格、架構、使用說明），Obsidian 放決策、實驗紀錄和待辦。
- **不需要筆記**

AskUserQuestion 一題最多只能放 4 個選項，所以 vault 太多時，只列最常用的那個，其他 vault 寫在題目裡，請使用者用 Other 輸入。

選好的路徑在下文統稱為「筆記路徑」。選「都要」時，筆記路徑指的是 Obsidian 那個資料夾。

「其他」這題是開放的，要真的讀使用者寫了什麼並照做；如果是本 skill 沒寫到的需求，用常識判斷怎麼做，做不到就說清楚。

問完後，用一段簡短清單複述「我等下會建這些東西」，讓使用者確認一次再動手。

---

## 第二段：依需求建立環境

依使用者的答案，只做被選到的項目。順序如下（後面的步驟會用到前面的結果）：

1. **專案資料夾**：建好並 `cd` 進去。
2. **Python 環境**（選了才做）：
   - `python -m venv .venv`
   - 直接用 venv 裡的 Python 裝套件，不要用 `activate`，因為每次指令的 shell 環境不會保留：
     - Windows：`.venv/Scripts/python.exe -m pip install ...`
     - macOS / Linux：`.venv/bin/python -m pip install ...`
   - 下文的「venv Python」指的就是上面這個路徑
   - 有裝套件就寫 `requirements.txt`
3. **`.env` 與 `.env.example`**：
   - `.env` 放實際值（之後的 TG 設定也寫在這裡）
   - `.env.example` 只放變數名稱和註解，不放真值，方便之後分享 repo
4. **`.gitignore`**：至少包含 `.venv/`、`.env`、`__pycache__/`、`*.pyc`、`.pytest_cache/`。`.env` 一定要被忽略，因為裡面會有 TG token。
5. **CLAUDE.md**：寫一份短的（30 行內就好），讓之後每次叫出 Claude 都知道專案背景：
   ```markdown
   # <專案名>

   <專案用途，一兩句>

   ## 環境
   - Python：`<venv Python 路徑>`（執行程式一律用這個）
   - 機密設定在 `.env`（不進 git）

   ## 筆記
   - 決策與實驗結論寫到：`<筆記路徑>`（命名 `YYYY-MM-DD_主題.md`）

   ## Telegram
   - 設定在 `.env` 的 `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID`
   ```
   沒選的項目就整段不寫，不要留空殼。
6. **筆記**（依選項而定）：
   - 現有 vault：在 vault 裡建子資料夾 `<筆記路徑>`。
   - 新 vault：建資料夾，再在裡面建一個空的 `.obsidian/` 子資料夾，Obsidian 才認得出這是 vault。建好後提醒使用者：要在 Obsidian 裡點「開啟資料夾作為 vault」才會出現在清單上。不要直接去改 `obsidian.json`，Obsidian 開著的時候會把它蓋回去。
   - 專案 `docs/`：建 `<專案>/docs/`，這些筆記會跟著進 git。
   - 都要：同時做「現有 vault」（或新 vault）和「專案 `docs/`」兩項，並在 CLAUDE.md 的「筆記」段寫明分工（`docs/` 放規格與架構，Obsidian 放決策與實驗）。
   - 除了「不需要筆記」，每一種都要建首頁筆記 `<筆記路徑>/00_<專案名>.md`，內容是：專案用途、建立日期（用今天的實際日期）、專案程式碼路徑、一個空的「待辦」清單。
7. **Git**（選了才做）：`git init`，把上面的檔案 `git add`，做第一個 commit（`chore: 初始化專案`）。有遠端網址就 `git remote add origin <url>`，但**不要 push**，push 是對外動作，要使用者另外說。

每一步做完不用逐一報告，全部做完再一起報告。某一步失敗（例如 Python 找不到），就停下來說明原因，問使用者要怎麼處理，不要硬跳過假裝成功。

---

## 第三段：請使用者填 Bot Token 和 Chat ID，先連上 Telegram

只在使用者選了要 Telegram 時才做。目標是「專案剛建好就先連通」，具體要拿 TG 做什麼之後再說，所以這裡只做到**設定寫好、確認連得到**為止，不寫 bot 邏輯。

1. 先在 `.env` 和 `.env.example` 寫好空的欄位：
   ```
   TELEGRAM_BOT_TOKEN=
   TELEGRAM_CHAT_ID=
   ```
2. **直接用文字編輯器打開 `.env` 讓使用者填**，用不會卡住終端機的寫法（不要等編輯器關掉）：
   - Windows：PowerShell `Start-Process notepad.exe "<.env 完整路徑>"`
   - macOS：`open -e "<.env 完整路徑>"`
   - Linux：`xdg-open "<.env 完整路徑>" &`

   接著告訴使用者：「編輯器已經打開 `.env`，請把 Bot Token 和 Chat ID 填在等號後面，存檔後回來跟我說一聲。」同時附上 `.env` 的完整路徑，編輯器沒跳出來時使用者還能自己去開。不要請使用者把 token 貼在對話裡，因為對話紀錄會留存，token 外流別人就能操控 bot。
   - 使用者不知道去哪拿時可以提示：Bot Token 找 BotFather（`/newbot`）；Chat ID 可以先傳一則訊息給 bot，再打 `getUpdates` 從回應裡找 `chat.id`。
3. 使用者說填好後，**測試連線**：
   - 用 venv Python（沒有 venv 就用系統 Python）讀 `.env`，呼叫 `https://api.telegram.org/bot<token>/getMe`，確認 bot 活著。
   - 再送一則「<專案名> 已連線 ✅」到 Chat ID。
   - 印出來的東西和錯誤訊息裡都不能出現 token（例外處理時不要把整個 URL 印出來）。
4. 回報結果：連上了，還是卡在哪一步（欄位還是空的、token 錯、chat id 錯或使用者還沒先傳訊息給 bot），並說明下一步要做什麼。連不上不要算成功。

---

## 最後的總結

用簡短清單告訴使用者：

- 建了哪些東西（路徑寫完整）
- 哪些項目跳過了、為什麼
- TG 連線狀態
- 還需要使用者自己做的事（例如填 token、建遠端 repo）

不用貼出檔案全文，使用者自己會打開看。
