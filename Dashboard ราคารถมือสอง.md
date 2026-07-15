# Dashboard ราคารถมือสอง

คู่มือฉบับสมบูรณ์สำหรับสร้าง dashboard ดัชนีราคารถยนต์มือสองจาก Bank of Thailand ให้ได้หน้าตาและพฤติกรรมเหมือน dashboard ต้นฉบับ

## 1. เป้าหมายของ dashboard

สร้าง dashboard แบบ interactive สำหรับติดตามดัชนีราคารถยนต์มือสองของไทย โดยแยกเป็น

- รถยนต์นั่งมือสอง
- รถยนต์บรรทุกมือสอง
- ดัชนีรวม เพื่อใช้ประกอบภาพรวมล่าสุดและตาราง

Dashboard ต้องตอบโจทย์นักลงทุน คือดูแนวโน้มราคาในอดีต เปรียบเทียบฤดูกาลในแต่ละปี และดู momentum ของราคามือสองผ่าน MoM, QoQ, YoY โดยไม่ผสม metric ผิด grain

## 2. แหล่งข้อมูลหลัก

ใช้ข้อมูลจาก Bank of Thailand report:

```text
https://app.bot.or.th/BTWS_STAT/statistics/BOTWEBSTAT.aspx?reportID=919&language=TH
```

ชื่อรายงาน:

```text
EC_EI_040 ดัชนีราคารถยนต์มือสอง 1/
```

แหล่งข้อมูลที่ BOT ระบุ:

```text
บริษัท สหการประมูล จำกัด (มหาชน) และธนาคารแห่งประเทศไทย
```

หมายเหตุที่ต้องใส่ใน dashboard:

```text
ตัวเลขดัชนีทั้งหมดได้ถูกปรับฐานในการคำนวณให้ปี 2558 = 100
```

ข้อจำกัดสำคัญของ source:

```text
BOT report ระบุช่วงข้อมูลที่มีจริงเป็น ม.ค. 2554 - พ.ค. 2569
จึงไม่ควรเติมข้อมูลปี 2550-2553 เอง หากไม่มี source อื่นรองรับ
```

## 3. Output ที่ต้องสร้าง

ให้สร้างไฟล์เหล่านี้ในโฟลเดอร์ `outputs/`

```text
used_car_price_dashboard.html
used_car_price_data.json
used_car_price_monthly.csv
used_car_price_quarterly.csv
used_car_price_yearly.csv
used_car_price_dashboard_desktop.png
used_car_price_dashboard_mobile.png
used_car_price_dashboard_clicked.png
```

ไฟล์ HTML ต้องเป็น dashboard ที่เปิดได้เองแบบ static โดย embed dataset ไว้ในไฟล์ หรือโหลด dataset local static ได้ โดยไม่มี dependency server-side

## 4. วิธีดึงข้อมูลจาก BOT

หน้า BOT เป็น ASP.NET WebForms จึงต้องดึงข้อมูลอย่างระมัดระวัง

ขั้นตอนที่ใช้ใน dashboard ต้นฉบับ:

1. เปิดหน้า report 919 ด้วย `GET`
2. อ่าน hidden fields:
   - `__VIEWSTATE`
   - `__VIEWSTATEGENERATOR`
   - `__EVENTVALIDATION`
3. ส่ง `POST` กลับไปหน้าเดิม โดยตั้งค่า:

```text
drpPeriod=MTH
drpFromMonth=xxxx01xx
drpFromYear=2011xxxx
drpToMonth=xxxx04xx
drpToYear=2026xxxx
btnSubmit=Submit
```

4. หลัง submit จะได้ตารางครบทุกเดือน มี 186 คอลัมน์ใน HTML table `dgExcel`
5. ใช้ postback ของปุ่ม CSV export:

```text
imbExportText.x=12
imbExportText.y=12
```

6. ไฟล์ที่ได้จาก BOT เป็น CSV ชื่อ `EC_EI_040.csv` encoding UTF-8 with BOM

ในการ run จริง ณ dashboard ต้นฉบับ:

```text
ปรับปรุงล่าสุด : 30 มิ.ย. 2569 14:30
วันที่เรียกข้อมูล : 15 ก.ค. 2569 23:06
ช่วงข้อมูล: ม.ค. 2554 - พ.ค. 2569
```

## 5. โครงสร้างข้อมูลต้นทาง

CSV จาก BOT มี rows สำคัญ 3 series:

```text
ดัชนีราคารถยนต์มือสอง
ดัชนีราคารถยนต์นั่งมือสอง
ดัชนีราคารถยนต์บรรทุกมือสอง
```

Map series เป็น category:

```text
ดัชนีราคารถยนต์มือสอง -> overall -> ดัชนีรวม
ดัชนีราคารถยนต์นั่งมือสอง -> passenger -> รถยนต์นั่งมือสอง
ดัชนีราคารถยนต์บรรทุกมือสอง -> truck -> รถยนต์บรรทุกมือสอง
```

Header เดือนใน CSV เป็นภาษาไทยและปี พ.ศ. เช่น:

```text
พ.ค. 2569 p
เม.ย. 2569
มี.ค. 2569
ก.พ. 2569
ม.ค. 2569
...
ม.ค. 2554
```

ต้องแปลงปี พ.ศ. เป็น ค.ศ. สำหรับ key ภายใน:

```text
year_ce = year_be - 543
```

และเก็บทั้งสองแบบ:

```text
year_be = 2569
year_ce = 2026
date = 2026-05
month = 5
period_label = พ.ค. 2569
period_display = พ.ค. 2569
provisional = true ถ้า header มี p
```

## 6. การสร้างข้อมูลรายเดือน

รายเดือนคือค่าดัชนีตรงจาก BOT ไม่ต้อง aggregate

จำนวน rows ที่ถูกต้องใน dashboard ต้นฉบับ:

```text
185 เดือน x 3 series = 555 rows
```

ค่าสุดท้ายที่ต้อง reconcile ได้:

```text
รถยนต์นั่งมือสอง พ.ค. 2569 = 87.78
รถยนต์บรรทุกมือสอง พ.ค. 2569 = 66.81
ดัชนีรวม พ.ค. 2569 = 75.15
```

## 7. การสร้างข้อมูลรายไตรมาส

รายไตรมาสให้คำนวณจากค่าเฉลี่ยเลขคณิตของดัชนีรายเดือนในไตรมาสนั้น

สูตร:

```text
quarter_value = average(monthly_index_in_quarter)
```

ตัวอย่าง:

```text
Q1 = average(Jan, Feb, Mar)
Q2 = average(Apr, May, Jun)
Q3 = average(Jul, Aug, Sep)
Q4 = average(Oct, Nov, Dec)
```

ถ้าไตรมาสยังไม่ครบ 3 เดือน ต้องใส่ flag:

```text
complete = false
month_count = จำนวนเดือนที่มีข้อมูล
status = partial/YTD
```

ตัวอย่างใน dashboard ต้นฉบับ:

```text
Q2 2569 มีข้อมูล เม.ย. และ พ.ค. 2569 เพราะข้อมูลล่าสุดถึง พ.ค. 2569
ดังนั้น Q2 2569 = partial/YTD
```

จำนวน rows รายไตรมาสที่ถูกต้อง:

```text
62 quarters x 3 series = 186 rows
```

## 8. การสร้างข้อมูลรายปี

รายปีให้คำนวณจากค่าเฉลี่ยเลขคณิตของดัชนีรายเดือนในปีนั้น

สูตร:

```text
year_value = average(monthly_index_in_year)
```

ถ้าปียังไม่ครบ 12 เดือน ต้องใส่ flag:

```text
complete = false
month_count = จำนวนเดือนที่มีข้อมูล
status = partial/YTD
```

ตัวอย่างใน dashboard ต้นฉบับ:

```text
ปี 2569 เป็น YTD ม.ค.-พ.ค.
รถยนต์นั่งมือสอง ปี 2569 YTD = 94.01
```

จำนวน rows รายปีที่ถูกต้อง:

```text
16 years x 3 series = 48 rows
```

## 9. สูตร MoM, QoQ, YoY

ใช้สูตร percent change:

```text
change_pct = (current_value / previous_value - 1) * 100
```

กฎสำคัญ ห้ามผสม metric ผิด grain:

```text
รายเดือน: แสดง MoM และ YoY เท่านั้น
รายไตรมาส: แสดง QoQ และ YoY เท่านั้น
รายปี: แสดง YoY เท่านั้น
```

### รายเดือน

MoM:

```text
current month เทียบ previous month
ม.ค. เทียบ ธ.ค. ปีก่อน
```

YoY:

```text
current month เทียบเดือนเดียวกันของปีก่อน
```

ห้ามแสดง QoQ ในรายเดือน

### รายไตรมาส

QoQ:

```text
current quarter เทียบ previous quarter
Q1 เทียบ Q4 ปีก่อน
```

YoY:

```text
current quarter เทียบ quarter เดียวกันของปีก่อน
```

ห้ามแสดง MoM ในรายไตรมาส

### รายปี

YoY:

```text
current year average เทียบ previous year average
```

ห้ามแสดง MoM หรือ QoQ ในรายปี

## 10. Layout dashboard

โครงสร้างหน้า:

1. Header
   - ชื่อ dashboard: `Dashboard ราคารถยนต์มือสอง`
   - subtitle: `ดัชนีราคารถยนต์นั่งมือสองและรถยนต์บรรทุกมือสอง จาก BOT EC_EI_040`
   - badge coverage: `ม.ค. 2554 - พ.ค. 2569`
   - badge latest: `พ.ค. 2569`
   - badge base index: `ปี 2558 = 100`

2. KPI cards 4 ใบ
   - รถยนต์นั่งล่าสุด
   - รถบรรทุกล่าสุด
   - ส่วนต่าง นั่ง - บรรทุก
   - ค่าเฉลี่ยปีล่าสุด

3. Filter panel
   - Frequency segmented buttons:
     - รายเดือน
     - รายไตรมาส
     - รายปี
   - Year filter:
     - ล่าสุด 5 ปี
     - เลือกทั้งหมด
     - ล้าง
     - checkbox ปี 2554-2569
   - Table category:
     - รถยนต์นั่งมือสอง
     - รถยนต์บรรทุกมือสอง
     - ดัชนีรวม

4. Chart section
   - กราฟรถยนต์นั่งมือสอง
   - กราฟรถยนต์บรรทุกมือสอง

5. Detail section
   - ภาพรวมล่าสุด
   - ตารางข้อมูลตาม filter

6. Source and methodology
   - source link
   - source update date
   - method รายเดือน/ไตรมาส/ปี
   - coverage caveat

## 11. พฤติกรรมของกราฟ

กราฟต้องมี 2 กราฟ:

```text
รถยนต์นั่งมือสอง
รถยนต์บรรทุกมือสอง
```

ในรายเดือนและรายไตรมาส:

```text
1 เส้น = 1 ปี
แต่ละปีใช้สีต่างกัน
กด filter เลือกปีที่จะเทียบได้
```

ในรายปี:

```text
แสดงค่าเฉลี่ยรายปี/YTD ของปีที่เลือก
```

จุดในกราฟต้อง interact ได้ 2 แบบ:

1. เอาเมาส์วางบนจุด
   - แสดง tooltip ชั่วคราว

2. กดจุดบนกราฟ
   - แสดง panel ถาวรใต้กราฟนั้น
   - ต้องแยกกันระหว่างกราฟรถยนต์นั่งมือสองและกราฟรถยนต์บรรทุกมือสอง

Panel ใต้กราฟต้องแสดง:

```text
ข้อมูลที่เลือก: รถยนต์นั่งมือสอง หรือ รถยนต์บรรทุกมือสอง
ความถี่: รายเดือน / รายไตรมาส / รายปี
ช่วงเวลา
ดัชนี
MoM หรือ QoQ ตาม grain
YoY
สถานะ: ครบ / p / partial/YTD
```

กฎ metric ใน panel ใต้กราฟ:

```text
รายเดือน: ดัชนี + MoM + YoY
รายไตรมาส: ดัชนี + QoQ + YoY
รายปี: ดัชนี + YoY เท่านั้น
```

ข้อความ default ก่อนกดจุด:

```text
กดจุดบนกราฟเพื่อแสดงข้อมูลช่วงเวลานั้นใต้กราฟ
```

## 12. ภาพรวมล่าสุด

ส่วนภาพรวมล่าสุดต้องเปลี่ยนตาม frequency filter ไม่ใช่ fixed monthly เท่านั้น

ถ้าเลือก `รายเดือน`:

```text
แสดงเดือนล่าสุด
แสดง MoM และ YoY
ไม่แสดง QoQ
```

ถ้าเลือก `รายไตรมาส`:

```text
แสดงไตรมาสล่าสุด
แสดง QoQ และ YoY
ไม่แสดง MoM
```

ถ้าเลือก `รายปี`:

```text
แสดงปีล่าสุด/YTD
แสดงเฉพาะ YoY
ไม่แสดง MoM หรือ QoQ
```

ข้อความ note:

```text
คอลัมน์ขวาสุดแสดง MoM และ YoY ตามรายเดือน; รายเดือนไม่แสดง QoQ, รายไตรมาสไม่แสดง MoM, รายปีแสดงเฉพาะ YoY
```

ให้ปรับข้อความตาม frequency ที่เลือก เช่น `QoQ และ YoY ตามรายไตรมาส`

## 13. ตารางข้อมูลตาม filter

ตารางต้องเปลี่ยนตาม frequency และ category

หัวตารางรายเดือน:

```text
เดือน | ความถี่ | ปี | ดัชนี | MoM | YoY | สถานะ
```

หัวตารางรายไตรมาส:

```text
ไตรมาส | ความถี่ | ปี | ดัชนี | QoQ | YoY | สถานะ
```

หัวตารางรายปี:

```text
ปี | ความถี่ | ดัชนี | YoY | สถานะ
```

ห้ามใช้หัวคอลัมน์รวมแบบ:

```text
MoM/QoQ/YoY
```

เพราะทำให้ผู้อ่านไม่รู้ว่าเป็น metric ใด

Table note ด้านขวาบนควรแสดง:

```text
รถยนต์นั่งมือสอง | รายเดือน | 52 rows
```

หรือเปลี่ยนตาม filter จริง

## 14. สีและการออกแบบ

แนวทาง visual:

- ใช้พื้นหลังอ่อน อ่านง่าย
- ใช้ card radius ประมาณ 8px
- ไม่ใช้ landing page หรือ hero ใหญ่
- หน้าแรกต้องเป็น dashboard ใช้งานจริงทันที
- สีปีต้องแตกต่างกัน
- ใช้สีเขียวสำหรับค่าบวก และสีแดงสำหรับค่าลบ
- อย่าให้ text ล้นกรอบบน mobile
- mobile ต้อง stack controls และ cards ลงมาเป็น 1 column

ชุดสีปีที่ใช้ใน dashboard ต้นฉบับ:

```text
2554 #2563eb
2555 #dc2626
2556 #059669
2557 #d97706
2558 #7c3aed
2559 #0891b2
2560 #be123c
2561 #4d7c0f
2562 #9333ea
2563 #ea580c
2564 #0f766e
2565 #4338ca
2566 #b45309
2567 #0284c7
2568 #c026d3
2569 #16a34a
```

## 15. Validation checklist

หลังสร้าง dashboard ต้องตรวจทุกข้อ:

### Data validation

```text
coverage_start = 2011-01
coverage_end = 2026-05
monthly rows = 555
quarterly rows = 186
yearly rows = 48
```

ค่าสุดท้าย:

```text
passenger May 2026 = 87.78
truck May 2026 = 66.81
overall May 2026 = 75.15
```

YTD flag:

```text
ปี 2569 complete = false
ปี 2569 month_count = 5
Q2 2569 complete = false
Q2 2569 month_count = 2
```

### UI validation

```text
รายเดือนมี MoM และ YoY ไม่แสดง QoQ
รายไตรมาสมี QoQ และ YoY ไม่แสดง MoM
รายปีมี YoY เท่านั้น
ไม่มีหัวตาราง MoM/QoQ/YoY
ตารางมีคอลัมน์ความถี่
กดจุดในกราฟแล้วข้อมูลแสดงใต้กราฟ
tooltip ยังทำงานเมื่อ hover
desktop layout ไม่ล้น
mobile layout ไม่ล้น
```

### Browser validation

ควรถ่าย screenshot อย่างน้อย:

```text
used_car_price_dashboard_desktop.png
used_car_price_dashboard_mobile.png
used_car_price_dashboard_clicked.png
```

และตรวจว่ากราฟไม่ blank

## 16. ขั้นตอนการ publish GitHub Pages

ให้สร้างโฟลเดอร์ static site เช่น:

```text
github-site/
```

ใส่ไฟล์:

```text
index.html
used_car_price_dashboard.html
used_car_price_data.json
used_car_price_monthly.csv
used_car_price_quarterly.csv
used_car_price_yearly.csv
Dashboard ราคารถมือสอง.md
README.md
screenshots/
```

ให้ `index.html` redirect หรือ link ไปที่ `used_car_price_dashboard.html`

ตัวอย่าง repo name:

```text
used-car-price-dashboard
```

หลัง push แล้ว enable GitHub Pages:

```text
branch = main
path = /
```

ถ้าใช้ GitHub API แล้วเจอ HTTP 422 ตอน enable Pages ให้เช็ค payload ว่าต้องส่ง nested source:

```text
source[branch]=main
source[path]=/
```

## 17. Prompt สำหรับส่งให้ Codex ทำซ้ำ

คัดลอก prompt นี้ให้ Codex:

```text
ช่วยสร้าง Dashboard ราคารถมือสองย้อนหลังจาก BOT report EC_EI_040

แหล่งข้อมูล:
https://app.bot.or.th/BTWS_STAT/statistics/BOTWEBSTAT.aspx?reportID=919&language=TH

เงื่อนไข:
1. ดึงข้อมูลรายเดือนจาก BOT ให้ครบช่วงที่ BOT มีจริง
2. ถ้า BOT มีข้อมูลเริ่ม ม.ค. 2554 ไม่ต้องเติมปี 2550-2553 เอง ให้เขียน caveat ชัดเจน
3. สร้างข้อมูลรายเดือน / รายไตรมาส / รายปี
4. รายไตรมาส = ค่าเฉลี่ยรายเดือนในไตรมาส
5. รายปี = ค่าเฉลี่ยรายเดือนในปีนั้น ปีล่าสุดที่ไม่ครบ 12 เดือนให้ถือเป็น YTD
6. แยกกราฟรถยนต์นั่งมือสองและรถยนต์บรรทุกมือสอง
7. กราฟรายเดือนและรายไตรมาสให้ 1 เส้น = 1 ปี และแต่ละปีใช้สีต่างกัน
8. มี filter เลือกความถี่ รายเดือน / รายไตรมาส / รายปี
9. มี filter เลือกปีที่ต้องการเปรียบเทียบ
10. จุดในกราฟต้อง hover แล้วเห็น tooltip และ click แล้วแสดงข้อมูลใต้กราฟนั้น
11. ข้อมูลใต้กราฟต้องแสดงความถี่ ช่วงเวลา ดัชนี MoM/QoQ/YoY ตาม grain และสถานะ
12. รายเดือนแสดง MoM และ YoY เท่านั้น ห้ามแสดง QoQ
13. รายไตรมาสแสดง QoQ และ YoY เท่านั้น ห้ามแสดง MoM
14. รายปีแสดง YoY เท่านั้น ห้ามแสดง MoM หรือ QoQ
15. ตารางข้อมูลตาม filter ต้องมีคอลัมน์ความถี่ และหัวคอลัมน์ metric ต้องชัดเจน
16. ห้ามใช้หัวคอลัมน์ MoM/QoQ/YoY รวมกัน
17. สร้างไฟล์ HTML static เปิดได้ทันที พร้อม JSON/CSV data extracts
18. ตรวจด้วย browser screenshot desktop/mobile และ click validation

Output:
- used_car_price_dashboard.html
- used_car_price_data.json
- used_car_price_monthly.csv
- used_car_price_quarterly.csv
- used_car_price_yearly.csv
- screenshot desktop/mobile/clicked
```

## 18. ข้อผิดพลาดที่ต้องระวัง

1. อย่า assume ว่ามีข้อมูลปี 2550 ถ้า BOT report ไม่มี
2. อย่า scrape แค่ตารางหน้าแรก เพราะหน้าแรกอาจโชว์แค่ 6 เดือนล่าสุด
3. ต้อง submit form ให้ช่วงครบก่อน หรือใช้ CSV export หลัง submit
4. อย่าใช้ MoM/QoQ/YoY รวมในคอลัมน์เดียวโดยไม่ระบุว่าเป็น metric อะไร
5. อย่าแสดง QoQ ในรายเดือน
6. อย่าแสดง MoM ในรายไตรมาส
7. อย่าแสดง MoM/QoQ ในรายปี
8. ต้อง click จุดกราฟแล้วแสดงรายละเอียดใต้กราฟ ไม่ใช่แค่ hover tooltip
9. ต้อง validate ด้วย screenshot เพราะ canvas อาจมีแกนแต่ไม่มีเส้นถ้า data model ขาด `period_index`
10. ถ้า PowerShell แสดงภาษาไทยเพี้ยน ให้ verify file ด้วย UTF-8 read ไม่ใช่ดูจาก console อย่างเดียว

## 19. สถานะ dashboard ต้นฉบับ

Dashboard ต้นฉบับสร้างและ validate แล้วด้วยข้อมูล:

```text
coverage: ม.ค. 2554 - พ.ค. 2569
monthly rows: 555
quarterly rows: 186
yearly rows: 48
latest passenger: พ.ค. 2569 = 87.78
latest truck: พ.ค. 2569 = 66.81
```

Click validation ผ่าน:

```text
รายเดือน: กดจุดแล้วแสดง MoM + YoY
รายไตรมาส: กดจุดแล้วแสดง QoQ + YoY
รายปี: กดจุดแล้วแสดงเฉพาะ YoY
```

