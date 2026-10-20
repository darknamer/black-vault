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
- รองรับทั้ง **GitLab และ GitHub** (ตั้ง `git_provider` ของบัญชีที่ใช้งาน)
- **ผู้ใช้โต้ตอบผ่าน CLI `blackvault`** จาก repo [`black-vault-cli`](../black-vault-cli/README.md) — ส่วน `black-vault-lib` เป็น implementation detail (สนใจเฉพาะเมื่อจะ contribute) → เริ่มที่ [🚀 การใช้งาน CLI (Developer Quick Start)](#-การใช้งาน-cli-developer-quick-start)
- _(กำลังพัฒนา)_ solution ทดลอง **`type: nas`** สำหรับ sync ไฟล์ขึ้น NAS — เปิดเฉพาะโหมด develop (`BLACKVAULT_DEV=1`)

> 📌 **อ่านก่อนลงมือ:** สิ่งที่ต้องมีเพื่อ build/รันจริง ดู [docs/02-getting-started/requirements.md](docs/02-getting-started/requirements.md) · สรุปว่า CLI **ทำอะไรได้แล้วตอนนี้** (verify กับโค้ดจริง) ดู [docs/30-reference/cli-capabilities.md](docs/30-reference/cli-capabilities.md)
>
> รายละเอียดทั้งหมดย้ายไปอยู่ใน [`docs/`](docs/README.md) — README นี้เป็นแค่ดัชนี

---

## 🚀 การใช้งาน CLI (Developer Quick Start)

ผู้ใช้ปลายทางใช้งานผ่าน **binary `blackvault`** ซึ่ง build จาก repo [`black-vault-cli`](../black-vault-cli/README.md) — ดาวน์โหลดจาก GitHub Releases ของ `black-vault-cli` (6 targets: linux / windows / darwin × amd64 / arm64) หรือ build จาก source (Go 1.24.x + `black-vault-lib` เป็น sibling directory)

```bash
blackvault init                       # ตั้งค่าครั้งแรก (provider, token, workspace) → ~/.blackvault/config.yaml
blackvault open group/repo            # clone ลง workspace
blackvault git commit group/repo -m "ข้อความ"   # คำสั่ง git โดยไม่ต้อง cd
blackvault status                     # repo ไหน active / closed
blackvault close group/repo           # ปิด workspace — push เองก่อนปิดเสมอ
```

ขั้นตอนเต็ม (ติดตั้ง, flag ของ `init`, workflow ประจำวัน, ไฟล์ใต้ `~/.blackvault/`, หัวข้อ advanced) → **[docs/02-getting-started/quick-start.md](docs/02-getting-started/quick-start.md)**

---

## 🧩 Repositories

| Repo | ภาษา | หน้าที่ |
| --- | --- | --- |
| **black-vault** | — | เอกสารและ concept (repo นี้) |
| **black-vault-lib** | Go | Library — logic ทั้งหมด (config, workspace, clone, SQLite store, GitLab/GitHub client, NAS) + `.proto` |
| **black-vault-cli** | Go | CLI (Cobra) — ตัวห่อบาง ๆ ของ lib + `serve` (gRPC ให้ GUI) |
| **black-vault-gui** | Flutter | Desktop GUI — gRPC client ไปที่ `blackvault serve` |
| **black-vault-api** | C# / ASP.NET | Backend ระบบ **License** |

สถาปัตยกรรม, core modules และการเชื่อมต่อ CLI ↔ GUI → **[docs/00-app-structure/architecture.md](docs/00-app-structure/architecture.md)**

---

## 🛠️ Tech Stack

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

## 📚 เอกสาร (Documentation)

ดัชนีเต็มอยู่ที่ **[docs/README.md](docs/README.md)**

| Band | โฟลเดอร์ | เนื้อหา |
| --- | --- | --- |
| 00 | [`00-app-structure/`](docs/00-app-structure/architecture.md) | โครงสร้าง repo, สถาปัตยกรรม, core modules, การเชื่อมต่อ CLI ↔ GUI |
| 01 | [`01-overview/`](docs/01-overview/concept.md) | concept & philosophy, use case, สิ่งที่ไม่ใช่, สรุปผลิตภัณฑ์ + แผนพัฒนา ([product.md](docs/01-overview/product.md)) |
| 02 | [`02-getting-started/`](docs/02-getting-started/quick-start.md) | quick start ของ CLI, requirements ([requirements.md](docs/02-getting-started/requirements.md)) |
| 10 | [`10-features/`](docs/10-features/mvp-features.md) | Key Features (MVP), UX flow open/close/status, Phase 2 |
| 30 | [`30-reference/`](docs/30-reference/cli-capabilities.md) | สิ่งที่ CLI ทำได้ตอนนี้ (verify กับโค้ด), โครงข้อมูล ([datatable.md](docs/30-reference/datatable.md)) |
| 40 | [`40-status/`](docs/40-status/progress.md) | งานที่เหลือ (Next Steps), สิ่งที่ทำไปแล้วในแต่ละ repo |

---

## 📌 สถานะ (Status)

MVP + gRPC CLI ↔ GUI ทำครบแล้ว — งานที่เหลือหลัก:

- **GAP-1** คำนวณ dirty state จริง (`Status()` ยังคืน `Dirty=false` เสมอ) — **push เองก่อนปิดเสมอ**
- **NAS-1** transfer layer ของ solution NAS (ตอนนี้ dry-run / วางแผนได้เท่านั้น)
- **GH-2 / GH-3** แยก workspace/registry ต่อ provider, flag `--provider`
- **IDE-1** เปิด IDE อัตโนมัติหลัง `open`

รายละเอียด → [docs/40-status/progress.md](docs/40-status/progress.md) · ข้อจำกัดที่รู้ → [cli-capabilities.md §13.9](docs/30-reference/cli-capabilities.md#139-ข้อจำกัดที่รู้-ยังไม่ทำ)
