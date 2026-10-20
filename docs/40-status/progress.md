# Git BlackVault — สถานะและงานที่เหลือ

> ย้ายมาจาก [README](../../README.md) §10 งานที่เหลือจริง (Next Steps) และ §11 (สิ่งที่ทำไปแล้วในแต่ละ repo, สถานะของ Tasks ชุดเดิม) — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้

## 10. งานที่เหลือจริง (Next Steps)

CLI spec, `config.yaml` และ MVP ถูก implement ไปแล้ว (ดู [11](../00-app-structure/architecture.md#11-repositories--setup) และ [13](../30-reference/cli-capabilities.md#13-สิ่งที่-cli-ทำได้ตอนนี้)) — ที่เหลือคือ:

| # | งาน | อยู่ที่ | สถานะ |
| --- | --- | --- | --- |
| GAP-1 | คำนวณ **dirty state** จริง (`Status()` ยังคืน `Dirty=false` เสมอ) แล้วเปิดทาง `close` ให้เตือน/ถามยืนยันได้ | lib + cli | ยังไม่เริ่ม |
| NAS-1 | **transfer layer** ของ solution NAS (ตอนนี้ dry-run/วางแผนได้ แต่ sync จริงคืน error `not implemented yet`) + ตัวอ่าน state ฝั่ง remote เพื่อต่อกับ plan engine | lib | ค้างอยู่ |
| GH-2 | แยก **workspace/registry ต่อ provider** — path ยังเป็น `<profile>/<group>/<repo>` และตาราง `repos` ยังไม่มีคอลัมน์ provider จึงยังไม่ isolate กรณี id ชนกันระหว่าง GitHub กับ GitLab ในโปรไฟล์เดียว | lib | ยังไม่เริ่ม |
| GH-3 | flag `--provider` ต่อคำสั่ง | cli | ยังไม่เริ่ม |
| IDE-1 | เปิด IDE อัตโนมัติหลัง `open` (`BLACKVAULT_OPEN_IDE` ถูกอ่านแล้วแต่ยังเป็น no-op) | cli | ยังไม่เริ่ม |

รายละเอียดเชิงข้อกำหนดแบบเต็ม (testable "shall" + citation จากโค้ด + known gaps ของทั้ง 4 repo) อยู่ที่ [black-vault-cli/docs/01-overview/requirements.md](../../../black-vault-cli/docs/01-overview/requirements.md)

---

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

งานที่เหลือหลังจากนี้ดู [10. งานที่เหลือจริง](#10-งานที่เหลือจริง-next-steps) และข้อจำกัดปัจจุบันดู [13.9](../30-reference/cli-capabilities.md#139-ข้อจำกัดที่รู้-ยังไม่ทำ)
