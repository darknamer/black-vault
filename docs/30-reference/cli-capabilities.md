# Git BlackVault — สิ่งที่ CLI ทำได้ตอนนี้

> ย้ายมาจาก [README](../../README.md) §13 สิ่งที่ CLI ทำได้ตอนนี้ — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้

## 13. สิ่งที่ CLI ทำได้ตอนนี้

สรุปจาก **command tree จริง** ของ `black-vault-cli` (`cmd/root.go` + ไฟล์ของแต่ละคำสั่ง) และเมธอดที่มีจริงใน `black-vault-lib` — ไม่ใช่โรดแมป
รายละเอียดของแต่ละ flag ดู [README ของ black-vault-cli](../../../black-vault-cli/README.md)

### 13.1 ภาพรวมกลุ่มคำสั่ง

| กลุ่ม | คำสั่ง | ทำอะไร |
| --- | --- | --- |
| **ตั้งค่า** | `init`, `config get/set` | สร้าง/แก้ `config.yaml` ทีละคีย์ |
| **บัญชี** | `account list/add/select/remove/edit` | จัดการหลายบัญชี git + NAS ในลิสต์เดียว |
| **โปรไฟล์** | `workspace profile list/add/select/remove/edit/rename` | โปรไฟล์ workspace ผูกกับบัญชี กำหนด root บน disk |
| **Repo metadata** | `workspace repo list/sync/rename` | metadata ของ repo ต่อโปรไฟล์ (เก็บใน `config.yaml`) |
| **Lifecycle** | `open`, `close`, `status` | เปิด (clone) / ปิด (ลบ) / ดูสถานะ |
| **Git** | `git <subcommand>` | คำสั่ง git ใน workspace (ดู [13.4](#134-คำสั่ง-git-ใน-workspace)) |
| **gRPC** | `serve` | เปิด gRPC server ให้ GUI |
| **ติดตั้ง/วินิจฉัย** | `install`, `doctor`, `install-git`, `version`, `completion` | bootstrap, ตรวจสถานะ, เตรียม portable git, เวอร์ชัน, shell completion |
| **develop เท่านั้น** | `nas sync` | solution NAS (ต้อง `BLACKVAULT_DEV=1`) |

### 13.2 Workspace lifecycle

**`open [group/repo]`** — clone ลง `workspaces/<profile>/<group>/<repo>/`

| Flag | ทำอะไร |
| --- | --- |
| `--shallow` | shallow clone |
| `--branch <name>` | clone เจาะ branch (ข้าม default branch policy) |
| `--detached` | โหมด detached / review (ข้าม policy เช่นกัน) |
| `--all` | เปิด**ทุกโปรเจกต์**จาก remote โดยข้ามตัวที่ active อยู่แล้ว |
| `--limit <n>` | ใช้กับ `--all` — `-1` = ไม่จำกัด, `0` = ไม่เปิดเลย |
| `--refresh` | ใช้กับ `--all` — ข้าม cache รายการ remote |
| `--account <id>` / `--profile <name>` | เลือกบัญชี/โปรไฟล์เฉพาะคำสั่งนี้ (ไม่เปลี่ยนค่า active ถาวร) |
| `--clone-mode https\|https-auth\|ssh` | เลือก transport (ดีฟอลต์ `https`) |
| `--skip-verify-https` | ข้าม TLS verify ตอน clone (สำหรับ host ที่ใช้ self-signed cert) |

- **ไม่มี flag `--depth`** — ความลึกกำหนดโดย `--shallow`
- **default branch policy:** ทุกครั้งที่ open จะเตรียม `main` (ไม่มีก็ลอง `master`) แล้ว**ถ้ามี `origin/develop` จะจบที่ `develop`** — best-effort, ต้องมี git binary, fetch/pull ล้มไม่ทำให้ open ล้ม

**`close [group/repo]`** — ปิด workspace

| Flag | ทำอะไร |
| --- | --- |
| `--purge` (ดีฟอลต์ `true`) | ลบไฟล์และไดเรกทอรีทิ้ง |
| `--no-purge` | เก็บไฟล์ไว้ ปิดเฉพาะในทะเบียน (ชนะ `--purge` เมื่อสั่งพร้อมกัน) |
| `--all` | ปิดทุก workspace ที่ active |
| `--account` / `--profile` | เหมือน `open` |

- **ไม่มี `--force`** และไม่มีการถามยืนยัน — ลบตรง ๆ ตามปรัชญาของโปรเจกต์
- ปิด path ที่ไม่มีอยู่แล้ว = สำเร็จ (idempotent); path ที่ไม่ใช่ไฟล์/ไดเรกทอรีปกติ (symlink ไป device ฯลฯ) คืน error ชัดเจนแทนการลบเงียบ ๆ
- ลบแล้วเก็บกวาดไดเรกทอรีแม่ที่ว่างเปล่า แต่ไม่ลบ root ของโปรไฟล์และไม่ลบ group ที่ยังมี repo อื่นอยู่

**`status`** — ส่วนหัว (บัญชี / connectivity / โปรไฟล์ / workspace root) + รายการ `ACTIVE` และ `CLOSED` ดูตัวอย่างที่ [5.3](../10-features/mvp-features.md#53-status)

### 13.3 หลายบัญชี และโปรไฟล์ workspace

- **บัญชี:** เก็บ git และ NAS ไว้ใน**ลิสต์เดียว** (`accounts[]` แต่ละตัวมี `type: git|nas`) พร้อมตัวชี้เดียว `active_account`; **id สร้างจาก URL อัตโนมัติ** (ซ้ำได้ด้วย `{url}(2)`) หรือกำหนดเองตอน `add`
  `account list` / `add` / `select` / `remove` / `edit` — บัญชี NAS โผล่เฉพาะโหมด develop
- **โปรไฟล์ workspace:** ตั้งชื่อเองแล้วผูกบัญชีหนึ่งบัญชี; โปรไฟล์ที่ active กำหนดทั้ง **root บน disk** (`workspace.dir/<profile>/`) และบัญชีที่ใช้กับ API + scope ใน registry
  `workspace profile list` / `add` / `select` / `remove` / `edit` / `rename` — `rename` ย้ายทั้งชื่อใน config, metadata repo และโฟลเดอร์บน disk ให้สอดคล้องกัน
  ชื่อเริ่มต้น: บัญชีแรกได้ `git-default`, บัญชีที่เพิ่มทีหลังได้ host slug เช่น `git-github.com`
- **Repo metadata ต่อโปรไฟล์** (คนละชุดกับ SQLite registry — เก็บใน `config.yaml` แก้ด้วยมือได้):
  `workspace repo list` (ดึงสดจาก remote มาเก็บ), `workspace repo sync` (สแกน disk แล้วปรับ `active`/`closed`), `workspace repo rename` (เปลี่ยนชื่อ repo ฝั่ง local — ย้ายโฟลเดอร์ + แก้ config + แก้ registry)
- **สลับชั่วคราวต่อคำสั่ง:** `--account <id>` และ `--profile <name>` บน `open` / `close` / `status` / `nas sync` — ไม่เปลี่ยนค่า active ถาวร

### 13.4 คำสั่ง Git ใน workspace

คำสั่งที่ทำงาน**ในตัว repo** รับ **`[group/repo]`** เป็น argument แรก (เป็น identifier ของ workspace **ไม่ใช่ path**) และต้องเปิด workspace ไว้ก่อน — ยกเว้น `list` กับ `repo-list` ที่เป็นคำสั่งระดับ "รายการ"

```text
blackvault git
├── list [group]              # รายการ repo จาก registry ในเครื่อง (-g, -s active|closed|all, --json)
├── repo-list                 # รายการ repo สดจาก remote (GitLab หรือ GitHub API) (-f, -l, --refresh, --json)
├── add [group/repo] [paths...]   # ไม่ระบุ path = git add -A
├── commit [group/repo] -m "..."  # ดีฟอลต์ add -A ก่อน; --no-add เพื่อข้าม
├── fetch [group/repo]        # -r/--remote, --refspec, --all, --prune
├── pull  [group/repo]        # -r/--remote, -b/--branch, --rebase
├── push  [group/repo]        # -r/--remote, -b/--branch, -u/--set-upstream, --force
├── merge [group/repo] [branch]   # --no-ff, --squash, -m
├── branch
│   ├── create [group/repo] [branch]
│   ├── switch [group/repo] [branch]
│   ├── rename [group/repo] [new] | [group/repo] [old] [new]
│   ├── delete [group/repo] [branch]        # -f/--force (= -D), ลบเฉพาะ local
│   └── set-upstream [group/repo] [upstream] [branch]
├── remote
│   ├── list [group/repo]
│   ├── add [group/repo] [name] [url]
│   ├── remove [group/repo] [name]
│   └── set-url [group/repo] [name] [url]
├── flow
│   ├── init [group/repo]                    # --main, --develop
│   ├── feature start [group/repo] [name]
│   ├── release start  [group/repo] [name]   # --from
│   ├── release finish [group/repo] [name]   # --main, --develop, --tag
│   ├── hotfix  start  [group/repo] [name]   # --from
│   └── hotfix  finish [group/repo] [name]   # --main, --develop, --tag
└── help
```

> คำสั่งกลุ่มนี้**ต้องมี git binary จริง** — go-git fallback ใช้ได้เฉพาะตอน clone เท่านั้น (ดู [requirements.md §12.1](../02-getting-started/requirements.md#121-เครื่องมือที่ต้องมี))
> ยังไม่มี: `git status`, `git log`, `git diff`, `git stash`, `git tag`, `git rebase` และ `flow feature finish`

### 13.5 gRPC (serve)

```bash
blackvault serve            # ดีฟอลต์ 127.0.0.1:50051
blackvault serve --port 51000
```

- **ผูก loopback เท่านั้น** (`127.0.0.1`) ไม่เปิดออกทุก interface
- **TLS** ด้วย self-signed cert ที่สร้างใหม่ทุกเซสชัน (private key อยู่ในหน่วยความจำเท่านั้น)
- **session token** สุ่มแบบ cryptographically secure ต่อการรัน — บังคับตรวจผ่าน metadata `x-blackvault-token` **ทุก RPC** ทั้ง unary และ stream; ผิด/ไม่มี = `Unauthenticated`
- **dynamic port:** ถ้าพอร์ตที่ขอไม่ว่างจะไล่หาตัวถัดไป (สูงสุด 20 พอร์ต) แล้วเขียนพอร์ตที่ใช้จริงลง `~/.blackvault/grpc_port`
- ปิดแบบ graceful (Ctrl-C / SIGTERM) แล้วลบไฟล์เซสชันทั้งสามทิ้ง

**RPC ที่ register แล้วครบ 10 ตัว:** `Ping`, `OpenWorkspace`, `CloseWorkspace`, `GetStatus`, `ListRepositories`, `ListAccounts`, `ListWorkspaceProfiles`, `SelectWorkspaceProfile`, `Doctor` และ `ObserveOperations` (server-streaming — สตรีม log ของ operation ให้ console ฝั่ง GUI) พร้อม Health probe

> ⚠️ RPC `CloseWorkspace` ปิดแบบ purge เสมอ — field `force` ใน proto ถูกรับแต่ยังไม่ถูกใช้
> gRPC **ไม่ได้** expose คำสั่ง git หรือ NAS — ส่วนนั้นยังต้องเรียกผ่าน CLI

### 13.6 ติดตั้งและวินิจฉัย

| คำสั่ง | ทำอะไร |
| --- | --- |
| `install` | โหลด config → สร้างไดเรกทอรีที่ขาด → **ping git host จริงทุกบัญชี** → สรุปผ่าน/ไม่ผ่าน (exit ไม่เป็นศูนย์ถ้ามีข้อไหนไม่ผ่าน) |
| `doctor` | วินิจฉัยแบบ**อ่านอย่างเดียว** — config, เวอร์ชัน cli, เวอร์ชัน lib, git executable, จำนวนบัญชี git, workspace dir, เปิด SQLite, สถานะ gRPC server, จำนวน repo active/closed; รายงานเป็น `OK`/`WARN`/`FAIL` |
| `install-git` | สร้างโฟลเดอร์ `~/.blackvault/tools/git` แล้วบอกวิธีดาวน์โหลด/ตั้ง `git_path` |
| `version` | เวอร์ชันของ cli และ lib |
| `completion bash\|zsh\|...` | สคริปต์ shell completion — เติม subcommand, flag, `[group/repo]`, id บัญชี, ชื่อโปรไฟล์ และคีย์ config โดย**อ่านจาก SQLite/config ในเครื่องเท่านั้น ไม่เรียก remote** |

### 13.7 รูปแบบผลลัพธ์ (`--output`)

global flag `--output json|table|text` (ดีฟอลต์ `text` ซึ่งเหมาะกับ pipe/script) — **รองรับเฉพาะคำสั่งเหล่านี้**:

| คำสั่ง | ค่าที่รองรับ |
| --- | --- |
| `status` | `json` / `table` / `text` |
| `account list` | `json` / `table` / `text` |
| `workspace profile list` | `json` / `table` / `text` |
| `workspace repo list` และ `workspace repo sync` | `json` / `table` / `text` |
| `doctor` | **`json` / `text` เท่านั้น** — `--output table` คืน error ชัดเจน |

- คำสั่งอื่น (เช่น `git list`, `git repo-list`) ยังใช้ `--json` เฉพาะตัว
- `--json` เดิมยังใช้ได้และ**ชนะ `--output` เสมอ** (backward compat แต่ deprecated)
- `output.table_border` ใน config เปิด/ปิดเส้นขอบของโหมด `table` (ดีฟอลต์ปิด)

### 13.8 โหมด develop — NAS

ต้องตั้ง `BLACKVAULT_DEV=1` และ `type: nas` ถึงจะใช้ได้:

```bash
BLACKVAULT_DEV=1 blackvault nas sync --dry-run
```

| Flag | ทำอะไร |
| --- | --- |
| `--dry-run` | แสดงรายการไฟล์ที่จะ sync (เคารพ `.nasignore`) โดยไม่ transfer |
| `--watch` | เฝ้าดู workspace แล้ว sync อัตโนมัติเมื่อไฟล์เปลี่ยน |
| `--account` / `--profile` | เลือกบัญชี NAS ปลายทาง / โปรไฟล์ต้นทาง |

- ต้นทางคือโฟลเดอร์ของ**โปรไฟล์ที่ active** (`workspace.dir/<profile>/`) ไม่ใช่ `workspace.dir` ทั้งก้อน
- ✅ ทำได้แล้ว: `.nasignore` matcher, snapshot/scan, plan engine (upload/skip/conflict/delete-remote/keep-remote), polling watcher, รายงาน dry-run
- ❌ **ยังทำไม่ได้:** transfer จริง — สั่ง sync แบบไม่ใช่ dry-run จะคืน error ว่ายังไม่ implement

### 13.9 ข้อจำกัดที่รู้ (ยังไม่ทำ)

| ข้อจำกัด | ผลกระทบ |
| --- | --- |
| **dirty state ยังไม่ถูกคำนวณ** — `Status()` คืน `Dirty=false` เสมอ | ช่อง `(dirty)` / คอลัมน์ `DIRTY` เรนเดอร์ไว้พร้อมแล้วแต่ยังไม่มีทางแสดงค่าจริง และ `close` ไม่มีทางเตือนว่ามีงานค้าง — **ต้อง push เองก่อนปิดเสมอ** |
| **NAS transfer layer ยังไม่มี** | `nas sync` ใช้ได้แค่ dry-run / วางแผน |
| **workspace + registry ยังไม่แยกต่อ provider** (GH-2) | ถ้าเปิด repo จาก GitHub และ GitLab ที่มี id ชนกันในโปรไฟล์เดียวกัน จะยังไม่ถูก isolate — เลี่ยงด้วยการ**แยกโปรไฟล์ตาม provider** |
| **ไม่มี flag `--provider` ต่อคำสั่ง** (GH-3) | เปลี่ยน provider ต้องสลับบัญชี/โปรไฟล์ |
| **เปิด IDE อัตโนมัติยังไม่ทำ** | `BLACKVAULT_OPEN_IDE` ถูกอ่านแต่ยังไม่ทำอะไร — ต้องเปิดโฟลเดอร์เอง |
| **gRPC ยังไม่ครอบคลุมทุกอย่าง** | คำสั่ง git และ NAS ยังไม่มีใน proto — GUI ต้องพึ่ง CLI สำหรับส่วนนั้น |
