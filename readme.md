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
- ไม่ sync กับ NAS ไม่เก็บไฟล์ค้าง ไม่ผูก lifecycle กับเครื่อง

---

## 📑 สารบัญ

- [📖 สรุปสั้น ๆ](#-สรุปสั้น-ๆ)
- [📑 สารบัญ](#-สารบัญ)
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

- ใช้หลาย repo จาก **GitLab**
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
         ├── Git Provider (GitLab)
         ├── Workspace Manager
         └── File System Guard
```

---

### 4.2 Core Modules

| Module                | หน้าที่                                                                      |
| --------------------- | ------------------------------------------------------------------------- |
| **Git Provider**      | GitLab API, Token auth, Repo / branch / permission check                  |
| **Workspace Manager** | clone, delete, lock (ป้องกันลบโดยไม่ตั้งใจ), detect dirty state (uncommitted)  |
| **Safety Guard**      | เตือนเมื่อมี uncommitted changes / unpushed commits, รองรับ optional `--force` |

---

## 5. UX Flow (Desktop / CLI)

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
| **Workspace Profiles**    | dev / review / hotfix — different clone depth / branch |
| **Time-based Auto Close** | ปิดอัตโนมัติหลัง idle X ชั่วโมง                               |
| **Encrypted Temp Vault**  | encrypt workspace folder (LUKS / fscrypt)              |
| **IDE Integration**       | VS Code extension, เปิดผ่าน command โดยตรง               |

---

## 7. What This Project Is NOT

- ❌ Git GUI
- ❌ GitHub Desktop clone
- ❌ Backup tool
- ❌ NAS sync

> **Git BlackVault = lifecycle manager ของ repo**

---

## 8. Tech Stack

| Layer   | Option                      |
| ------- | --------------------------- |
| Core    | Go / Rust                   |
| CLI     | Cobra / Clap                |
| Desktop | Tauri / Electron (optional) |
| Config  | YAML                        |
| Auth    | GitLab Token                |
| OS      | Linux / macOS / Windows     |

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
