# workstation

> Frank Lin 的個人開發工作站，基於 [create-agentic-dev-env](https://github.com/franKobayasi/agentic-dev-env)（ADE 框架）建立。把服務清單、流程慣例、產品規格與開發方法集中在這個 repo，注入本機的 Forge，讓每次開的 Claude Code session 都用同一套上下文與做事流程。

本 repo 是 ADE，知識與 skill 的唯一事實來源：

- **知識**（`hub/knowledge/`）：服務清單、流程慣例、產品規格與 PRD，agent 需要時自己讀
- **方法**（`hub/skills/`）：`ade-*` skill，把開需求、開發、交付、維護文件的做法寫成 agent 照著跑的流程

框架本身的設計與完整說明見上游 repo；本檔只寫這台工作站的用法。

---

## ADE 與 Forge

| | 是什麼 | 怎麼修改 |
| --- | --- | --- |
| **ADE**（本 repo） | 知識與 skill 的唯一事實來源 | 分支加 PR；在 Forge 中由 `ade-contribute` 引導 |
| **Forge** | 本機開 Claude Code 的目錄，含 ADE 的唯讀副本與全部 skill，服務 repo 放在其中的 `workspaces/` | 副本不手改，`update` 會整個覆蓋；`workspaces/` 下的 repo 照平常方式開發 |

```
  ADE  ──── init / update ────▶  Forge 的唯讀副本  ─────────▶  agent 讀取使用
     ▲                                                              │
     └──────────  PR（ade-contribute 引導） ◀───────────  發現過期 / 要新增
```

> [!IMPORTANT]
> session 一律從 Forge 的根目錄開啟。在 `workspaces/<service>/` 裡開 Claude Code 會載不到 ade skill 與知識庫。

---

## 建立 Forge

找一個目錄當 Forge，一台機器一個，在該目錄執行：

```sh
pnpm dlx "git+ssh://git@github.com/franKobayasi/workstation.git" init
```

init 建立 `CLAUDE.md` 的 managed 區段、`.claude/ade/knowledge/`、`.claude/skills/ade-*/`、`workspaces/` 與 `.ade.json`，既有檔案不覆蓋。已有固定放 repo 的資料夾時，加上 `--workspaces <路徑>`，`workspaces/` 會建成指向它的 symlink。

日後更新：在 Forge 說「更新 Forge」（`ade-update`），或把上面指令的 `init` 換成 `update`。session 開始時會比對 `.ade.json` 的 `commit` 與遠端 HEAD，落後時提醒。

### SSH 與 GitHub 帳號

本 repo 是 public，讀取（init、update、`git ls-remote`）用 `git@github.com` 即可。推送需要 franKobayasi 帳號的金鑰，本機以 SSH alias `frankobayasi` 對應：

- `workspaces/workstation/` 工作副本的 remote 設為 `git@frankobayasi:franKobayasi/workstation.git`，否則 push 會因權限被拒
- `gh` 目前登入的帳號不是本 repo 的協作者時，`gh pr create` 會失敗；改用 `gh auth switch` 切到 franKobayasi，或開 push 回應的網址手動建 PR

---

## 常用場景

在 Forge 開 Claude Code，用自然語言說即可觸發。完整清單隨時說「有哪些 skill」（`ade-help`）查詢，各 skill 細節見 `hub/skills/<name>/SKILL.md`。

| 想做的事 | 說一句 | 觸發 |
| --- | --- | --- |
| 拿到最新知識與 skill | 「更新 Forge」 | `ade-update` |
| 查看已註冊的服務 | 「列出服務」 | `ade-list-service` |
| 註冊新服務 | 「新增服務」 | `ade-add-service` |
| 把想法整理成需求文件 | 「建 PRD」 | `ade-create-prd` |
| 把定案的 PRD 落進規格 | 「PRD 轉 spec」 | `ade-prd-to-spec` |
| commit 與發 PR | 「commit」／「發 MR」 | `ade-commit`／`ade-ship` |
| 開發完成後對齊規格 | 「對齊 spec」 | `ade-align-spec` |
| 檢查規格與實作是否漂移 | 「規格還對嗎」 | `ade-spec-audit` |
| 修正或補充 ADE 的知識 | 「同步到 ADE」「把這個記回知識庫」 | `ade-contribute` |
| 把做法定成慣例 | 「以後都這樣做」 | `ade-add-process` |
| 新增 skill | 「新增 skill」 | `ade-add-skill` |
| 查看或修改 Forge 設定 | 「Forge 設定」 | `ade-config` |

需求的生命週期：`ade-create-prd` → `ade-prd-to-spec`（加上尚未實作標記）→ 開發 → `ade-align-spec`（移除標記、PRD 標為已實作）；hotfix 等計畫外變更由 `ade-spec-audit` 補抓。

---

## 本 repo 結構

```
hub/                     # init／update 注入每個 Forge 的內容
├── CLAUDE.section.md    # Forge 中 CLAUDE.md 的 managed 區段
├── knowledge/           # Forge 的 .claude/ade/knowledge/，導覽見 hub/knowledge/README.md
│   ├── CONTEXT.md       # 詞彙表
│   ├── services/        # 服務描述檔，一服務一份 yaml
│   ├── process/         # 流程與慣例，含建立知識、撰寫 PRD 與 ADR 的規則
│   ├── specs/           # 產品規格
│   └── prd/             # 一次開發一份的需求文件
└── skills/              # Forge 的 .claude/skills/
.claude/skills/          # 本 repo 內可用的 skill：實體目錄是本 repo 專用，symlink 指向 hub/skills/ 中標 ade-scope: both 的
.githooks/pre-commit     # commit 前產生 ADR 與知識庫的索引、執行知識庫檢查
docs/adr/                # 不注入 Forge 的決策紀錄
UPSTREAM-CANDIDATES.md   # 本地模式下 [upstream-candidate] 的落點
```

## 維護原則

- **副本唯讀**：Forge 中的內容會被 update 覆蓋，所有修改回到本 repo 走 PR
- **知識必須通過檢查**：建立或修改知識依 `ade-build-knowledge`。clone 後執行 `git config core.hooksPath .githooks` 與 `pnpm install`，pre-commit hook 會產生 `INDEX.md` 並執行 `node cli.js check`
- **機制回饋上游、內容不外流**：skill 寫法、模板、流程等機制改良用 `ade-feedback-upstream` 回饋給上游框架；`knowledge/` 下的內容不外流
