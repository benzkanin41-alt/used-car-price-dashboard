# Dashboard ราคารถยนต์มือสอง

Static dashboard สำหรับติดตามดัชนีราคารถมือสองจาก Bank of Thailand report `EC_EI_040_S2`.

## Online Dashboard

เปิดไฟล์หลัก:

```text
used_car_price_dashboard.html
```

## Source

Bank of Thailand:

```text
https://app.bot.or.th/BTWS_STAT/statistics/BOTWEBSTAT.aspx?reportID=1036&language=TH
```

คำอธิบายวิธีคำนวณและรอบเผยแพร่ของ ธปท.: https://app.bot.or.th/BTWS_STAT/statistics/DownloadFile.aspx?file=EC_EI_040_S2_TH.PDF

Coverage ใน dashboard ชุดนี้:

```text
ม.ค. 2560 - ก.ย. 2569
```

BOT ระบุฐานดัชนี:

```text
ปี 2564 = 100
```

ตรวจ CSV สดเมื่อ 6 พ.ย. 2569 ข้อมูลล่าสุดเป็น ก.ย. 2569 โดย ธปท. อัปเดตเมื่อ 31 ต.ค. 2569 เวลา 14:30. นำค่าปรับย้อนหลังจากต้นทางมาคำนวณใหม่ทั้งชุด และ Q3/2569 มีข้อมูลครบ 3 เดือนแล้ว. ตารางใหม่นี้เปลี่ยนวิธีคำนวณและปีอ้างอิงจาก `EC_EI_040`; ไม่ควรต่อสองชุดเข้าด้วยกัน. ตารางใหม่ย้อนหลังถึง ม.ค. 2560 เท่านั้น.

คำขอรอบนี้ต้องการถึง ต.ค. 2569 แต่ ธปท. ยังไม่เผยแพร่งวดดังกล่าว ณ วันที่ตรวจ จึงแสดงป้ายรอข้อมูล ต.ค. 2569 โดยคงตัวเลขจริงถึง ก.ย. 2569. คำอธิบายข้อมูลของ ธปท. ระบุความล่าช้า 1 เดือน และเผยแพร่วันทำการสุดท้ายของเดือนถัดไป.

การอัปเดตครั้งต่อไปใน workspace ต้นฉบับ (สคริปต์ `work/` อยู่เหนือโฟลเดอร์ repository นี้): รัน `python work/fetch_bot_919.py 1036` จากโฟลเดอร์หลัก แล้วรัน `python work/build_used_car_dashboard.py --through 2026-10` (ระบุงวดที่ต้องการในรูป YYYY-MM), ตรวจด้วย `python work/validate_dashboard.py` และ browser QA ก่อนคัดลอก outputs ไปยังไฟล์ชื่อเดียวกันในโฟลเดอร์นี้.

## Files

```text
index.html
used_car_price_dashboard.html
used_car_price_data.json
used_car_price_monthly.csv
used_car_price_quarterly.csv
used_car_price_yearly.csv
Dashboard ราคารถมือสอง.md
screenshots/
```

## Interaction Rules

- รายเดือน: แสดง `MoM` และ `YoY` เท่านั้น
- รายไตรมาส: แสดง `QoQ` และ `YoY` เท่านั้น
- รายปี: แสดงเฉพาะ `YoY`
- กดจุดบนกราฟแล้วรายละเอียดจะแสดงใต้กราฟนั้น

## Validation

Validated locally on 2026-11-06 (BOT source updated 2026-10-31):

```text
monthly rows: 351
quarterly rows: 117
yearly rows: 30
latest passenger Sep 2026: 86.81
latest truck Sep 2026: 79.67
```
