# Git BlackVault — Concept & Philosophy

> ย้ายมาจาก [README](../../README.md) §1 Concept & Philosophy, §2 Core Use Case, §7 What This Project Is NOT, §9 Tagline Ideas — คงเลขหัวข้อเดิมไว้เพื่อให้ลิงก์ที่อ้างถึงยังใช้ได้

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

## 7. What This Project Is NOT

- ❌ Git GUI
- ❌ GitHub Desktop clone
- ❌ Backup tool
- ❌ NAS sync *(โดยค่าเริ่มต้น `type: git` — มี solution ทดลอง `type: nas` แยกต่างหากในโหมด develop)*

> **Git BlackVault = lifecycle manager ของ repo**

---

## 9. Tagline Ideas

- *Open what you need. Burn the rest.*
- *Repos are temporary. Control is permanent.*
- *No sync. No clutter. Just work.*
