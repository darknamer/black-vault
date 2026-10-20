# Git BlackVault — Key Features (MVP), UX Flow และ Phase 2

> ย้ายมาจาก [README](../../README.md) §3 Key Features (MVP), §5 UX Flow, §6 Optional Advanced Features (Phase 2) — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้
>
> §1–§7 เป็น **concept/ปรัชญาของผลิตภัณฑ์** — สถานะจริงของโค้ดดู [cli-capabilities.md](../30-reference/cli-capabilities.md)

## 3. Key Features (MVP)

### 3.1 Repository Registry

- เชื่อม GitLab หรือ GitHub ด้วย Token
- ดึง repo list
- tag / group / project

**โครงข้อมูลจริง** (ตาราง `repos` ใน SQLite — `~/.blackvault/blackvault.db`) และ metadata ต่อโปรไฟล์ใน `config.yaml` — ดู [datatable.md](../30-reference/datatable.md)

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
> ตอนรัน `blackvault serve` จะมีไฟล์เซสชันเพิ่มชั่วคราว: `grpc_port`, `grpc_token`, `grpc_cert.pem` (ลบทิ้งตอนปิด) — ดู [requirements.md §12.4](../02-getting-started/requirements.md#124-ไฟล์รันไทม์ใต้-blackvault)

---

### 3.4 Open Modes

- Open (normal clone)
- Open (shallow clone)
- Open (branch-specific)
- Open (detached / review mode)

---

## 5. UX Flow (Desktop / CLI)

> ครั้งแรกตั้งค่าด้วย `blackvault init` (เลือก provider gitlab/github, workspace, ฯลฯ) — flow หลักคือ open / close / status ด้านล่าง ส่วนคำสั่ง Git (commit/branch/merge/git-flow), `config`, `nas` ฯลฯ ดูรายการเต็มใน [README ของ black-vault-cli](../../../black-vault-cli/README.md)
>
> ขั้นตอนติดตั้ง–ใช้งานแบบลงมือได้จริง ดู [🚀 การใช้งาน CLI (Developer Quick Start)](../02-getting-started/quick-start.md#-การใช้งาน-cli-developer-quick-start) ด้านบน

### 5.1 Open

```bash
blackvault open group/repo
```

- clone ลง `workspaces/<profile>/<group>/<repo>/`
- ใช้ **default branch policy** อัตโนมัติ: เตรียม `main` (ไม่มีก็ลอง `master`) แล้ว**ถ้ามี `origin/develop` จะจบที่ `develop`** — เป็น best-effort, ระบุ `--branch`/`--detached` เอง = ข้าม policy
- พิมพ์ path ที่ clone ลง พร้อมบรรทัด `On branch: <b>`

> ยังไม่มีการเปิด IDE ให้อัตโนมัติ — ดู [13.9](../30-reference/cli-capabilities.md#139-ข้อจำกัดที่รู้-ยังไม่ทำ)

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

> **Workspace Profiles ทำเสร็จแล้ว** และย้ายออกจากตารางนี้ — ดู [13.3](../30-reference/cli-capabilities.md#133-หลายบัญชี-และโปรไฟล์-workspace)

| Feature                   | รายละเอียด                                              |
| ------------------------- | ------------------------------------------------------ |
| **NAS Sync Solution** *(เริ่มแล้ว — develop)* | `type: nas` — sync ไฟล์ขึ้น NAS (rsync/ssh/ftp/smb/nfs), `.nasignore`, manual/interval/watch, นโยบายแก้ conflict/ลบ; **วางแผน + dry-run ได้แล้ว เหลือ transfer layer จริง** |
| **Dirty-state detection** | ตรวจว่ามีการแก้ที่ยังไม่ commit/push ก่อนปิด — ช่อง `dirty` มีอยู่ใน output แล้วแต่ยังคืน `false` เสมอ |
| **Time-based Auto Close** | ปิดอัตโนมัติหลัง idle X ชั่วโมง                               |
| **Encrypted Temp Vault**  | encrypt workspace folder (LUKS / fscrypt)              |
| **IDE Integration**       | VS Code extension, เปิดผ่าน command โดยตรง               |
