# Git BlackVault — เอกสาร (docs)

ดัชนีเอกสารของ repo `black-vault` — จัดเป็นโฟลเดอร์ตามหัวข้อ เลขนำหน้า `00`–`99` (README หลักอยู่ที่ [../README.md](../README.md))

| Band | โฟลเดอร์ | ไฟล์ | เนื้อหา |
| --- | --- | --- | --- |
| **00** | `00-app-structure/` | [architecture.md](00-app-structure/architecture.md) | 5 repo + หน้าที่, การเชื่อมต่อ CLI ↔ GUI (gRPC/proto), §4 App Architecture + Core Modules |
| **01** | `01-overview/` | [concept.md](01-overview/concept.md) | §1 Concept & Philosophy, §2 Core Use Case, §7 What This Project Is NOT, §9 Tagline |
| | | [product.md](01-overview/product.md) | สรุปผลิตภัณฑ์, สถานะต่อ repo, พฤติกรรม runtime, สัญญา gRPC, แผนพัฒนาต่อ (เฟส A–D) |
| **02** | `02-getting-started/` | [quick-start.md](02-getting-started/quick-start.md) | ติดตั้ง/รับ binary, `blackvault init`, workflow ประจำวัน, ไฟล์ใต้ `~/.blackvault/` |
| | | [requirements.md](02-getting-started/requirements.md) | §12 Requirements — เครื่องมือ, layout โฟลเดอร์, บัญชี, ไฟล์รันไทม์, โหมด develop |
| **10** | `10-features/` | [mvp-features.md](10-features/mvp-features.md) | §3 Key Features (MVP), §5 UX Flow (open/close/status), §6 Phase 2 |
| **30** | `30-reference/` | [cli-capabilities.md](30-reference/cli-capabilities.md) | §13 สิ่งที่ CLI ทำได้ตอนนี้ (verify กับโค้ด) + §13.9 ข้อจำกัดที่รู้ |
| | | [datatable.md](30-reference/datatable.md) | โครงตาราง `repos` (SQLite) + metadata ต่อโปรไฟล์ |
| **40** | `40-status/` | [progress.md](40-status/progress.md) | §10 งานที่เหลือ (GAP-1, NAS-1, GH-2/3, IDE-1), สิ่งที่ทำไปแล้วในแต่ละ repo, Tasks ชุดเดิม |

> หมายเหตุ: เลขหัวข้อ §1–§13 คงตาม README เดิม (ก่อนย่อ) เพื่อให้ anchor/ลิงก์เดิมยังตามหาได้

## การเพิ่มเอกสารใหม่

1. วางไฟล์ในโฟลเดอร์ band ที่ตรงหัวข้อ — **ห้ามวางไฟล์หัวข้อไว้ที่ราก `docs/`** (ยกเว้นไฟล์นี้)
2. ถ้าไม่มี band ที่ตรง ให้สร้างโฟลเดอร์ใหม่ `NN-<kebab-slug>/` (เลข 2 หลัก) ตามช่วง:

   | ช่วง | ใช้สำหรับ |
   | --- | --- |
   | `00` | โครงสร้างแอป / repo / สถาปัตยกรรม (สงวนไว้) |
   | `01`–`09` | ภาพรวม, getting started |
   | `10`–`29` | feature / ผลิตภัณฑ์ |
   | `30`–`39` | reference (CLI, API, data model) |
   | `40`–`49` | สถานะ, changelog, release notes |
   | `50`–`69` | engineering guides |
   | `70`–`89` | UX / UI |
   | `90`–`99` | archive / deprecated |

3. เพิ่มแถวในตารางด้านบน และอัปเดตตาราง **📚 เอกสาร** ใน [README หลัก](../README.md) ถ้าเพิ่ม band ใหม่
