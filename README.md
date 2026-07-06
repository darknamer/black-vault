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
- รองรับทั้ง **GitLab และ GitHub** (ตั้ง `git_provider`)
- **ผู้ใช้โต้ตอบผ่าน CLI `blackvault`** จาก repo [`black-vault-cli`](../black-vault-cli/README.md) — ส่วน `black-vault-lib` เป็น implementation detail (สนใจเฉพาะเมื่อจะ contribute) → เริ่มที่ [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start)
- _(กำลังพัฒนา)_ solution ทดลอง **`type: nas`** สำหรับ sync ไฟล์ขึ้น NAS — เปิดเฉพาะโหมด develop (`BLACKVAULT_DEV=1`)

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
- [10. Next Steps](#10-next-steps)
- [11. Repositories \& Setup](#11-repositories--setup)
  - [การเชื่อมต่อ](#การเชื่อมต่อ)
  - [สิ่งที่ทำไปแล้วในแต่ละ repo](#สิ่งที่ทำไปแล้วในแต่ละ-repo)
  - [Tasks — สิ่งที่ต้องทำต่อ (เมื่อมีเครื่องพร้อม)](#tasks--สิ่งที่ต้องทำต่อ-เมื่อมีเครื่องพร้อม)

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

- เชื่อม GitLab (Token)
- ดึง repo list
- tag / group / project

**โครงข้อมูลตัวอย่าง:**

```
repo_id
name
group
default_branch
last_opened
status (closed | active)
```

---

### 3.2 Workspace Lifecycle

| Action              | Result                 |
| ------------------- | ---------------------- |
| **Open Workspace**  | clone repo → เตรียม env |
| **Active**          | ใช้งานตามปกติ            |
| **Close Workspace** | delete directory 100%  |
| **Force Close**     | kill process + delete  |

> ⚠️ **สำคัญ:** ไม่มี sync, ไม่มี backup อัตโนมัติ — user ต้อง **push เองก่อนปิด**

---

### 3.3 Workspace Structure

```
~/.blackvault/
├── config.yaml
├── cache/
└── workspaces/
    └── gitlab/
        └── group-name/
            └── repo-name/
```

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
[ Desktop App / CLI ]
         │
         ▼
[ BlackVault Core Service ]
         │
         ├── Git Provider (GitLab / GitHub)
         ├── Workspace Manager
         ├── NAS Solution (develop) — sync ไฟล์ขึ้น NAS
         └── File System Guard
```

---

### 4.2 Core Modules

| Module                | หน้าที่                                                                      |
| --------------------- | ------------------------------------------------------------------------- |
| **Git Provider**      | GitLab / GitHub, Token auth, provider-agnostic clone URL (`group/repo`)    |
| **Workspace Manager** | clone (system/portable git หรือ go-git fallback), delete, list active, SQLite registry |
| **NAS Solution** *(develop)* | sync ไฟล์ workspace ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, นโยบายแก้ conflict/ลบ, โหมด manual/interval/watch |
| **Safety Guard**      | optional `--force` ตอนปิด (detect dirty state เป็น TODO)                     |

---

## 5. UX Flow (Desktop / CLI)

> ครั้งแรกตั้งค่าด้วย `blackvault init` (เลือก provider gitlab/github, workspace, ฯลฯ) — flow หลักคือ open / close / status ด้านล่าง ส่วนคำสั่ง Git (commit/branch/merge/git-flow), `config`, `nas` ฯลฯ ดูรายการเต็มใน [README ของ black-vault-cli](../black-vault-cli/README.md)
>
> ขั้นตอนติดตั้ง–ใช้งานแบบลงมือได้จริง ดู [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start) ด้านบน

### 5.1 Open

```bash
blackvault open group/repo
```

- clone
- เปิดโฟลเดอร์ (VS Code / Cursor / IDE)

---

### 5.2 Close

```bash
blackvault close group/repo
```

- ตรวจสอบ dirty state
- ยืนยัน
- ลบ directory

---

### 5.3 Status

```bash
blackvault status
```

**ตัวอย่างผลลัพธ์:**

```
ACTIVE:
- group/api-service
- group/frontend-web

CLOSED:
- group/infra
```

---

## 6. Optional Advanced Features (Phase 2)

| Feature                   | รายละเอียด                                              |
| ------------------------- | ------------------------------------------------------ |
| **NAS Sync Solution** *(เริ่มแล้ว — develop)* | `type: nas` — sync ไฟล์ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, manual/interval/watch, นโยบายแก้ conflict/ลบ; เหลือ transfer layer จริง |
| **Workspace Profiles**    | dev / review / hotfix — different clone depth / branch |
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
| GUI          | Flutter (desktop)                                |
| CLI ↔ GUI    | gRPC (proto ใน lib)                              |
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

## 10. Next Steps

ถ้าคุณอยากต่อ:

- ผมแตก **CLI spec**
- ออกแบบ **config.yaml**
- หรือทำ **MVP roadmap 2–4 สัปดาห์**

บอกได้เลยว่าจะไปทาง **CLI-first**, **Desktop-first**, หรือ **Hybrid**

---

## 11. Repositories & Setup

โปรเจกต์แบ่งเป็น 5 repo (หรือโฟลเดอร์ใน monorepo) ดังนี้:

| Repo                | ภาษา           | หน้าที่                                                                                                                          |
| ------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **black-vault**     | —              | เอกสารและ concept (repo นี้)                                                                                                    |
| **black-vault-lib** | Go             | Library — logic ทั้งหมด: config, workspace lifecycle, clone (provider-agnostic), **SQLite store** (registry + cache), GitLab client, solution NAS; เก็บ API contract (`.proto`) สำหรับ gRPC |
| **black-vault-cli** | Go             | CLI (Cobra) — ตัวห่อบาง ๆ ของ lib: `init`/`config`, `open`/`close`/`status`, คำสั่ง Git (commit/branch/merge/remote/git-flow), `nas` (develop), `serve` (gRPC ให้ GUI)                |
| **black-vault-gui** | Flutter        | Desktop GUI; เป็น gRPC client ไปที่ `blackvault serve` (localhost:50051)                                                        |
| **black-vault-api** | C# / ASP.NET   | Backend แยกต่างหากสำหรับระบบ **License** (Customer / License / LicenseActivation) + environment config (Local/Dev/Staging/Prod) |

### การเชื่อมต่อ

- **CLI ↔ GUI:** ผ่าน gRPC — CLI รัน `blackvault serve` แล้ว GUI connect ไปที่ `localhost:50051`
- **Proto:** อยู่ที่ `black-vault-lib/api/proto/blackvault.proto` (และ copy ไว้ใน CLI / GUI สำหรับ generate Go และ Dart)

### สิ่งที่ทำไปแล้วในแต่ละ repo

- **black-vault-lib:** Config (YAML — `type`, `git_provider` gitlab/github, NAS keys), Workspace (open/close/list), provider-agnostic clone (system/portable git + go-git fallback), **SQLite store** (repos + cache ที่ `~/.blackvault/blackvault.db` สร้างใหม่ถ้าไม่มี), GitLab client (stub), **solution NAS** (`internal/nas`: `.nasignore` matcher, plan engine แก้ conflict/ลบ, polling watcher — มี unit test), Service (Open/Close/Status/ListRepositories + accessor ของ config/NAS), proto นิยาม BlackVaultService
- **black-vault-cli:** Cobra (ตัวห่อบาง ๆ ของ lib) — `init`/`config`, `open`/`close`/`status`, คำสั่ง Git (add/commit/fetch/pull/push, branch, merge, remote, git-flow), `install-git`, `version`, `nas` (develop), `serve`; **command tests** ใน `tests/cmd/`; proto สำหรับ generate Go (`make proto`). _หมายเหตุ:_ build เต็มต้อง checkout `black-vault-lib` เป็น sibling ที่มี `Git*` methods ครบ (เฟส 0 ใน README ของ CLI)
- **black-vault-gui:** โปรเจกต์ Flutter, หน้าเชื่อม gRPC (placeholder), proto สำหรับ generate Dart (`make proto`)
- **black-vault-api:** Backend C# / ASP.NET Core — ระบบ License (Customer / License / LicenseActivation), environment config 4 แบบ (Local/Development/Staging/Production) พร้อม `appsettings.{Environment}.json`

รายละเอียดและวิธี build/รัน ดูใน **README.md ของแต่ละ repo**

### Tasks — สิ่งที่ต้องทำต่อ (เมื่อมีเครื่องพร้อม)

ทำบนเครื่องที่มี CLI ที่ต้องการ (Go, Flutter, protoc ตามรายการ):

| #   | Repo                | Task                                                                               | หมายเหตุ                                                                   |
| --- | ------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| 1   | **black-vault-gui** | ติดตั้ง Flutter CLI แล้วรัน `flutter create . --platforms=windows,linux,macos`          | สร้างโฟลเดอร์ `windows/`, `linux/`, `macos/` (ตอนนี้ยังไม่มีในเครื่องที่ไม่มี Flutter) |
| 2   | black-vault-gui     | รัน `flutter pub get` แล้วทดสอบ `flutter run -d macos` (หรือ windows/linux)           | ให้แอปขึ้นและกด Connect ได้                                                   |
| 3   | black-vault-cli     | ติดตั้ง protoc + protoc-gen-go, protoc-gen-go-grpc แล้วรัน `make proto`                 | ได้ไฟล์ `.pb.go`, `_grpc.pb.go`                                             |
| 4   | black-vault-cli     | แก้ `cmd/serve.go` ให้ register BlackVaultService และใช้ generated code แล้ว build ใหม่ | ให้ `blackvault serve` รัน gRPC จริง                                         |
| 5   | black-vault-gui     | รัน `make proto` (ต้องมี protoc + protoc_plugin)                                      | ได้ Dart client ใน `lib/src/generated/`                                    |
| 6   | black-vault-gui     | ต่อ UI กับ gRPC client จริง (แทน placeholder)                                         | Connect แล้วเรียก GetStatus / Open / Close ได้                               |

ถ้าเครื่องไหนยังไม่มี Flutter — ไปทำ task 1–2, 5–6 บนเครื่องที่มี Flutter อีกรอบเดียวก็ได้
