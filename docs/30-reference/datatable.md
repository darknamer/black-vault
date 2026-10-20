# Git BlackVault — โครงข้อมูล (Data Table)

> โครงข้อมูลจริงที่ระบบใช้เก็บทะเบียน repo — ย้ายมาจาก README §3.1 (ปัจจุบันอยู่ที่ [mvp-features.md §3.1 Repository Registry](../10-features/mvp-features.md#31-repository-registry))

---

## ตาราง `repos` (SQLite — `~/.blackvault/blackvault.db`)

```
git_account      -- id ของบัญชีที่เป็นเจ้าของแถว (composite PK ร่วมกับ id)
id               -- รูป slash เช่น group/repo หรือ group/subgroup/repo
name
group_name
default_branch
last_opened
status (closed | active)
```

> registry นี้แยก scope **ต่อบัญชี** — เปลี่ยนบัญชีแล้วรายการ repo เปลี่ยนตาม
> อีกชุดหนึ่งคือ metadata ต่อโปรไฟล์ที่เก็บใน `config.yaml` (`workspace.repositories[<profile>]`) จัดการผ่าน `workspace repo` — ดู [cli-capabilities.md §13.3](cli-capabilities.md#133-หลายบัญชี-และโปรไฟล์-workspace)
