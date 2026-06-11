# Dashboard ราคารถยนต์มือสอง

Static dashboard สำหรับติดตามดัชนีราคารถยนต์มือสองจาก Bank of Thailand report `EC_EI_040`.

## Online Dashboard

เปิดไฟล์หลัก:

```text
used_car_price_dashboard.html
```

## Source

Bank of Thailand:

```text
https://app.bot.or.th/BTWS_STAT/statistics/BOTWEBSTAT.aspx?reportID=919&language=TH
```

Coverage ใน dashboard ชุดนี้:

```text
ม.ค. 2554 - เม.ย. 2569
```

BOT ระบุฐานดัชนี:

```text
ปี 2558 = 100
```

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

Validated locally before publish:

```text
monthly rows: 552
quarterly rows: 186
yearly rows: 48
latest passenger Apr 2026: 86.68
latest truck Apr 2026: 63.81
```
