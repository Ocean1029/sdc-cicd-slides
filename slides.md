---
theme: default
title: CI/CD 與部署自動化
info: |
  ## SDC SRE Workshop — CI/CD 篇
  Day 1 · 持續整合與部署
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
---

# CI/CD 與部署自動化

SDC SRE Workshop · Day 1

<div class="pt-12">
  <span class="text-sm opacity-60">按空白鍵或右方向鍵開始 →</span>
</div>

---
layout: section
---

# 1.1 軟體開發的多人協作

---

# 所有人都直接 push main 的問題

多人共用一條 `main` 分支、直接 push，很容易出事：

- 改動沒被 review 過就直接進 `main`，bug 沒人擋
- 並行修同一段程式碼很容易 conflict，merge 過程沒規範
- 有 bug 的版本一進到 `main` 就會影響上線服務
- revert 時 history 已被多人交錯 commit 打亂，難以乾淨回溯

---

# 解法：PR 流程

- 各自開 feature branch 開發
- 寫到段落發 **Pull Request**
- 其他人看 diff、留 comment、提 review
- 通過了再 merge 回 `main`
- `main` 永遠保持「隨時可以部署」的狀態

---

# Pull Request 頁面

<div class="flex justify-center mt-2">
  <img src="/github-pull-request.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

標題、Reviewer / Assignee、Conversation / Commits / **Checks** / Files changed

</div>

---

# 工作流程

```
main 永遠穩定
  │
  ├── feature/add-user-auth
  │     ├── commit
  │     ├── commit
  │     └── PR ──▶ review ──▶ merge ──▶ main
  │
  └── feature/fix-login
        └── ...
```

SDC 內部專案（例如 Core System backend）就是這個模式。

---
layout: section
---

# 1.2 什麼是 CI/CD？

---

# CI/CD 全名

**Continuous Integration / Continuous Deployment**

把編譯、測試、部署這些原本靠人手動做的步驟，**自動化**成一條 pipeline：

- 程式碼一推上去，CI 自動跑檢查
- 通過之後 CD 自動部署

---

# 解決什麼問題？

**品質**

- 每次變更都跑同一套檢查 → 不會有「我電腦上可以跑啊」
- 部署流程寫死 → 不會漏步驟

**速度**

- 重複工作交給機器 → 工程師專心做產品
- 變更變小、發版變頻 → 風險變低、定位變快

---

# CI vs CD

| 階段 | 全名 | 負責 |
|------|------|------|
| **CI** | Continuous Integration | 合併**前**自動跑品質檢查 |
| **CD** | Continuous Deployment | 合併**後**自動部署到對應環境 |

- CI 怎麼實作 → 第三章
- CD 怎麼實作 → 第四章

---

# 對照：有沒有 CI/CD 的差別

| 沒有 CI/CD | 有 CI/CD |
|-----------|---------|
| 忘記跑測試就 merge，壞掉的 code 進 `main` | 沒過測試就無法 merge |
| 「在我電腦上可以跑」 | 大家在同一套 runner 環境跑 |
| revert 不回去、history 一團亂 | 每次變更都小、都驗證過、容易 rollback |

---

# 常見 CI/CD 工具

- **GitHub Actions** — 本課重點，與 GitHub 深度整合
- **GitLab CI/CD** — GitLab 內建，跟 GitLab 生態系緊密
- **Jenkins** — 老牌、可客製化、設定複雜
- **CircleCI** — 雲端服務，速度快、易用
- **Travis CI** — 早期廣為使用，對 OSS 友善

---
layout: section
---

# 第二章
## GitHub Actions 基礎

---
layout: section
---

# 2.1 事前準備

---

# 你需要什麼

- **GitHub 帳號**（沒有的話到 [github.com](https://github.com) 註冊）
- 一個新的 public repo：`github-actions-lab`
- Clone 到本機

```bash
git clone https://github.com/<your-username>/github-actions-lab.git
cd github-actions-lab
```

---
layout: section
---

# 2.2 先跑一個 Workflow

---

# Workflow 檔案放哪裡？

GitHub Actions 的 workflow 一律放在這個目錄：

```bash
mkdir -p .github/workflows
```

```
my-repo/
├── .github/
│   └── workflows/
│       └── hello.yml          ← 這裡
├── main.go
└── go.mod
```

---

# 第一個 workflow 要做什麼

讓 push 到 `main` 之後自動：

1. 印一段 hello 訊息
2. 列出 runner 的作業系統 / Go / Docker 版本
3. 列出這次 workflow 觸發時的 GitHub 資訊（repo、branch、commit、誰、什麼事件）

每件事各寫成一個 step，全部放在同一個 job 底下。

---

# `.github/workflows/hello.yml`

```yaml
name: Hello GitHub Actions

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  greeting:
    runs-on: ubuntu-latest
    steps:
      - name: Say Hello
        run: echo "Hello, GitHub Actions!"

      - name: Show System Info
        run: |
          echo "OS: $(uname -s)"
          echo "Go Version: $(go version)"
          echo "Docker Version: $(docker --version)"

      - name: Show GitHub Context
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Actor: ${{ github.actor }}"
```

> 一行一行的意思待會兒解釋，現在不用急著看懂。

---

# Push 上去

```bash
git add .github/workflows/hello.yml
git commit -m "Add first GitHub Actions workflow"
git push origin main
```

接著打開 GitHub，點上方的 **Actions** 分頁。

---

# Actions 分頁

<div class="flex justify-center mt-2">
  <img src="/github-actions-tab.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

剛剛的 commit message 變成一筆 workflow run

</div>

---

# 點進去看 Step Logs

<div class="flex justify-center mt-2">
  <img src="/github-actions-step-logs.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

每個 step 都有打勾。`Show System Info` 可以看到 runner 預裝的 Go / Docker 版本。

</div>

---
layout: section
---

# 2.3 Workflow 結構元素

---

# 五個元素

```
Workflow (.github/workflows/hello.yml)
├── Event           on: push / workflow_dispatch
├── Job: greeting
│   ├── Runner      runs-on: ubuntu-latest
│   ├── Step 1      run: echo "Hello"
│   ├── Step 2      run: echo "OS: ..."
│   └── Step 3      run: echo "Repository: ..."
```

外加一個語法工具：`${{ ... }}` 表達式，用來讀 GitHub 自動提供的資訊。

---

# Workflow

一個定義自動化流程的 **YAML 檔案**。

- 必須放在 **`.github/workflows/`** 目錄
- 副檔名 `.yml` 或 `.yaml`
- 一個 repo 可以有**多個** workflow

```
my-repo/
├── .github/
│   └── workflows/
│       ├── ci.yml          # workflow 1
│       ├── pr-check.yml    # workflow 2
│       └── release.yml     # workflow 3
```

---

# Event — 觸發條件

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

| Event | 觸發時機 |
|-------|---------|
| `push` | 推程式碼到指定分支時 |
| `pull_request` | 建立或更新 PR 時 |
| `schedule` | 定時排程（cron 語法） |
| `workflow_dispatch` | 手動觸發 |
| `release` | 建立 Release 時 |

---

# Job — 執行單元

```yaml
jobs:
  greeting:
    runs-on: ubuntu-latest
```

- 一個 workflow 可以有多個 job
- **預設平行執行**
- 想要排順序要靠 `needs:`（第三章會看到）

---

# Step — 執行步驟

每個 job 由一連串 step 組成，**依序**在同一個檔案系統上執行。

兩種寫法：

```yaml
# run — 自己執行 shell 指令
- run: echo "Hello"

# uses — 引用別人寫好的 Action
- uses: actions/checkout@v4
```

`uses` 格式：`{owner}/{repo}@{version}`

> 建議鎖版本（`@v4`），不要用 `@main`。常用 Action 在 [GitHub Marketplace](https://github.com/marketplace?type=actions) 找。

---

# 表達式 `${{ ... }}` 與 Context

讀取 GitHub 在執行時自動提供的資訊：

```yaml
- run: echo "Hello, ${{ github.actor }}!"
```

`github` context 常用欄位：

| 表達式 | 說明 | 範例 |
|--------|------|-----|
| `github.repository` | Repo 全名 | `octocat/hello-world` |
| `github.ref_name` | 分支或 tag 名稱 | `main` |
| `github.sha` | commit SHA | `abc123...` |
| `github.actor` | 觸發者 | `octocat` |
| `github.event_name` | 觸發事件 | `push` |

> 還有一個 `secrets` context，第四章部署到 Fly.io 會用到。

---

# Runner — 執行機器

```yaml
runs-on: ubuntu-latest
```

| 標籤 | 作業系統 |
|------|---------|
| `ubuntu-latest` | Ubuntu LTS — 預設選這個 |
| `windows-latest` | Windows Server |
| `macos-latest` | macOS — iOS 開發才需要 |

GitHub-hosted runner 預裝了 Go、Node、Python、Git、Docker、各種雲端 CLI，不用自己 `apt install`。

> 需要特殊環境可以用 self-hosted runner，本課全部用 GitHub-hosted。

---
layout: section
---

# 2.4 手動觸發與失敗處理

---

# 手動觸發 Workflow

`on: workflow_dispatch` 讓你不 push 程式碼也可以觸發。

1. Actions 分頁 → 找到 **Hello GitHub Actions**
2. 右上角 **Run workflow** 按鈕
3. 選分支、按綠色 **Run workflow**

<div class="mt-2 flex justify-center">
  <img src="/github-actions-run-workflow.png" class="max-h-[16rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 text-center mt-2">

這次的 `Event` 欄位會顯示 `workflow_dispatch`（不是 `push`）

</div>

---

# 故意讓 Pipeline 失敗

CI 失敗是常態，學會看失敗訊息是 SRE 基本功。

下面這個 workflow 故意藏了一個 bug，會讓 pipeline 跑到中間就失敗：

```yaml
name: Repo Info
on:
  push:
    branches: [main]

jobs:
  fetch-info:
    runs-on: ubuntu-latest
    steps:
      - name: Fetch repository info
        run: curl -s "https://api.github.com/repos/${{ github.repository }}" > repo.json

      - name: Show repository name
        run: jq -r .full_name repo.json

      - name: Show star count
        run: |
          STARS=$(jq -e .stars repo.json)
          echo "This repo has $STARS stars"

      - name: Show description
        run: jq -r .description repo.json
```

---

# 失敗時的 Summary

<div class="flex justify-center mt-2">
  <img src="/github-actions-failed-summary.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-2 text-center">

紅色叉叉、`Failure` 狀態。`Show star count` 之後的 `Show description` 沒被執行。

</div>

---

# 預設行為

> 任何一個 step 失敗（exit code 不是 0），**後面的 step 全部不會跑**，整個 job 標記為 failure。

點開失敗的 step 看 log：

<div class="mt-2 flex justify-center">
  <img src="/github-actions-failed-step-detail.png" class="max-h-[18rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-2 text-center">

`jq` 找不到 `stars` 欄位 — 那正確欄位叫什麼？

</div>

---

# 直接看 GitHub API

從 `Fetch repository info` step 的 log 找到 `curl` URL，瀏覽器打開：

<div class="mt-2 flex justify-center">
  <img src="/github-api-json-stargazers.png" class="max-h-[20rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-2 text-center">

正確欄位是 `stargazers_count`，不是 `stars`。

</div>

---

# 修正後

把 `.stars` 改成 `.stargazers_count`，commit、push：

<div class="mt-2 flex justify-center">
  <img src="/github-actions-fixed-run.png" class="max-h-[20rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-2 text-center">

「看失敗訊息 → 對照原始資料 → 修正 → 重跑」是日常會反覆做的循環。

</div>

---
layout: end
---

# 第二章結束

下一章：**Go 專案 CI Pipeline**

---
layout: section
---

# 第三章
## Go 專案 CI Pipeline

---
layout: section
---

# 3.1 什麼是 CI？

---

# CI 三件事

| 檢查 | 做什麼 | 不做會怎樣 |
|------|-------|----------|
| **Lint** | 風格 / 潛在錯誤檢查 | 程式碼亂、隱藏 bug 變多 |
| **Test** | 跑單元測試 | 改了 A 結果 B 壞掉 |
| **Build** | 把程式碼編譯起來 | 連跑都跑不起來還合進 `main` |

每次變更都自動執行；任何一個失敗就擋住合併。

---
layout: section
---

# 3.2 範例專案

---

# `CI-CD/examples/sample-app/`

一個簡單的 Go HTTP API：

```
sample-app/
├── main.go            # entry point
├── handler.go         # handlers
├── handler_test.go    # unit tests
├── go.mod
├── Dockerfile
└── .golangci.yml
```

| Endpoint | 回傳 |
|----------|------|
| `GET /` | `Hello, GitHub Actions!` |
| `GET /health` | `{"status":"ok"}` |
| `GET /version` | `{"version":"dev"}` |

---

# 在本機跑跑看

```bash
cd CI-CD/examples/sample-app
go run .
```

<div class="flex justify-center mt-4">
  <img src="/sample-app-running.png" class="max-h-[16rem] rounded shadow" />
</div>

接著我們就要替它建 CI pipeline。

---
layout: section
---

# 3.3 完整 CI Workflow

---

# Pipeline 三階段

```
        ┌──────┐
        │ push │
        └───┬──┘
     ┌──────┴──────┐
     ▼             ▼
  ┌──────┐     ┌──────┐
  │ Lint │     │ Test │   ← 平行
  └───┬──┘     └───┬──┘
     └──────┬──────┘
            ▼
        ┌───────┐
        │ Build │   ← 等前兩個過才跑
        └───────┘
```

---

# 觸發條件

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

兩種角色：

- **`pull_request`**：合併**前**的攔截
- **`push`**：合併**後**的最終驗證

> `pull_request` 跑的不是 PR 分支本身，而是「合併進 main 之後的模擬 commit」 — 衝突跟相容性問題在 PR 階段就會被抓到。

---

# Lint Job

```yaml
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-go@v5
      with:
        go-version: '1.24'
    - name: Run golangci-lint
      uses: golangci/golangci-lint-action@v6
      with:
        version: latest
```

- `actions/checkout` — 把程式碼下載到 runner
- `actions/setup-go` — 安裝 Go
- `golangci-lint-action` — 跑 lint，自動套用 `.golangci.yml`

---

# Test Job

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-go@v5
      with:
        go-version: '1.24'
    - name: Verify dependencies
      run: go mod verify
    - name: Run tests
      run: go test -v -race -coverprofile=coverage.out ./...
    - name: Show coverage
      run: go tool cover -func=coverage.out
    - name: Upload coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coverage.out
```

---

# `go test` 的 Flag

| Flag | 用途 |
|------|------|
| `-v` | 詳細輸出，每個測試函式都印 |
| `-race` | Data race detector，抓 goroutine 競爭問題 |
| `-coverprofile=coverage.out` | 產生覆蓋率報告 |
| `./...` | 遞迴跑所有子套件的測試 |

> 覆蓋率：核心業務邏輯一般建議 80% 以上。

---

# 為什麼每個 job 都要重跑 checkout / setup-go？

> **每個 job 跑在全新的 runner 上**，前一個 job 下載的程式碼、安裝的工具，到下一個 job 就不存在了。

也因此跨 job 傳檔案要靠 **Artifact**：

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage.out
```

- Actions 頁面可以手動下載
- 同一個 workflow 的另一個 job 可用 `download-artifact` 取回
- 預設保留 90 天

---

# Build Job

```yaml
build:
  needs: [lint, test]            # ← 等前兩個 job 都過才跑
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-go@v5
      with:
        go-version: '1.24'
    - name: Build binary
      run: go build -o bin/app .
    - name: Upload binary
      uses: actions/upload-artifact@v4
      with:
        name: app-binary
        path: bin/app
```

---

# `needs:` 控制依賴

- **沒寫 `needs`** → 跟其他 job 平行執行（像 `lint` / `test`）
- **有寫 `needs`** → 等指定的 job 全部成功才開始
- **`needs` 中有任何一個失敗** → 這個 job 直接被跳過

---

# 為什麼 build 要等 lint + test？

如果 lint 或 test 沒過，build 出來的 binary 也用不上。

- 節省 CI 資源（不浪費時間在必然沒用的建置）
- 失敗訊號明確（知道是 lint / test，不是 build）
- 邏輯正確（品質有檢查過才值得 build）

> Build 出來的 binary 同樣用 `upload-artifact` 上傳，後續 deploy 直接拿，不用再編一次。

---
layout: end
---

# 第三章結束

下一章：**部署到雲端**

---
layout: section
---

# 第四章
## 部署到雲端平台

---
layout: section
---

# 4.1 什麼是 CD？

---

# CD 在做什麼

把通過 CI 檢查的程式碼**自動搬到實際運行的環境**：

- 打包（Docker image / binary）
- 部署到對應環境
- 後續處理（通知、監控）

> 從 push 到使用者看到新功能，全自動，沒有人攔在中間。

這種模式對「自動化測試的信心」要求很高，測試是你唯一的安全網。

---
layout: section
---

# 4.2 Environments 與 Secrets

---

# GitHub Environments

讓你定義不同的部署環境（`staging`、`production`），各自獨立設定：

- **Secrets** — 不同環境不同的 API key
- **Protection Rules** — 例如 production 需手動核准
- **Deployment branches** — 限制哪個分支能部署

---

# Secret 放哪裡？

GitHub 有**兩種** secret，新手常搞混：

| 放法 | 設定位置 | 適用 |
|------|---------|-----|
| **Repository secret** | Settings → Secrets and variables → Actions | 所有 workflow / job 都可讀。**最簡單的預設** |
| **Environment secret** | Settings → Environments → `<env>` → Secrets | 只有宣告 `environment: <name>` 的 job 可讀。可搭配 protection rules |

兩種都用 `${{ secrets.XXX }}` 讀。本章只用 `FLY_API_TOKEN`，repository secret 就夠。

---

# Secrets 安全守則

- **永遠不要硬寫進程式碼**，一律用 `${{ secrets.XXX }}`
- **Fork PR 不能存取 secrets**（GitHub 自動防護）
- 成員離開時相關 secrets 立刻輪換

```yaml
# Good
env:
  API_KEY: ${{ secrets.API_KEY }}

# Bad — NEVER do this
env:
  API_KEY: "sk-abc123xyz789"
```

---
layout: section
---

# 4.3 實戰：部署到 Fly.io

---

# 為什麼選 Fly.io？

[Fly.io](https://fly.io) 是開發者友善的容器平台：

- **讀你的 Dockerfile** — 不用自己管 registry
- **一個指令完成部署** — `flyctl deploy`
- **免費額度足夠實驗**
- **全球邊緣節點**

對這個 Go HTTP API 範例來說，是最輕量的選擇。

> Fly.io 目前要求綁信用卡（即使只用免費額度）。沒卡的話可以只看 workflow 學概念。

---

# 前置準備（本機）

1. 註冊 Fly.io、Billing 綁信用卡
2. 安裝 `flyctl`
3. `flyctl auth login`
4. `flyctl launch`（互動式，會問 app name / region / DB / deploy now）
   - App name 全域唯一，建議 `<你的名字>-sample-app`
   - Region 選 `nrt` 或 `hkg`
   - Postgres / Redis 選 No
   - Deploy now? 選 No（CI 會做）
5. 產生 `fly.toml`，**記得 commit**
6. `flyctl tokens create deploy` → 拿 token
7. token 存成 GitHub repo secret `FLY_API_TOKEN`

---

# Deploy Workflow

```yaml
name: Deploy to Fly.io

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Setup flyctl
        uses: superfly/flyctl-actions/setup-flyctl@master

      - name: Deploy to Fly.io
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

---

# 三個重點

**`environment: production`**

- 告訴 GitHub 這個 job 用 `production` 環境
- 第一次看到的 environment 會自動建，不用事先設
- 想加「手動核准」就在 Settings → Environments 設 Required reviewers

**`flyctl deploy --remote-only`**

- 讀 `fly.toml` 跟 `Dockerfile`
- 在 Fly.io 的 builder 上 build image，**不在 runner 本地建**
- 省時省資源

**`FLY_API_TOKEN`**

- 透過 env 傳給 `flyctl`，自動認證

---
layout: section
---

# 4.4 補充：多環境部署

> 進階補充。前面的單環境流程已足夠完成基本 CD pipeline。

---

# Staging → Production

```
┌────────┐     ┌────────────┐     ┌──────────────┐
│  Code  │────▶│  Staging   │────▶│  Production  │
│  Merge │     │  (auto)    │     │  (manual)    │
└────────┘     └────────────┘     └──────────────┘
```

Fly.io 慣例：每個環境一個 app（`myapp-staging`、`myapp-prod`），用 `--app` 切換。

---

# 兩階段 Workflow（節錄）

```yaml
jobs:
  deploy-staging:
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master

      - run: flyctl deploy --remote-only --app myapp-staging
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

      - name: Smoke test
        run: |
          for i in 1 2 3 4 5; do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://myapp-staging.fly.dev/health)
            if [ "$STATUS" = "200" ]; then exit 0; fi
            sleep 10
          done
          exit 1

  deploy-production:
    needs: deploy-staging         # ← staging 過了才能跑
    environment: production       # ← 設 Required reviewers 就會卡手動核准
    steps:
      - run: flyctl deploy --remote-only --app myapp-prod
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

---

# 關鍵設計

1. **Staging 自動部署** — push 後直接上 staging
2. **Smoke test** — retry 迴圈避免新 instance 還沒起好就誤判失敗
3. **Production 手動核准** — `environment: production` + protection rules
4. **`needs:`** — production 一定在 staging 成功之後

---

# 設定 Protection Rules

Settings → Environments → `production`：

- 勾 **Required reviewers**，加核准人
- 可選 **Wait timer**（例如 5 分鐘冷靜期）

設好之後，workflow 跑到 `deploy-production` 會**暫停等待核准**。

---
layout: section
---

# 工作坊回顧

---

# 完整 CI/CD 全景

```
開發者 push
  │
  ▼
┌──────────────── CI (ch3) ────────────────┐
│  ┌──────┐  ┌──────┐                      │
│  │ Lint │  │ Test │   ← parallel         │
│  └──┬───┘  └──┬───┘                      │
│     └────┬────┘                          │
│          ▼                               │
│      ┌───────┐                           │
│      │ Build │                           │
│      └───────┘                           │
└──────────────────────────────────────────┘
  │
  │ merge to main
  ▼
┌──────────────── CD (ch4) ────────────────┐
│              ┌─────────────┐             │
│              │   Deploy    │             │
│              │  to Fly.io  │             │
│              └─────────────┘             │
│  Advanced: staging → smoke → production  │
└──────────────────────────────────────────┘
```

---

# 原本的痛點 vs 現在

| 原本的痛點 | 現在怎麼解 |
|-----------|-----------|
| 手動測試每次都花好久 | push 後 CI 自動跑 lint / test / build |
| 合併之後才發現壞掉 | PR 階段就跑一次 CI，壞掉的合不進來 |
| 部署又忘了步驟 | `flyctl deploy` 一個指令，前置都寫進 workflow |
| 誰都可以亂部署 production | Environment protection rules 強制手動核准 |
| API key 散落各處 | 統一放 GitHub Secrets，log 自動遮蔽 |

---

# 沒講到的東西

- Release 自動化（`on: push: tags: ['v*']` + `softprops/action-gh-release`）
- Image 掃描（Trivy、Snyk）
- GitOps（ArgoCD、Flux）
- 監控告警（下一個 workshop 的 Prometheus！）

實務遇到再深入即可。

---

# 延伸資源

- [GitHub Actions 官方文件](https://docs.github.com/en/actions)
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [Fly.io Docs](https://fly.io/docs/)
- [The Twelve-Factor App](https://12factor.net/)
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)

---
layout: end
---

# CI/CD 篇結束

下午：**Prometheus**

謝謝大家！
