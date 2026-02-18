# Everything Claude Code（everything-claude-code）閱讀報告

來源：<https://github.com/affaan-m/everything-claude-code>（已複製到本機：`/home/istale/.openclaw/workspace/everything-claude-code`）

## 1) 這個 repo 是什麼？
Everything Claude Code（ECC）是一套針對 **Claude Code** 的「生產級配置集合」，主打把日常開發中反覆用到的流程收斂成可重用的：
- **agents**（子代理/分工角色）
- **skills**（工作流/領域知識）
- **commands**（slash commands 快捷指令）
- **hooks**（事件觸發自動化）
- **rules**（規範/準則）
- **MCP configs**（外部工具/服務接入設定）

官方 README 強調：這些配置是在 10+ 個月密集使用、做真實產品的過程中演化出來。

## 2) Repo 組成總覽（本機掃描）
本機目錄結構包含：
- `agents/`：13 個 agent 定義（.md）
- `skills/`：43 個 skill（多為 `skills/<name>/SKILL.md`）
- `commands/`：31 個 command（.md）
- `hooks/`：hook 配置（`hooks/hooks.json` + 說明文件）
- `rules/`：規則（common + 語言分層）
- `contexts/`：dev/review/research 等上下文模板
- `mcp-configs/`：MCP 設定範例
- `plugins/`、`.claude-plugin/`：Claude Code plugin/marketplace 安裝所需檔案
- `docs/`：文件（含 zh-TW/ja-JP 等翻譯）

> 以 repo 自述：提供約 **13 agents / 43 skills / 31 commands**（數量可能隨版本變動）。

## 3) 安裝方式（ECC 提供的兩條路）
### A) 以 Claude Code Plugin 安裝（推薦）
README 提供的指令流程：
- `/plugin marketplace add affaan-m/everything-claude-code`
- `/plugin install everything-claude-code@everything-claude-code`

但 **rules** 無法透過 plugin 自動分發，仍需另外裝。

### B) 手動安裝 / 用安裝器
repo 內含 `install.sh`：
- 目標：把 `rules/` 以「保留 common/ + language/ 目錄結構」方式拷到目標（預設 `~/.claude/rules/`）
- 支援多語言選擇（typescript/python/golang…）
- 另支援 `--target cursor`（把規則/代理/技能/指令/MCP 安裝到 `.cursor/`）

此外還有一個重要 skill：`skills/configure-ecc/`
- 目的：互動式安裝精靈（引導選擇要裝哪些 skills/rules，並做路徑/覆蓋檢查）

## 4) Agents（子代理）概念與清單
ECC 的 agents 是「可委派、可限權」的角色化子代理，常見用途：規劃、架構、TDD、Code Review、安全檢查、build error 排查、E2E。

本機 `agents/` 看到的 13 個：
- architect
- planner
- tdd-guide
- code-reviewer
- security-reviewer
- build-error-resolver
- e2e-runner
- refactor-cleaner
- doc-updater
- go-reviewer / go-build-resolver
- python-reviewer
- database-reviewer

（每個 agent 都是一份可直接拿來當「分工角色」的 prompt/規範。）

## 5) Commands（slash 指令）概念與清單
ECC 把常用工作流包成可直接呼叫的 commands（.md），例如：
- `/plan`：規劃
- `/verify`：驗證流程
- `/tdd`、`/e2e`、`/test-coverage`：測試相關
- `/code-review`：審查
- `/refactor-clean`：清理/重構
- `/sessions`：session 管理
- `/pm2`、`/multi-plan`、`/multi-execute`、`/multi-backend`、`/multi-frontend`：多服務/多代理編排
- `/setup-pm`：套件管理器設定/偵測

## 6) Hooks（事件觸發自動化）
ECC 的 hooks（見 `hooks/README.md` 與 `hooks/hooks.json`）重點：
- PreToolUse / PostToolUse：工具前後掛勾
- Stop / SessionStart / SessionEnd / PreCompact：生命週期掛勾

範例包含：
- 強制提醒/要求在 tmux 下跑長時間命令
- 阻擋亂寫 `.md/.txt`（避免 repo 產生垃圾文件）
- 編輯後自動 format、跑 tsc、檢查 console.log
- Session 結束時做狀態保存（銜接「記憶持久化」與「持續學習」）

## 7) Skills（工作流/領域知識）概覽
skills 以「框架/語言」「資料庫」「工作流/品質」為主。
本機 `skills/` 看到的部分例子：
- 語言/框架：python-patterns、django-*、springboot-*、golang-*、frontend-patterns、backend-patterns
- 品質流程：tdd-workflow、verification-loop、security-review、security-scan、eval-harness、strategic-compact
- 記憶/學習：continuous-learning、continuous-learning-v2、iterative-retrieval
- 工具型：configure-ecc（互動安裝精靈）

值得注意的幾個「系統級」skill：
- **configure-ecc**：讓使用者用互動方式挑選要安裝的技能與規則
- **continuous-learning / v2**：把 session 中「可重用的模式」萃取成 skill（降低重複 prompt 成本）
- **strategic-compact**：對 context/window 管理提出操作建議

## 8) 文件與翻譯
repo 提供多語言文件，包含繁中：
- `docs/zh-TW/README.md`
- `docs/zh-TW/agents/*`
- `docs/zh-TW/commands/*`
- `docs/zh-TW/rules/*`
- `docs/zh-TW/skills/*`

## 9) 我覺得它對你（OpenClaw）最有價值的點
以你目前的使用情境來看，ECC 可以當作：
1) **「工作流範本庫」**：把常用任務流程（規劃、驗證、review、測試、重構）固定化
2) **「可複用的提示工程資產」**：agents/skills/commands 很適合作為你自己的標準作業程序
3) **「上下文/記憶管理的參考實作」**：hooks + continuous learning 的組合，有不少可移植的設計思路

## 10) 下一步建議（你如果要用它）
- 你要把 ECC 當「參考」還是要把它的 rules/skills 真正裝到你的環境？
  - 參考：我們只挑幾個重點 skill/agent 轉成 OpenClaw 的工作規範
  - 真正安裝：依 README/`install.sh`/`configure-ecc` 走一次，把 rules/skills 匯入你的 Claude Code（如果你有在用）

---

> 本報告只涵蓋 repo 的整體架構與重點；若你希望我針對某個 skill（例如 `continuous-learning-v2` 或 `verification-loop`）做「逐段拆解 + 適用情境 + 可以怎麼移植到 OpenClaw」的深入報告，告訴我你想看哪 3 個我再展開。