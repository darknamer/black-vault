# Git BlackVault — โครงสร้างแอปและ repo

> ย้ายมาจาก [README](../../README.md) §4 App Architecture, §11 Repositories & Setup (ตาราง repo + การเชื่อมต่อ) — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้
>
> ภาพรวมเชิงผลิตภัณฑ์/การไหลของข้อมูลแบบละเอียดดู [product.md](../01-overview/product.md) · โครงตารางข้อมูลดู [datatable.md](../30-reference/datatable.md)

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
| **SQLite Store**      | registry `repos` (scope ต่อบัญชี) + `cache` ที่ `~/.blackvault/blackvault.db` — pure-Go ไม่ต้องใช้ CGO; โครงคอลัมน์ดู [datatable.md](../30-reference/datatable.md) |
| **NAS Solution** *(develop)* | วางแผน sync ไฟล์ workspace ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, นโยบายแก้ conflict/ลบ, โหมด manual/interval/watch — **transfer layer จริงยังไม่ทำ** |
| **Safety Guard**      | กัน path traversal **สองชั้น** — `NormalizeRepoPath` ปฏิเสธ absolute path / `..` / Windows volume ตั้งแต่ต้นทาง แล้ว `SafePathFor` ยืนยันอีกครั้งว่ายังอยู่ใต้ workspace root ก่อนแตะ filesystem ทุกครั้ง; scrub credential ออกจาก error/output. **หมายเหตุ:** `close` **ไม่มี** `--force` และยังไม่ตรวจ dirty state — ดู [13.9](../30-reference/cli-capabilities.md#139-ข้อจำกัดที่รู้-ยังไม่ทำ) |
