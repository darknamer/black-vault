# Git BlackVault — การใช้งาน CLI (Developer Quick Start)

> ย้ายมาจาก [README](../../README.md) §🚀 การใช้งาน CLI (Developer Quick Start) — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้
>
> สิ่งที่ต้องมีก่อน build/รันจริง ดู [requirements.md](requirements.md)

## 🚀 การใช้งาน CLI (Developer Quick Start)

ผู้ใช้ปลายทางใช้งานผ่าน **binary `blackvault`** ซึ่ง build จาก repo [`black-vault-cli`](../../../black-vault-cli/README.md) — ส่วน [`black-vault-lib`](../../../black-vault-lib/README.md) คือ engine ภายใน (สนใจเฉพาะเมื่อจะ contribute โค้ด)
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

รายละเอียดเต็ม (command reference ~2000 บรรทัด) อยู่ใน [README ของ black-vault-cli](../../../black-vault-cli/README.md):

- ตัวอย่างคำสั่งทั้งหมด → [ตัวอย่างคำสั่ง](../../../black-vault-cli/docs/02-getting-started/usage.md#usage-examples)
- รายละเอียด `init` + flags ทั้งหมด → [เริ่มต้นใช้งานด้วย init](../../../black-vault-cli/docs/02-getting-started/usage.md#เริ่มต้นใช้งานด้วย-init)
- รายละเอียดการ build / cross-compile → [การ build](../../../black-vault-cli/docs/02-getting-started/building.md#build)
- หลาย account (git/NAS) → [account](../../../black-vault-cli/docs/10-commands/commands-reference.md#account-multi)
- workspace profiles → [workspace profile](../../../black-vault-cli/docs/10-commands/commands-reference.md#workspace-profiles)
- bulk open/close (`--all`) → [bulk open/close](../../../black-vault-cli/docs/10-commands/implemented-commands.md#bulk-open-close)
- gRPC server สำหรับ GUI → [GUI กับ serve](../../../black-vault-cli/docs/02-getting-started/usage.md#gui-กับ-serve)
- internals ของ lib (สำหรับ contributor) → [README ของ black-vault-lib](../../../black-vault-lib/README.md)
