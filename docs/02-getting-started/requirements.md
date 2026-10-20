# 12. Requirements (สิ่งที่ต้องมี)

> ย้ายมาจาก [README §12](../README.md#12-requirements-สิ่งที่ต้องมี) — คงเลขหัวข้อเดิม (12.1–12.6) ไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้

ข้อกำหนดสำหรับ **build และรันของจริง** ณ สถานะปัจจุบันของ repo (ไม่ใช่แผน) — spec เชิงข้อกำหนดแบบเต็มอยู่ที่ [black-vault-cli/docs/REQUIREMENTS.md](../../black-vault-cli/docs/REQUIREMENTS.md)

---

## สารบัญ

- [12.1 เครื่องมือที่ต้องมี](#121-เครื่องมือที่ต้องมี)
- [12.2 โครงสร้างโฟลเดอร์ที่บังคับ](#122-โครงสร้างโฟลเดอร์ที่บังคับ)
- [12.3 บัญชีและสิทธิ์](#123-บัญชีและสิทธิ์)
- [12.4 ไฟล์รันไทม์ใต้ ~/.blackvault/](#124-ไฟล์รันไทม์ใต้-blackvault)
- [12.5 โหมด develop](#125-โหมด-develop)
- [12.6 ตรวจว่าพร้อมใช้งาน](#126-ตรวจว่าพร้อมใช้งาน)

---

## 12.1 เครื่องมือที่ต้องมี

| สิ่งที่ต้องมี | เวอร์ชัน | จำเป็นกับ | หมายเหตุ |
| --- | --- | --- | --- |
| **Go** | **1.24.x** (`toolchain go1.24.2`) | `black-vault-lib`, `black-vault-cli` | ตรวจด้วย `go version` ก่อน build |
| **git** (executable) | ทั่วไป | clone + คำสั่ง `blackvault git ...` | ใช้ของระบบ (PATH) หรือ portable ที่ `~/.blackvault/tools/git` |
| **protoc** + `protoc-gen-go` + `protoc-gen-go-grpc` | — | **เฉพาะเมื่อแก้ `.proto`** | ไฟล์ `.pb.go` commit ไว้ใน repo แล้ว — build ปกติ **ไม่ต้องมี** |
| **Flutter** (desktop) + `protoc_plugin` | stable | `black-vault-gui` | `protoc_plugin` เฉพาะตอน regenerate Dart |
| **.NET / ASP.NET Core** | — | `black-vault-api` | ระบบ License แยกต่างหาก |
| **CGO** | **ไม่ต้อง** | — | SQLite เป็น pure-Go (`modernc.org/sqlite`) — build ด้วย `CGO_ENABLED=0` ได้ |
| **OS** | — | ทั้งหมด | Linux / macOS / Windows |

> ⚠️ **เรื่อง git ที่ต้องรู้:** ถ้าเครื่อง**ไม่มี git binary เลย** การ `open` (clone) ยังทำได้ผ่าน **go-git fallback** ที่ฝังอยู่ใน lib
> แต่ **คำสั่ง `blackvault git ...` ทุกตัวใช้ไม่ได้** (commit/push/branch/merge/flow) เพราะทั้งหมดต้องเรียก git ตัวจริง — จะได้ error ว่าหา git executable ไม่เจอ
> ลำดับการหา git: ค่าใน config (`git_path`) → `git` บน PATH → portable ที่ `~/.blackvault/tools/git` → go-git

## 12.2 โครงสร้างโฟลเดอร์ที่บังคับ

`black-vault-cli/go.mod` มี `replace github.com/darknamer/black-vault-lib => ../black-vault-lib` ดังนั้น **สอง repo ต้องอยู่ข้างกัน** ไม่งั้น `go build` / `go test` จะล้มทันที:

```text
<parent>/
├── black-vault/        # repo นี้ — เอกสาร/concept
├── black-vault-lib/    # ต้องอยู่ตรงนี้ — ผูกกับ replace ใน go.mod ของ cli
├── black-vault-cli/
├── black-vault-gui/
└── black-vault-api/
```

CI ของทั้ง GitHub Actions และ GitLab CI ก็ checkout `black-vault-lib` เป็น sibling ด้วยรูปแบบเดียวกัน

## 12.3 บัญชีและสิทธิ์

| ต้องมี | ใช้ตอนไหน |
| --- | --- |
| **Personal Access Token** ของ GitLab (ส่งเป็น `PRIVATE-TOKEN`) **หรือ** GitHub (ส่งเป็น `Bearer`) | ดึงรายการ repo (`git repo-list`, `workspace repo list`, `open --all`), ตรวจ connectivity (`install`, `status`), clone repo private |
| **GitHub Enterprise**: ตั้ง `git_url` เป็น host ขององค์กร | ระบบจะ resolve API เป็น `<git_url>/api/v3` ให้เอง |
| **ssh agent / key** | เฉพาะเมื่อ clone ด้วย `--clone-mode ssh` (ไม่ใช้ token) |

> 🔐 **ข้อควรระวังด้านความปลอดภัย:** token ถูกเก็บเป็น **plaintext** ใน `~/.blackvault/config.yaml` (ไฟล์เขียนด้วย mode `0600`)
> และโหมด `--clone-mode https-auth` จะ**ฝัง `user:token` ลงใน origin URL** ซึ่งถูกบันทึกลง `.git/config` และมองเห็นได้จาก `git remote -v` / `ps` — เป็นพฤติกรรมที่ตั้งใจ (ให้ repo private ใช้งานได้เองโดยไม่ต้องมี credential helper) แต่ควรรู้ก่อนใช้บนเครื่องที่ใช้ร่วมกัน
> error/output ที่ระบบพิมพ์ออกมาจะถูก scrub credential เป็น `***:***@` ให้เสมอ

## 12.4 ไฟล์รันไทม์ใต้ `~/.blackvault/`

lib เป็นผู้จัดการไฟล์เหล่านี้ทั้งหมด (CLI/GUI ไม่เขียนเอง):

```text
~/.blackvault/
├── config.yaml        # บัญชี + โปรไฟล์ + cache/output settings (mode 0600)
├── blackvault.db      # SQLite: ตาราง repos (registry) + cache — สร้างอัตโนมัติถ้าไม่มี
├── cache/
├── tools/git/         # portable git (ถ้าใช้; เตรียมด้วย `blackvault install-git`)
├── workspaces/
│   └── <profile>/<group>/<repo>/
├── grpc_port          # ┐ สร้างตอน `blackvault serve` (ทั้งสามไฟล์ mode 0600)
├── grpc_token         # │ GUI อ่านไปใช้ต่อ gRPC
└── grpc_cert.pem      # ┘ ลบทิ้งอัตโนมัติตอนปิด serve แบบ graceful
```

> โครงสร้างคอลัมน์ของตาราง `repos` ใน `blackvault.db` ดู [DATATABLE.md](DATATABLE.md)

## 12.5 โหมด develop

ตั้ง `BLACKVAULT_DEV=1` เพื่อปลดล็อกส่วนที่ยังพัฒนาไม่เสร็จ:

- คำสั่ง `blackvault nas` (ไม่ตั้งจะไม่ถูก register เลย — ไม่ขึ้นใน `--help`)
- `type: nas` และการเพิ่มบัญชีชนิด NAS
- คีย์ config ของ NAS (`nas_protocol`, `nas_url`, `nas_username`, `nas_password`, `nas_path`, `nas_port`, `nas_sync_mode`, `nas_sync_interval`, `nas_conflict_policy`, `nas_delete_policy`)

## 12.6 ตรวจว่าพร้อมใช้งาน

```bash
blackvault init      # ตั้ง config ครั้งแรก (interactive หรือสั่งผ่าน flag) — จะเรียก install ให้เองแบบ best-effort
blackvault install   # bootstrap: สร้างไดเรกทอรีที่ขาด + ping git host จริงทุกบัญชี
blackvault doctor    # วินิจฉัยแบบอ่านอย่างเดียว — ไม่สร้างไดเรกทอรี ไม่แก้ DB ไม่ ping remote
```

| คำสั่ง | มี side effect ไหม | ใช้ตอนไหน |
| --- | --- | --- |
| `install` | **มี** — สร้างไดเรกทอรี + เรียก network (ping) | หลังตั้งค่าใหม่ หรือเปลี่ยน credentials |
| `doctor` | **ไม่มี** — อ่านอย่างเดียว | เช็คสถานะประจำ / ตอนหาสาเหตุปัญหา |

`doctor` รายงานเป็น `OK` / `WARN` / `FAIL` — เฉพาะ `FAIL` (เช่น เปิด SQLite ไม่ได้, ไม่มีบัญชี git) ที่ทำให้ exit code ไม่เป็นศูนย์ ส่วน `WARN` (เช่น `serve` ยังไม่รัน) เป็นแค่คำเตือน
