<p align="center">
  <strong>Git BlackVault</strong>
</p>
<p align="center">
  <em>Open what you need. Burn the rest.</em>
</p>
<p align="center">
  <sub>by <a href="https://github.com/darknamer">darknamer</a></sub>
</p>

---

## 📖 สรุปสั้น ๆ

**Git BlackVault** คือแอปสำหรับจัดการ Git Repository แบบ **Ephemeral Workspace**

- **เปิดใช้เมื่อจำเป็น** → Clone → ทำงาน → **ปิดแล้วลบทิ้งทั้งหมด**
- โหมดหลัก (`type: git`) ไม่ sync กับ NAS ไม่เก็บไฟล์ค้าง ไม่ผูก lifecycle กับเครื่อง
<<<<<<< HEAD
- รองรับทั้ง **GitLab และ GitHub** (ตั้ง `git_provider`)
- **ผู้ใช้โต้ตอบผ่าน CLI `blackvault`** จาก repo [`black-vault-cli`](../black-vault-cli/README.md) — ส่วน `black-vault-lib` เป็น implementation detail (สนใจเฉพาะเมื่อจะ contribute) → เริ่มที่ [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start)
=======
- รองรับทั้ง **GitLab และ GitHub** (ตั้ง `git_provider` ของบัญชีที่ใช้งาน)
>>>>>>> c75a56e (Update README and PRODUCT documentation to clarify features, improve structure, and enhance navigation with additional sections and links.)
- _(กำลังพัฒนา)_ solution ทดลอง **`type: nas`** สำหรับ sync ไฟล์ขึ้น NAS — เปิดเฉพาะโหมด develop (`BLACKVAULT_DEV=1`)

> 📌 **อ่านก่อนลงมือ:** สิ่งที่ต้องมีเพื่อ build/รันจริง ดู [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) · สรุปว่า CLI **ทำอะไรได้แล้วตอนนี้** (verify กับโค้ดจริง) ดู [13. สิ่งที่ CLI ทำได้ตอนนี้](#13-สิ่งที่-cli-ทำได้ตอนนี้)
>
> ส่วน §1–§7 เป็น **concept/ปรัชญาของผลิตภัณฑ์** ส่วน §11–§13 เป็น **สถานะจริงของโค้ด ณ ปัจจุบัน**

---

## 📑 สารบัญ

- [📖 สรุปสั้น ๆ](#-สรุปสั้น-ๆ)
- [📑 สารบัญ](#-สารบัญ)
- [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start)
  - [ติดตั้ง / รับ binary](#ติดตั้ง--รับ-binary)
  - [ครั้งแรก: `blackvault init`](#ครั้งแรก-blackvault-init)
  - [Workflow ประจำวัน](#workflow-ประจำวัน)
  - [ไฟล์อยู่ที่ไหน](#ไฟล์อยู่ที่ไหน)
  - [หัวข้อ advanced ใน CLI README](#หัวข้อ-advanced-ใน-cli-readme)
- [1. Concept \& Philosophy](#1-concept--philosophy)
- [2. Core Use Case](#2-core-use-case)
- [3. Key Features (MVP)](#3-key-features-mvp)
  - [3.1 Repository Registry](#31-repository-registry)
  - [3.2 Workspace Lifecycle](#32-workspace-lifecycle)
  - [3.3 Workspace Structure](#33-workspace-structure)
  - [3.4 Open Modes](#34-open-modes)
- [4. App Architecture](#4-app-architecture)
  - [4.1 Overall Architecture](#41-overall-architecture)
  - [4.2 Core Modules](#42-core-modules)
- [5. UX Flow (Desktop / CLI)](#5-ux-flow-desktop--cli)
  - [5.1 Open](#51-open)
  - [5.2 Close](#52-close)
  - [5.3 Status](#53-status)
- [6. Optional Advanced Features (Phase 2)](#6-optional-advanced-features-phase-2)
- [7. What This Project Is NOT](#7-what-this-project-is-not)
- [8. Tech Stack](#8-tech-stack)
- [9. Tagline Ideas](#9-tagline-ideas)
- [10. งานที่เหลือจริง (Next Steps)](#10-งานที่เหลือจริง-next-steps)
- [11. Repositories \& Setup](#11-repositories--setup)
  - [การเชื่อมต่อ](#การเชื่อมต่อ)
  - [สิ่งที่ทำไปแล้วในแต่ละ repo](#สิ่งที่ทำไปแล้วในแต่ละ-repo)
  - [สถานะของ Tasks ชุดเดิม](#สถานะของ-tasks-ชุดเดิม)
- [12. Requirements (สิ่งที่ต้องมี)](#12-requirements-สิ่งที่ต้องมี)
- [13. สิ่งที่ CLI ทำได้ตอนนี้](#13-สิ่งที่-cli-ทำได้ตอนนี้)
  - [13.1 ภาพรวมกลุ่มคำสั่ง](#131-ภาพรวมกลุ่มคำสั่ง)
  - [13.2 Workspace lifecycle](#132-workspace-lifecycle)
  - [13.3 หลายบัญชี และโปรไฟล์ workspace](#133-หลายบัญชี-และโปรไฟล์-workspace)
  - [13.4 คำสั่ง Git ใน workspace](#134-คำสั่ง-git-ใน-workspace)
  - [13.5 gRPC (serve)](#135-grpc-serve)
  - [13.6 ติดตั้งและวินิจฉัย](#136-ติดตั้งและวินิจฉัย)
  - [13.7 รูปแบบผลลัพธ์ (--output)](#137-รูปแบบผลลัพธ์---output)
  - [13.8 โหมด develop — NAS](#138-โหมด-develop--nas)
  - [13.9 ข้อจำกัดที่รู้ (ยังไม่ทำ)](#139-ข้อจำกัดที่รู้-ยังไม่ทำ)

---

## 🚀 การใช้งาน CLI (Developer Quick Start)

ผู้ใช้ปลายทางใช้งานผ่าน **binary `blackvault`** ซึ่ง build จาก repo [`black-vault-cli`](../black-vault-cli/README.md) — ส่วน [`black-vault-lib`](../black-vault-lib/README.md) คือ engine ภายใน (สนใจเฉพาะเมื่อจะ contribute โค้ด)
section นี้สรุป flow หลักให้เริ่มใช้ได้ทันที — หัวข้อ advanced ดูลิงก์ท้าย section

### ติดตั้ง / รับ binary

**ทางที่ 1 — ดาวน์โหลด release binary:**

1. ไปที่ **GitHub Releases** ของ `black-vault-cli`
2. ดาวน์โหลดไฟล์ `blackvault-<os>-<arch>` ให้ตรงเครื่อง — มี 6 targets: linux / windows / darwin × amd64 / arm64 (Windows ลงท้าย `.exe`) พร้อม `checksums.txt` ให้ตรวจสอบ
3. rename เป็น `blackvault` แล้ววางใน `PATH`

> 🧪 อยากได้ build ล่าสุดจาก branch `develop` → ใช้ prerelease tag **`dev`** (rolling — สร้างใหม่ทุก push, version `0.0.0-dev.<short-sha>`, GitHub เท่านั้น)

**ทางที่ 2 — build จาก source:**

ต้องมี **Go 1.24.x** และ checkout `black-vault-lib` เป็น **sibling directory** (บังคับโดย `replace ... => ../black-vault-lib` ใน `go.mod` ของ CLI):

```text
<parent>/
  black-vault-cli/    # repo ของ CLI
  black-vault-lib/    # ต้องมี — ตาม replace ใน go.mod
```

```bash
cd black-vault-cli
go mod download
make build              # หรือ: go build -o blackvault .   (Windows: -o blackvault.exe)
./blackvault version    # ตรวจว่า build สำเร็จ
```

### ครั้งแรก: `blackvault init`

ตั้งค่าครั้งแรกด้วย `init` — โหมด interactive (กด Enter รับค่า default ในวงเล็บ) หรือส่ง flag สำหรับ script/CI:

```bash
# ถามทีละข้อ (แนะนำสำหรับครั้งแรก)
blackvault init

# ไม่ถามเลย — เหมาะกับ script/CI (เลือก provider แล้ว URL default ตั้งให้อัตโนมัติ)
blackvault init --yes --git-provider github --git-username <user>
```

สิ่งที่ `init` ถาม/ตั้งค่า (เขียนลง `~/.blackvault/config.yaml`):

| คำถาม                                                                        | Config key                  |
| ---------------------------------------------------------------------------- | --------------------------- |
| Git provider (`gitlab` / `github` — เลือกแล้วตั้ง URL default ให้)                | `git_provider`              |
| Git host URL                                                                  | `git_url`                   |
| Username / Email                                                              | `git_username`, `git_email` |
| Credentials (token)                                                           | `git_credentials`           |
| โหมด git (`system` = git ในเครื่อง / `portable` = `~/.blackvault/tools/git`)     | `git_path`                  |
| Workspace path (default `~/.blackvault/workspaces`)                           | `workspace.dir`             |

ตัวช่วย bootstrap / troubleshoot:

| คำสั่ง                    | ทำอะไร                                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------------------------- |
| `blackvault install`     | validate config, สร้าง directory ที่ขาด, ทดสอบ connectivity (`init` เรียกให้อัตโนมัติหลัง save config สำเร็จ)      |
| `blackvault doctor`      | วินิจฉัยแบบ **read-only** (config, git binary, DB, gRPC, จำนวน repo) — รายงานเป็น OK / WARN / FAIL            |
| `blackvault install-git` | เตรียมโฟลเดอร์ portable git ที่ `~/.blackvault/tools/git`                                                     |

### Workflow ประจำวัน

1. **เปิด workspace** — clone repo ลงโฟลเดอร์มาตรฐาน

   ```bash
   blackvault open group/repo                 # clone ปกติ
   blackvault open group/repo --shallow       # shallow clone
   blackvault open group/repo --branch main   # เจาะ branch
   ```

   เลือกวิธี clone ได้ด้วย `--clone-mode https|https-auth|ssh`

2. **ทำงาน git ผ่าน CLI โดยไม่ต้อง `cd`** — ทุกคำสั่งอ้าง `group/repo` เดิม

   ```bash
   blackvault git add group/repo
   blackvault git commit group/repo -m "ข้อความ"
   blackvault git pull group/repo
   blackvault git push group/repo
   ```

   มีครบทั้ง `branch`, `merge`, `remote`, `fetch`, `flow` (git-flow), `list`, `repo-list`

3. **ดูสถานะ** — repo ไหน active / closed

   ```bash
   blackvault status
   ```

4. **ปิด workspace = ลบโฟลเดอร์ทิ้งทั้งหมด**

   ```bash
   blackvault close group/repo            # ยืนยันก่อนลบ
   blackvault close group/repo --force
   ```

   > ⚠️ ไม่มี sync / backup อัตโนมัติ — **push เองก่อนปิดเสมอ**

คำสั่งที่รองรับใช้ global flag `--output json|table|text` ได้ (default `text`)

### ไฟล์อยู่ที่ไหน

ทุกอย่างอยู่ใต้ `~/.blackvault/`:

| Path                                     | ความหมาย                                                        |
| ---------------------------------------- | --------------------------------------------------------------- |
| `config.yaml`                            | config ผู้ใช้ (ดู/แก้ผ่าน `blackvault config get` / `config set`)   |
| `blackvault.db`                          | SQLite registry + cache (สร้างอัตโนมัติเมื่อใช้ครั้งแรก)                |
| `workspaces/<profile>/<group>/<repo>/`   | workspace ที่ clone (default dir คือ `~/.blackvault/workspaces`)   |
| `tools/git`                              | portable git (เตรียมด้วย `install-git`)                           |
| `grpc_port`, `grpc_token`, `grpc_cert.pem` | ไฟล์ session ของ `blackvault serve` (ให้ GUI ใช้ connect)         |

### หัวข้อ advanced ใน CLI README

รายละเอียดเต็ม (command reference ~2000 บรรทัด) อยู่ใน [README ของ black-vault-cli](../black-vault-cli/README.md):

- ตัวอย่างคำสั่งทั้งหมด → [ตัวอย่างคำสั่ง](../black-vault-cli/README.md#usage-examples)
- รายละเอียด `init` + flags ทั้งหมด → [เริ่มต้นใช้งานด้วย init](../black-vault-cli/README.md#เริ่มต้นใช้งานด้วย-init)
- รายละเอียดการ build / cross-compile → [การ build](../black-vault-cli/README.md#build)
- หลาย account (git/NAS) → [account](../black-vault-cli/README.md#account-multi)
- workspace profiles → [workspace profile](../black-vault-cli/README.md#workspace-profiles)
- bulk open/close (`--all`) → [bulk open/close](../black-vault-cli/README.md#bulk-open-close)
- gRPC server สำหรับ GUI → [GUI กับ serve](../black-vault-cli/README.md#gui-กับ-serve)
- internals ของ lib (สำหรับ contributor) → [README ของ black-vault-lib](../black-vault-lib/README.md)

---

## 1. Concept & Philosophy

**Git BlackVault** ไม่ใช่ Git client และไม่ใช่ sync tool  
แต่เป็น **Workspace Manager** สำหรับ repo ที่:

| ไม่ทำ                      | ทำ                                  |
| ------------------------ | ---------------------------------- |
| ❌ ไม่ sync กับ NAS         | ✅ เปิด–ปิดชัดเจน                      |
| ❌ ไม่เก็บไฟล์ค้าง            | ✅ ควบคุมได้ว่า repo ไหน “มีตัวตนอยู่ตอนนี้” |
| ❌ ไม่ผูก lifecycle กับเครื่อง |                                    |

แนวคิดคล้าย **black box / vault**:

> **เปิดกล่อง** → มี repo  
> **ปิดกล่อง** → ไม่มีอะไรเหลือ

---

## 2. Core Use Case

- ใช้หลาย repo จาก **GitLab หรือ GitHub**
- ไม่อยาก clone ค้างไว้ 20–30 repo
- อยากแยก context งานชัดเจน
- เครื่องโล่ง / backup ง่าย / ไม่เสี่ยงข้อมูลรั่ว

---

## 3. Key Features (MVP)

### 3.1 Repository Registry

- เชื่อม GitLab หรือ GitHub ด้วย Token
- ดึง repo list
- tag / group / project

**โครงข้อมูลจริง** (ตาราง `repos` ใน SQLite — `~/.blackvault/blackvault.db`) และ metadata ต่อโปรไฟล์ใน `config.yaml` — ดู [docs/DATATABLE.md](docs/DATATABLE.md)

---

### 3.2 Workspace Lifecycle

| Action              | Result                 |
| ------------------- | ---------------------- |
| **Open Workspace**  | clone repo → เตรียม env (ตั้ง branch ตาม default branch policy) |
| **Active**          | ใช้งานตามปกติ            |
| **Close Workspace** | delete directory 100% (ดีฟอลต์) |
| **Close (--no-purge)** | ปิดในทะเบียนแต่คงไฟล์ไว้บน disk |

> ⚠️ **สำคัญ:** ไม่มี sync, ไม่มี backup อัตโนมัติ และ **ไม่มีการตรวจ dirty ก่อนปิด** — user ต้อง **push เองก่อนปิด**
> ไม่มีสถานะ "Force Close" แยกต่างหาก: `close` ลบตรง ๆ อยู่แล้ว จึงไม่มี flag `--force`

---

### 3.3 Workspace Structure

```
~/.blackvault/
├── config.yaml          # บัญชี, โปรไฟล์, cache, output, git_path
├── blackvault.db        # SQLite registry + cache
├── cache/
├── tools/git/           # portable git (ถ้าใช้)
└── workspaces/
    └── <profile>/       # ชื่อโปรไฟล์ workspace เช่น git-default, git-github.com
        └── <group>/     # รองรับ subgroup ซ้อนหลายชั้น
            └── <repo>/
```

> ⚠️ path **ไม่มี** segment ชื่อ provider (เช่น `gitlab/`) — ตัวคั่นชั้นบนสุดคือ **ชื่อโปรไฟล์ workspace** ที่ผูกกับบัญชีหนึ่งบัญชี
> ตอนรัน `blackvault serve` จะมีไฟล์เซสชันเพิ่มชั่วคราว: `grpc_port`, `grpc_token`, `grpc_cert.pem` (ลบทิ้งตอนปิด) — ดู [docs/REQUIREMENTS.md §12.4](docs/REQUIREMENTS.md#124-ไฟล์รันไทม์ใต้-blackvault)

---

### 3.4 Open Modes

- Open (normal clone)
- Open (shallow clone)
- Open (branch-specific)
- Open (detached / review mode)

---

## 4. App Architecture

### 4.1 Overall Architecture

```
[ Desktop App (Flutter) ]
         │  gRPC — 127.0.0.1 เท่านั้น + TLS + session token
         ▼
[ CLI: blackvault serve ]───[ CLI: คำสั่งปกติ open/close/git/... ]
         │                              │
         └──────────────┬───────────────┘
                        ▼
         [ BlackVault Core Service (lib) ]
                        │
                        ├── Git Provider (GitLab / GitHub — API จริงทั้งคู่)
                        ├── Workspace Manager (clone/ลบ/list active)
                        ├── SQLite Store (registry + cache)
                        ├── NAS Solution (develop) — วางแผน sync ไฟล์ขึ้น NAS
                        └── File System Guard (กัน path traversal)
```

---

### 4.2 Core Modules

| Module                | หน้าที่                                                                      |
| --------------------- | ------------------------------------------------------------------------- |
| **Git Provider**      | interface กลาง `Provider` — GitLab (`/api/v4`) และ GitHub (`api.github.com` / GHES `/api/v3`) เป็น **client จริงทั้งคู่** (Ping + ListProjects); clone URL แบบ provider-agnostic 3 โหมด: `https` / `https-auth` (ฝัง `user:token@`) / `ssh` |
| **Workspace Manager** | clone (system/portable git หรือ go-git fallback), delete + prune โฟลเดอร์แม่ที่ว่าง, rename, list active (recursive หา `.git`), แยก root ต่อ **โปรไฟล์** |
| **SQLite Store**      | registry `repos` (scope ต่อบัญชี) + `cache` ที่ `~/.blackvault/blackvault.db` — pure-Go ไม่ต้องใช้ CGO; โครงคอลัมน์ดู [docs/DATATABLE.md](docs/DATATABLE.md) |
| **NAS Solution** *(develop)* | วางแผน sync ไฟล์ workspace ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, นโยบายแก้ conflict/ลบ, โหมด manual/interval/watch — **transfer layer จริงยังไม่ทำ** |
| **Safety Guard**      | กัน path traversal **สองชั้น** — `NormalizeRepoPath` ปฏิเสธ absolute path / `..` / Windows volume ตั้งแต่ต้นทาง แล้ว `SafePathFor` ยืนยันอีกครั้งว่ายังอยู่ใต้ workspace root ก่อนแตะ filesystem ทุกครั้ง; scrub credential ออกจาก error/output. **หมายเหตุ:** `close` **ไม่มี** `--force` และยังไม่ตรวจ dirty state — ดู [13.9](#139-ข้อจำกัดที่รู้-ยังไม่ทำ) |

---

## 5. UX Flow (Desktop / CLI)

> ครั้งแรกตั้งค่าด้วย `blackvault init` (เลือก provider gitlab/github, workspace, ฯลฯ) — flow หลักคือ open / close / status ด้านล่าง ส่วนคำสั่ง Git (commit/branch/merge/git-flow), `config`, `nas` ฯลฯ ดูรายการเต็มใน [README ของ black-vault-cli](../black-vault-cli/README.md)
>
> ขั้นตอนติดตั้ง–ใช้งานแบบลงมือได้จริง ดู [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start) ด้านบน

### 5.1 Open

```bash
blackvault open group/repo
```

- clone ลง `workspaces/<profile>/<group>/<repo>/`
- ใช้ **default branch policy** อัตโนมัติ: เตรียม `main` (ไม่มีก็ลอง `master`) แล้ว**ถ้ามี `origin/develop` จะจบที่ `develop`** — เป็น best-effort, ระบุ `--branch`/`--detached` เอง = ข้าม policy
- พิมพ์ path ที่ clone ลง พร้อมบรรทัด `On branch: <b>`

> ยังไม่มีการเปิด IDE ให้อัตโนมัติ — ดู [13.9](#139-ข้อจำกัดที่รู้-ยังไม่ทำ)

---

### 5.2 Close

```bash
blackvault close group/repo
```

- **ลบไฟล์และไดเรกทอรีทิ้งทันที** (ดีฟอลต์ `--purge`) — ไม่ถามยืนยัน และ**ไม่ตรวจ dirty state**
- เก็บกวาดไดเรกทอรีแม่ที่ว่างเปล่าขึ้นไปจนถึง root ของโปรไฟล์
- ปิด path ที่ไม่มีอยู่แล้ว = สำเร็จ (idempotent)
- `--no-purge` = เก็บไฟล์ไว้บน disk ปิดเฉพาะในทะเบียน

> ⚠️ **ไม่มี flag `--force`** — และเพราะไม่มีการตรวจ dirty ผู้ใช้ต้อง **push เองก่อนปิด** เสมอ

---

### 5.3 Status

```bash
blackvault status
```

**ตัวอย่างผลลัพธ์** (ส่วนหัวบัญชี/โปรไฟล์ + รายการ repo):

```
BlackVault status
  account:      https://gitlab.com (git, provider=gitlab)
  url:          https://gitlab.com
  connectivity: [OK  ] https://gitlab.com
  profile:      git-default
  workspace:    /Users/me/.blackvault/workspaces/git-default

ACTIVE:
  - group/api-service
  - group/frontend-web
CLOSED:
  - group/infra
```

- ส่วนที่ว่างจะพิมพ์ `(none)`
- **ACTIVE** อ่านจาก disk (โฟลเดอร์ที่มี `.git`), **CLOSED** = อยู่ใน registry แต่ไม่อยู่บน disk
- รองรับ `--output json|table|text` (ดีฟอลต์ `text`) และ `--account` / `--profile` สลับชั่วคราวต่อคำสั่ง

---

## 6. Optional Advanced Features (Phase 2)

> **Workspace Profiles ทำเสร็จแล้ว** และย้ายออกจากตารางนี้ — ดู [13.3](#133-หลายบัญชี-และโปรไฟล์-workspace)

| Feature                   | รายละเอียด                                              |
| ------------------------- | ------------------------------------------------------ |
| **NAS Sync Solution** *(เริ่มแล้ว — develop)* | `type: nas` — sync ไฟล์ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, manual/interval/watch, นโยบายแก้ conflict/ลบ; **วางแผน + dry-run ได้แล้ว เหลือ transfer layer จริง** |
| **Dirty-state detection** | ตรวจว่ามีการแก้ที่ยังไม่ commit/push ก่อนปิด — ช่อง `dirty` มีอยู่ใน output แล้วแต่ยังคืน `false` เสมอ |
| **Time-based Auto Close** | ปิดอัตโนมัติหลัง idle X ชั่วโมง                               |
| **Encrypted Temp Vault**  | encrypt workspace folder (LUKS / fscrypt)              |
| **IDE Integration**       | VS Code extension, เปิดผ่าน command โดยตรง               |

---

## 7. What This Project Is NOT

- ❌ Git GUI
- ❌ GitHub Desktop clone
- ❌ Backup tool
- ❌ NAS sync *(โดยค่าเริ่มต้น `type: git` — มี solution ทดลอง `type: nas` แยกต่างหากในโหมด develop)*

> **Git BlackVault = lifecycle manager ของ repo**

---

## 8. Tech Stack

| Layer        | ที่ใช้จริง                                          |
| ------------ | ------------------------------------------------ |
| Core / Lib   | Go (`black-vault-lib`)                            |
| CLI          | Go + Cobra (`spf13/cobra`)                        |
| Git          | system/portable git, fallback **go-git** (in-process) |
| Store        | SQLite (`modernc.org/sqlite`, pure-Go, CGO ปิด)   |
| GUI          | Flutter (desktop — Windows / Linux / macOS)      |
| CLI ↔ GUI    | gRPC บน loopback + **TLS** (self-signed ต่อเซสชัน) + **session token** ทุก RPC; proto อยู่ใน lib และ `.pb.go` commit ไว้แล้ว (ไม่ต้องมี `protoc` ตอน build) |
| API backend  | C# / ASP.NET Core (`black-vault-api`) — License   |
| Config       | YAML (`~/.blackvault/config.yaml`)               |
| Auth         | GitLab / GitHub Token                            |
| OS           | Linux / macOS / Windows                          |

---

## 9. Tagline Ideas

- *Open what you need. Burn the rest.*
- *Repos are temporary. Control is permanent.*
- *No sync. No clutter. Just work.*

---

## 10. งานที่เหลือจริง (Next Steps)

CLI spec, `config.yaml` และ MVP ถูก implement ไปแล้ว (ดู [11](#11-repositories--setup) และ [13](#13-สิ่งที่-cli-ทำได้ตอนนี้)) — ที่เหลือคือ:

| # | งาน | อยู่ที่ | สถานะ |
| --- | --- | --- | --- |
| GAP-1 | คำนวณ **dirty state** จริง (`Status()` ยังคืน `Dirty=false` เสมอ) แล้วเปิดทาง `close` ให้เตือน/ถามยืนยันได้ | lib + cli | ยังไม่เริ่ม |
| NAS-1 | **transfer layer** ของ solution NAS (ตอนนี้ dry-run/วางแผนได้ แต่ sync จริงคืน error `not implemented yet`) + ตัวอ่าน state ฝั่ง remote เพื่อต่อกับ plan engine | lib | ค้างอยู่ |
| GH-2 | แยก **workspace/registry ต่อ provider** — path ยังเป็น `<profile>/<group>/<repo>` และตาราง `repos` ยังไม่มีคอลัมน์ provider จึงยังไม่ isolate กรณี id ชนกันระหว่าง GitHub กับ GitLab ในโปรไฟล์เดียว | lib | ยังไม่เริ่ม |
| GH-3 | flag `--provider` ต่อคำสั่ง | cli | ยังไม่เริ่ม |
| IDE-1 | เปิด IDE อัตโนมัติหลัง `open` (`BLACKVAULT_OPEN_IDE` ถูกอ่านแล้วแต่ยังเป็น no-op) | cli | ยังไม่เริ่ม |

รายละเอียดเชิงข้อกำหนดแบบเต็ม (testable "shall" + citation จากโค้ด + known gaps ของทั้ง 4 repo) อยู่ที่ [black-vault-cli/docs/REQUIREMENTS.md](../black-vault-cli/docs/REQUIREMENTS.md)

---

## 11. Repositories & Setup

โปรเจกต์แบ่งเป็น 5 repo (หรือโฟลเดอร์ใน monorepo) ดังนี้:

| Repo                | ภาษา           | หน้าที่                                                                                                                          |
| ------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **black-vault**     | —              | เอกสารและ concept (repo นี้)                                                                                                    |
| **black-vault-lib** | Go             | Library — logic ทั้งหมด: config, workspace lifecycle, clone (provider-agnostic), **SQLite store** (registry + cache), **GitLab + GitHub client จริง**, solution NAS; เก็บ API contract (`.proto`) สำหรับ gRPC |
| **black-vault-cli** | Go             | CLI (Cobra) — ตัวห่อบาง ๆ ของ lib: `init`/`config`/`account`/`workspace`, `open`/`close`/`status`, `install`/`doctor`, คำสั่ง Git (`git ...`), `nas` (develop), `serve` (gRPC ให้ GUI) |
| **black-vault-gui** | Flutter        | Desktop GUI (Windows/Linux/macOS); เป็น gRPC client ไปที่ `blackvault serve` บน `127.0.0.1` (พอร์ต dynamic อ่านจาก `~/.blackvault/grpc_port`) |
| **black-vault-api** | C# / ASP.NET   | Backend แยกต่างหากสำหรับระบบ **License** (Customer / License / LicenseActivation) + environment config (Local/Dev/Staging/Prod) |

### การเชื่อมต่อ

- **CLI ↔ GUI:** ผ่าน gRPC — CLI รัน `blackvault serve` (ผูก `127.0.0.1` เท่านั้น + TLS + session token) แล้ว GUI อ่าน `~/.blackvault/grpc_port` / `grpc_token` / `grpc_cert.pem` เพื่อต่อ; พอร์ตเริ่มที่ `50051` แล้วไล่หาตัวว่างถัดไปถ้าไม่ว่าง — **อย่า hardcode 50051**
- **Proto:** อยู่ที่ `black-vault-lib/api/proto/blackvault.proto` (และ copy ไว้ใน CLI / GUI สำหรับ generate Go และ Dart) — 10 RPC รวม `ObserveOperations` (server-streaming) สำหรับ console log ฝั่ง GUI

### สิ่งที่ทำไปแล้วในแต่ละ repo

- **black-vault-lib:** Config (YAML — `type`, บัญชีรวม `accounts[]` + `active_account`, บล็อก nested `workspace:`/`cache:`/`output:`, NAS keys) พร้อม migration จาก config รูปแบบเก่า, Workspace (open/close/list/rename + กัน path traversal สองชั้น), provider-agnostic clone 3 โหมด (system/portable git + go-git fallback), **SQLite store** (repos scope ต่อบัญชี + cache ที่ `~/.blackvault/blackvault.db` สร้างใหม่ถ้าไม่มี), **GitLab + GitHub client จริง** (Ping + ListProjects; GitHub เดินทุก org ที่เข้าถึงได้ และรองรับ GitHub Enterprise ผ่าน `/api/v3`), **solution NAS** (`internal/nas`: `.nasignore` matcher, plan engine แก้ conflict/ลบ, polling watcher — มี unit test), Service (Open/Close/Status/StatusSummary/ListRepositories/WorkspaceRepo*/Install/Doctor + คำสั่ง `Git*` 22 เมธอด + accessor ของ config/NAS), observer hub สำหรับสตรีม log, proto นิยาม BlackVaultService
- **black-vault-cli:** Cobra (ตัวห่อบาง ๆ ของ lib) — `init`/`config`/`account`/`workspace`, `open`/`close`/`status`, `install`/`doctor`, subcommand tree `git` (list/repo-list/add/commit/fetch/pull/push/merge/branch/remote/flow/help), `install-git`, `version`, shell completion, `nas` (develop), `serve` (register `BlackVaultService` ครบ + TLS + token); global flag `--output json|table|text`; **command tests** ใน `tests/cmd/`; `.pb.go` commit ไว้ใน repo (`make proto` ใช้เฉพาะตอนแก้ `.proto`)
- **black-vault-gui:** โปรเจกต์ Flutter พร้อมโฟลเดอร์ `windows/`, `linux/`, `macos/`, generated Dart client (`lib/src/generated/`) และ **gRPC client จริง** (`black_vault_service.dart` — ครบทุก RPC รวม `observeOperations` แบบ streaming, แนบ token + TLS), ธีม/ภาษา/เลย์เอาต์ responsive
- **black-vault-api:** Backend C# / ASP.NET Core — ระบบ License (Customer / License / LicenseActivation), environment config 4 แบบ (Local/Development/Staging/Production) พร้อม `appsettings.{Environment}.json`

รายละเอียดและวิธี build/รัน ดูใน **README.md ของแต่ละ repo**

### สถานะของ Tasks ชุดเดิม

✅ **Tasks ชุดเดิมทั้ง 6 ข้อทำเสร็จหมดแล้ว** (สร้าง platform ของ Flutter, generate stub ทั้ง Go และ Dart, register `BlackVaultService` ใน `serve`, ต่อ UI กับ gRPC client จริง)

| #   | Repo            | Task เดิม                                          | หลักฐานว่าเสร็จ                                                        |
| --- | --------------- | -------------------------------------------------- | --------------------------------------------------------------------- |
| 1–2 | black-vault-gui | สร้าง platform desktop + รันแอป                     | มีโฟลเดอร์ `windows/`, `linux/`, `macos/` ใน repo                        |
| 3   | black-vault-cli | `make proto` ให้ได้ `.pb.go`                         | `api/proto/blackvault.pb.go` + `blackvault_grpc.pb.go` commit ไว้แล้ว   |
| 4   | black-vault-cli | register `BlackVaultService` ใน `cmd/serve.go`       | register ครบ 10 RPC + Health probe พร้อม TLS/token interceptor          |
| 5   | black-vault-gui | generate Dart client                                | `lib/src/generated/blackvault.pbgrpc.dart` ฯลฯ                          |
| 6   | black-vault-gui | ต่อ UI กับ gRPC client จริง                          | `lib/src/black_vault_service.dart` เรียกจริงครบทุก RPC + streaming      |

งานที่เหลือหลังจากนี้ดู [10. งานที่เหลือจริง](#10-งานที่เหลือจริง-next-steps) และข้อจำกัดปัจจุบันดู [13.9](#139-ข้อจำกัดที่รู้-ยังไม่ทำ)

---

## 12. Requirements (สิ่งที่ต้องมี)

ข้อกำหนดสำหรับ **build และรันของจริง** ณ สถานะปัจจุบันของ repo (ไม่ใช่แผน) ย้ายไปอยู่ที่ **[docs/REQUIREMENTS.md](docs/REQUIREMENTS.md)** — ครอบคลุมเครื่องมือที่ต้องมี ([12.1](docs/REQUIREMENTS.md#121-เครื่องมือที่ต้องมี)), โครงสร้างโฟลเดอร์ที่บังคับ ([12.2](docs/REQUIREMENTS.md#122-โครงสร้างโฟลเดอร์ที่บังคับ)), บัญชีและสิทธิ์ ([12.3](docs/REQUIREMENTS.md#123-บัญชีและสิทธิ์)), ไฟล์รันไทม์ใต้ `~/.blackvault/` ([12.4](docs/REQUIREMENTS.md#124-ไฟล์รันไทม์ใต้-blackvault)), โหมด develop ([12.5](docs/REQUIREMENTS.md#125-โหมด-develop)) และวิธีตรวจว่าพร้อมใช้งาน ([12.6](docs/REQUIREMENTS.md#126-ตรวจว่าพร้อมใช้งาน))

> spec เชิงข้อกำหนดแบบเต็ม (testable "shall") อยู่คนละที่ — ดู [black-vault-cli/docs/REQUIREMENTS.md](../black-vault-cli/docs/REQUIREMENTS.md)

---

## 13. สิ่งที่ CLI ทำได้ตอนนี้

สรุปจาก **command tree จริง** ของ `black-vault-cli` (`cmd/root.go` + ไฟล์ของแต่ละคำสั่ง) และเมธอดที่มีจริงใน `black-vault-lib` — ไม่ใช่โรดแมป
รายละเอียดของแต่ละ flag ดู [README ของ black-vault-cli](../black-vault-cli/README.md)

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

**`status`** — ส่วนหัว (บัญชี / connectivity / โปรไฟล์ / workspace root) + รายการ `ACTIVE` และ `CLOSED` ดูตัวอย่างที่ [5.3](#53-status)

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

> คำสั่งกลุ่มนี้**ต้องมี git binary จริง** — go-git fallback ใช้ได้เฉพาะตอน clone เท่านั้น (ดู [docs/REQUIREMENTS.md §12.1](docs/REQUIREMENTS.md#121-เครื่องมือที่ต้องมี))
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
