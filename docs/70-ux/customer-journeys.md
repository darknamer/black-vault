# Git BlackVault — Customer Journeys (CLI)

[← กลับ docs](../README.md)

> 10 เส้นทางที่ผู้ใช้จะเจอจริงเมื่อใช้ `blackvault` — ใช้เป็นฐานจัดลำดับงาน UX และ known gaps
>
> **ทดสอบเมื่อ:** 2026-10-20 · `black-vault-cli` commit `a80a4a1` (build ในเครื่อง, รันใน `HOME` ว่างเพื่อจำลองผู้ใช้ใหม่)
> **อ้างอิงเพิ่ม:** [black-vault-cli/docs/01-overview/requirements.md](../../../black-vault-cli/docs/01-overview/requirements.md) §9 Known gaps

**สัญลักษณ์**

- 🔬 = รันจริงแล้วเห็นผลนี้ · 📄 = อ้างอิงจากโค้ด/เอกสาร (ทดสอบไม่ได้ในรอบนี้ — ต้องมี token จริงหรือ GUI)
- อารมณ์ผู้ใช้: 😀 ดี · 😐 เฉย ๆ · 😣 ติดขัด · 😱 เสี่ยงเสียงาน

---

## 1. ติดตั้งครั้งแรก — "ตั้งค่าเสร็จแล้วเหรอ?"

**ผู้ใช้:** dev คนใหม่ ดาวน์โหลด binary มาลองใช้

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| รัน `blackvault` | 🔬 เห็นสรุป account `https://gitlab.com`, profile `git-default` ดูเหมือนตั้งค่าไว้แล้ว | 😀 |
| รัน `blackvault status` | 🔬 ขึ้น `connectivity OK` ทั้งที่ยังไม่ได้ใส่ token | 😐 |
| รัน `blackvault doctor` | 🔬 `Error: config not found ... run 'blackvault init' first` | 😣 |
| รัน `blackvault init` | 📄 ถามทีละขั้น แต่ token ที่พิมพ์ **มองเห็นบนจอ** (GAP-14) | 😐 |

- **Pain point:** แต่ละคำสั่งบอกสถานะไม่ตรงกัน สองคำสั่งแรกบอกว่าพร้อม แต่ `doctor` บอกว่ายังไม่ได้ `init`
- **โอกาสปรับปรุง:** ถ้ายังไม่มี config ทุกคำสั่งควรพาไปที่ `init` เหมือนกัน และซ่อน token ตอนพิมพ์

## 2. Open ครั้งแรกตอนยังไม่มี token — "ค้างไปเลย"

**ผู้ใช้:** ข้ามขั้น `init` แล้วลอง open ทันที

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `blackvault search` | 🔬 error ชัดเจน: `git credentials not configured ... 'blackvault config set git_credentials <token>'` | 😀 |
| `blackvault open` (ใช้ picker) | 🔬 error ชัดเจนเหมือนกัน | 😀 |
| `blackvault open acme/frontend` | 🔬 **ค้างนานเกิน 2 นาที ไม่มี progress หรือข้อความใด ๆ** ต้องกด Ctrl+C เอง | 😱 |

- **Pain point:** `open` ที่ระบุ repo ไม่ตรวจ credentials ก่อน clone เหมือน `search` (คาดว่า git รอ prompt username/password อยู่เบื้องหลัง — ยังไม่ได้ยืนยันสาเหตุ)
- **โอกาสปรับปรุง:** ตรวจ credentials ก่อน clone, ตั้ง `GIT_TERMINAL_PROMPT=0` และมี timeout พร้อมข้อความบอกผู้ใช้

## 3. Open สำเร็จแล้ว — "โฟลเดอร์อยู่ไหน?"

**ผู้ใช้:** ตั้งค่าครบแล้ว เปิด repo แรก

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `blackvault open` | 📄 picker ให้เลือก repo จาก remote สะดวก | 😀 |
| clone เสร็จ | 📄 มี progress และขึ้นว่าเปิดสำเร็จ | 😀 |
| อยากเปิดใน IDE | 📄 ต้อง `cd` ไป path ยาว `~/.blackvault/workspaces/git-default/<group>/<repo>` เอง | 😣 |
| อ่าน help ละเอียดจนเจอ `BLACKVAULT_OPEN_IDE=1` | 📄 ฟีเจอร์ดี แต่ซ่อนอยู่ใน environment variable | 😐 |

- **โอกาสปรับปรุง:** เพิ่ม `blackvault path <repo>` (ใช้กับ `cd $(blackvault path acme/frontend)`) และ flag `open --ide cursor|code|idea`

## 4. Commit/push ประจำวัน — "สุดท้ายก็กลับไปใช้ git ปกติ"

**ผู้ใช้:** อยู่ในโฟลเดอร์ repo แล้ว จะ commit งาน

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `blackvault git commit -m "fix"` | 🔬 `Error: accepts 1 arg(s), received 0` | 😣 |
| พิมพ์ Repo ID เต็ม | 📄 commit ได้ แต่ **สั่ง `git add -A` ให้อัตโนมัติ** (ต้องใส่ `--no-add` เอง) ทำให้ไฟล์ `.env` หรือไฟล์ชั่วคราวติดไปด้วย | 😱 |
| ครั้งต่อไป | ผู้ใช้เลิกใช้แล้วพิมพ์ `git commit` ปกติแทน | 😐 |

- **Pain point:** คำสั่ง Git ของ BlackVault ใช้ยากกว่า `git` ปกติ ผู้ใช้จึงไม่มีเหตุผลที่จะใช้
- **โอกาสปรับปรุง:** หา Repo ID จากโฟลเดอร์ปัจจุบัน (เทียบกับ Registry), ไม่ add ทุกไฟล์เป็นค่าเริ่มต้น และเพิ่มคำสั่งที่ `git` ปกติทำไม่ได้ เช่น `git pull --all`, `git status --all`

## 5. เลิกงานแล้วล้างเครื่อง — "Burn the rest… แต่เผางานตัวเองไปด้วย"

**ผู้ใช้:** ต้องการคืนพื้นที่ disk ตอนเลิกงาน

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `blackvault close acme/frontend` | 📄 ค่าเริ่มต้นเก็บไฟล์ไว้ **disk จึงยังไม่ถูกล้าง** | 😐 |
| `close --all --purge` | 📄 ลบทันที **ไม่ตรวจงานที่ยังไม่ commit หรือยังไม่ push และไม่ถามยืนยัน** (GAP-1, GAP-2) | 😱 |
| พิมพ์ชื่อ repo ผิด หรือ close repo ที่ไม่เคย open | 🔬 ขึ้น `Closed acme/frontend (kept files)` เหมือนทำสำเร็จ | 😐 |

- **Pain point:** จุดที่ควรปลอดภัยที่สุดของแนวคิด "เผาทิ้ง" กลับไม่มีอะไรป้องกันเลย
- **โอกาสปรับปรุง:** ตรวจงานค้าง (uncommitted + unpushed) ก่อนลบพร้อมถามยืนยัน, มี `--force` สำหรับข้าม และขึ้น error เมื่อ close repo ที่ไม่อยู่ใน Registry

## 6. ชื่อโฟลเดอร์ซ้ำใน IDE — "web, web, web"

**ผู้ใช้:** ทีมที่มี repo ชื่อ `web` และ `api` หลายตัวใน group ต่างกัน

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| เปิดหลาย repo แล้วใน IDE เห็นแท็บ `web` ซ้ำกัน | สับสนว่าตัวไหนเป็นตัวไหน | 😣 |
| `workspace prefix set portal` จาก home | 🔬 `Error: path ... is outside workspace profile root` ข้อความชัด แต่ต้องรู้ก่อนว่าต้อง `cd` เข้าไปใน workspace | 😐 |
| สั่งอีกครั้งใน path ที่ถูก แล้ว `workspace repo sync` | 📄 ย้ายโฟลเดอร์เป็น `portal-bo-web` ถ้าชื่อชนกันจะ error โดยไม่ย้ายอะไรเลย | 😀 |
| ถ้าเปิด IDE ค้างไว้ระหว่าง sync | 📄 IDE อาจยังชี้ไปที่ path เดิม | 😐 |

- **ความเห็น:** เป็น journey ที่ **ออกแบบมาดีที่สุด** ในระบบ แก้ปัญหาจริงได้อย่างปลอดภัย ([ADR 0001](../../../black-vault-cli/docs/adr/0001-config-driven-layout-and-prefix.md))
- **โอกาสปรับปรุง:** ให้ `prefix set` ถามต่อว่า "sync now? [Y/n]" และเตือนให้ปิด IDE ก่อน sync

## 7. งานบริษัทและงานส่วนตัว — "สลับบัญชีได้ลื่น"

**ผู้ใช้:** ใช้ GitLab ของบริษัทและ GitHub ส่วนตัว

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `account add` ทีละบัญชี | 📄 เลือกชนิด git ก่อน แล้วสร้าง id จาก URL ให้อัตโนมัติ | 😀 |
| `workspace profile add work` / `personal` | 📄 แต่ละโปรไฟล์ผูก 1 บัญชี และโฟลเดอร์แยกกัน | 😀 |
| `open --profile personal me/blog` | 📄 สลับใช้เฉพาะคำสั่งนั้น ไม่เปลี่ยนโปรไฟล์ที่ active | 😀 |
| ผูก repo จาก 2 provider ในโปรไฟล์เดียว | 📄 Repo ID ชื่อเดียวกันจะชนกันบน disk (GAP-4 / GH-2) | 😐 |

- **ความเห็น:** ใช้งานได้ดีถ้าแยก 1 โปรไฟล์ต่อ 1 provider — ควรเขียนคำแนะนำนี้ไว้ใน [quick-start](../02-getting-started/quick-start.md)

## 8. Onboarding สมาชิกใหม่ทั้งทีม — "เปิดทุก repo ในคำสั่งเดียว"

**ผู้ใช้:** หัวหน้าทีมให้คนใหม่ดึง repo ทั้งหมดมาใช้

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `open --all --shallow` | 📄 clone ทุก repo ที่ token เข้าถึงได้ ข้ามตัวที่เปิดอยู่แล้ว และมี progress | 😀 |
| token เห็นหลายร้อยโปรเจกต์ | 📄 มี `--limit` แต่ **กรองตาม group ฝั่ง API ไม่ได้** (GAP-9) จึงอาจ clone เกินจำเป็น | 😣 |
| `list --json` เพื่อทำ script ต่อ | 🔬 ผลลัพธ์รูปแบบคงที่ ใช้ใน CI ได้ดี | 😀 |

- **โอกาสปรับปรุง:** เพิ่ม `open --all --group postway/portal` และแสดงจำนวนกับขนาดโดยประมาณก่อนเริ่ม clone

## 9. ใช้ผ่าน GUI — "กดปิดจาก GUI แล้วไฟล์หาย"

**ผู้ใช้:** ใช้ `black-vault-gui` แทน terminal

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| เปิด GUI | 📄 GUI เปิด `serve` ให้เอง หาพอร์ตอัตโนมัติ และเชื่อมผ่าน TLS กับ token | 😀 |
| หน้า Setup (บัญชีและโปรไฟล์) | 📄 **แค่แสดงคำสั่ง CLI ให้ไปรันใน terminal เอง** (GAP-7) | 😣 |
| Open private repo ผ่าน SSH | 📄 RPC `OpenWorkspace` ไม่มี clone_mode จึงทำไม่ได้ (GAP-12) | 😣 |
| กด Close | 📄 **gRPC `CloseWorkspace` purge ทุกครั้ง** ต่างจาก CLI ที่เก็บไฟล์เป็นค่าเริ่มต้น (`black-vault-cli/cmd/serve_handlers.go`) | 😱 |

- **Pain point:** ผู้ใช้ที่ชินกับ CLI จะคาดว่า Close ไม่ลบไฟล์ แต่การกด Close ใน GUI จะลบไฟล์
- **โอกาสปรับปรุง:** ให้ค่าเริ่มต้นของ GUI ตรงกับ CLI และเพิ่ม RPC สำหรับตั้งค่าบัญชีและโปรไฟล์

## 10. แก้ปัญหาเอง — "doctor บอกผ่าน แต่ commit ไม่ได้"

**ผู้ใช้:** เครื่อง Windows ที่ไม่มี git ติดตั้ง

| ขั้นตอน | สิ่งที่เจอ | อารมณ์ |
|---|---|:-:|
| `open` | 📄 clone ได้ เพราะมี go-git ฝังในตัวเป็นตัวสำรอง | 😀 |
| `git commit` | 📄 ล้มเหลว เพราะคำสั่ง Git ต้องใช้ git binary จริง | 😣 |
| `doctor` | 📄 git executable รายงาน **OK "go-git (embedded fallback)"** (GAP-13) | 😣 |
| ค้นจนเจอ `install-git` | 📄 แก้ได้ แต่ผู้ใช้ต้องหาเอง | 😐 |
| สังเกตเพิ่ม | 🔬 runtime error ทุกตัวพิมพ์ Usage ยาวต่อท้าย ทำให้ข้อความ error หลักจมหาย | 😐 |

- **โอกาสปรับปรุง:** ให้ `doctor` รายงาน WARN พร้อมแนะนำ `install-git` และตั้ง `SilenceUsage` ให้ runtime error ไม่พิมพ์ Usage

---

## สรุปภาพรวม

| # | Journey | อารมณ์โดยรวม | ระดับปัญหา |
|---|---|:-:|:-:|
| 1 | ติดตั้งครั้งแรก | 😐 | กลาง |
| 2 | Open ตอนยังไม่มี token | 😱 | **สูง** |
| 3 | หาโฟลเดอร์หลัง open | 😐 | กลาง |
| 4 | Commit/push ประจำวัน | 😣 | **สูง** |
| 5 | Close/purge ล้างเครื่อง | 😱 | **สูงมาก** |
| 6 | แก้ชื่อซ้ำด้วย prefix | 😀 | ต่ำ |
| 7 | หลายบัญชี/โปรไฟล์ | 😀 | ต่ำ |
| 8 | Onboarding ทั้งทีม | 😀 | กลาง |
| 9 | ใช้ผ่าน GUI | 😱 | **สูง** |
| 10 | แก้ปัญหาด้วย doctor | 😣 | กลาง |

ระบบทำได้ดีในเรื่อง **"จัดระเบียบ"** (journey 6, 7, 8) แต่ยังอ่อนในเรื่อง **"ปลอดภัยต่องานของผู้ใช้"** (journey 2, 5, 9) และ **"ใช้ทุกวันได้สะดวก"** (journey 3, 4)

### ลำดับงานที่แนะนำ (เรียงตามผลต่อผู้ใช้)

| ลำดับ | งาน | Journey | แก้ที่ |
|:-:|---|:-:|---|
| 1 | ตรวจงานค้างและถามยืนยันก่อน purge ทั้งใน CLI และ GUI | 5, 9 | lib + cli + gui |
| 2 | ตรวจ credentials และตั้ง timeout ก่อน clone | 2 | lib |
| 3 | หา repo จากโฟลเดอร์ปัจจุบัน และไม่ add ทุกไฟล์อัตโนมัติ | 4 | lib + cli |
| 4 | ทุกคำสั่งพาไป `init` เมื่อยังไม่มี config, `doctor` บอกตามจริงเรื่อง git binary | 1, 10 | lib + cli |
| 5 | เพิ่ม `blackvault path` และ flag `open --ide` | 3 | cli |
