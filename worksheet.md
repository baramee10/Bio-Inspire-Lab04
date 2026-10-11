# ใบงาน Lab04 (รายงาน 1–2 หน้า)

ชื่อกลุ่ม: มินเนี่ยนและผองเพื่อน
สมาชิก:  นางสาว	บุษกร    เสนเพ็ง      6752500029
        นางสาว  ภนิดา 	 การสร้าง     6752500169
        นาย	    บารมี    สำเภาทอง   6752500380
        นางสาว  ธนิฎฐา   วงศ์ศรีเผือก   6752500436
 วันที่: 11/10/2569

## A. ก่อนรัน (10 นาที)

วาด block diagram แล้วเขียนประโยคเชื่อมแต่ละคู่ S, P/C/B, H, R, phi/stride ให้เห็นว่า state เก็บประวัติอย่างไร

ขออนุญาติแนบเป็นภาพ Lab04_Block_Diagram.png

Stimulus (S) กระตุ้นให้เกิด Production (P) เพื่อสร้างฮอร์โมน ส่วน Clearance (C) และ Binding (B) ทำหน้าที่ลดระดับฮอร์โมน โดยคำนวณจากสถานะเดิมของระบบ จากนั้นนำ P − C − B มาคำนวณระดับฮอร์โมนใหม่ (H_new) แล้ว Receptor อ่านค่า H_new เพื่อสร้าง R_new ก่อนส่งต่อไปยัง Bounded Mapping สำหรับค่า phi และ stride ระบบเก็บค่า H_new ไว้ใช้เป็น H_old ในรอบถัดไป ทำให้การตอบสนองขึ้นอยู่กับประวัติของสิ่งกระตุ้น ไม่ใช่เฉพาะค่าปัจจุบัน

| Comparison | prediction ก่อนรัน | เหตุผลจาก term ในสมการ | ผลจริงหลังรัน |
|---|---|---|   ---|
| C2 vs C1 | C2 มีแนวโน้มทำให้ H เพิ่มเร็วกว่า C1 | C2 มี k_prod = 1.2 สูงกว่า C1 = 0.8 จึงทำให้ P สูงกว่าเมื่อ S เท่ากัน แต่ Peak H อาจเท่ากันเพราะ H ถูกจำกัดไม่เกิน 1 | |
| C3 vs C1 | C3 มีแนวโน้มลด H ช้ากว่า C1 หลังหยุด Stimulus | C3 มี k_clear = 0.25 ต่ำกว่า C1 = 0.6 จึงมีอัตรา Clearance ต่ำกว่าเมื่อ H เท่ากัน ทำให้ H คงอยู่ได้นานขึ้น | |
| C4 vs C1 | C4 มีแนวโน้มสะสม H มากกว่าและอาจฟื้นตัวช้ากว่า C1 | k_bind ของ C4 = 0 ทำให้ B = k_bind × R_old = 0 จึงไม่มีการนำ H ออกผ่าน Binding แต่ H ถูกจำกัดไม่เกิน 1 ทำให้ Peak อาจเท่ากัน | |
| C0 vs C1 | C0 ตอบสนองต่อ Stimulus โดยตรง ส่วน C1 ค่อย ๆ เปลี่ยนแปลงและมีผลจากสถานะก่อนหน้า | C0 ใช้ R = S โดยไม่มี Hormone State แต่ C1 มี H ที่เปลี่ยนตาม Production, Clearance และ Binding จึงมีการสะสมและค่อย ๆ ลดลงเมื่อหยุด Stimulus | |

เขียนว่า prediction ใดอาจเปลี่ยนเพราะ H มี upper bound=1
    Prediction ของ C2, C3 และ C4 อาจได้รับผลจาก Upper Bound = 1 เพราะแม้อัตราการสร้างเพิ่มขึ้นหรือการกำจัดฮอร์โมนลดลง แต่ H ไม่สามารถเกิน 1 ได้ ทำให้ค่า Peak H อาจเท่ากัน จึงควรเปรียบเทียบ AUC, Recovery Time และ Clamp Fraction เพิ่มเติม

## B. เติม code และคำนวณมือ (20 นาที)

1. แยก source equation กับ proposed course model คนละกรอบ
2. เติม receptor ก่อน แล้วทดสอบ H=.2, H_min=.1, H_max=.5
3. เติม step: เก็บ H_old และ R_old ก่อนคำนวณ removal
4. เมื่อ H_old=0,S=1 ให้คำนวณ P,C,B,H_raw,H_new,R_new,phi,stride แล้วทำรอบถัดไป
5. รัน checker เก็บ output หากไม่ผ่านให้บันทึก case และเหตุที่แก้

### 1. Source Equation
สมการจากเอกสารการสอนที่อ้างอิง Homchanthanakul et al. (2019)
- Hc(t) = beta × Hc(t-1) + Hg(t) - Hr(t)
- Hg(t) = alpha × f(S(t))
- Hr(t) = sum(gamma_i × Receptor_i(t))

สมการนี้อธิบายการเปลี่ยนแปลงของฮอร์โมนจากสถานะก่อนหน้า การสร้างฮอร์โมน และการนำฮอร์โมนออกผ่าน Receptor

### 2. Proposed Course Model
แบบจำลองที่รายวิชากำหนดสำหรับ Lab 4
- R_old = clamp((H_old - H_min)/(H_max - H_min), 0, 1)
- P = k_prod × S
- C = k_clear × H_old
- B = k_bind × R_old
- H_raw = H_old + dt × (P - C - B)
- H_new = clamp(H_raw, 0, 1)
- R_new = receptor(H_new)

แบบจำลองนี้ใช้ Euler Update โดยแยก Production, Clearance และ Binding ออกจากกัน และจำกัดระดับฮอร์โมนให้อยู่ระหว่าง 0 ถึง 1

ค่าพารามิเตอร์และสมการ Euler เป็นข้อกำหนดของรายวิชา ไม่ใช่ค่าที่ได้รับการยืนยันจากการทดลองกับหุ่นยนต์จริง และไม่ควรนำ beta ในสมการต้นทางมาแทน k_clear โดยตรง

| input_time_s | time_s | P | C | B | H_raw | H | R | phi |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|10.00| 10.05 | 0.8 | 0 | 0 | 0.04 | 0.04 | 0.04 | 0.208 |
|10.05| 10.10 | 0.8 | 0.024 | 0.008 | 0.0784 | 0.0784 | 0.0784 | 0.21568 |

แถว CSV ที่เลือกสอดคล้องกับ time convention อย่างไร?
     input_time_s คือเวลาเริ่มต้นของช่วงที่นำ Stimulus มาคำนวณ ส่วน time_s คือเวลาหลังการอัปเดตสถานะ โดย time_s = input_time_s + dt ดังนั้น Input ที่เวลา 10.00 วินาที จะให้สถานะใหม่ที่เวลา 10.05 วินาที

## C. Controlled comparison (20 นาที)

รัน C0–C4 ใช้ dt,duration,H0,stimulus,mapping เดียวกัน เปิด report.html และกรอกค่าจาก metrics.json ไม่อ่านค่าด้วยสายตาอย่างเดียว

| Condition | peak_H | AUC_H | latency s | rise s | recovery s | upper clamp fraction | phi TV rad |
|---|---:|---:|---:|---:|---:|---:|---:|
| C0 | N/A | N/A | 0.0500 | 0.0000 | 0.0500 | N/A | 0.4000 |
| C1 | 0.9997 | 9.9996 | 0.1500 | 2.7000 | 2.8500 | 0.0000 | 0.3998 |
| C2 | 1.0000 | 10.6724 | 0.1000 | 1.0500 | 2.8500 | 0.2900 | 0.3999 |
| C3 | 1.0000 | 11.3891 | 0.1500 | 1.4500 | 5.1000 | 0.2733 | 0.3979 |
| C4 | 1.0000 | 10.7547 | 0.1500 | 1.7000 | 3.8000 | 0.2583 | 0.3995 |

ตอบด้วยตัวเลข: C3 ฟื้นช้ากว่า C1 เท่าไร? C2/C3 peak เท่ากันหรือไม่ และ metric ใดแยกสองระบบนี้ได้? C0 มี H หรือไม่? TV ที่ต่ำกว่าเพียงอย่างเดียวพิสูจน์คุณภาพ gait ได้หรือไม่?
    C3 มี Recovery Time เท่ากับ 5.10 วินาที ขณะที่ C1 เท่ากับ 2.85 วินาที ดังนั้น C3 ฟื้นตัวช้ากว่า C1 อยู่ 2.25 วินาที เนื่องจาก C3 มีค่า k_clear ต่ำกว่า ทำให้อัตราการกำจัดฮอร์โมนลดลง และส่งผลให้ระดับฮอร์โมนและการตอบสนองของ Receptor คงอยู่นานขึ้น

    C2 และ C3 มี Peak H เท่ากันที่ 1.0000 เนื่องจากระดับฮอร์โมนถูกจำกัดด้วย Upper Bound = 1 แต่มีพฤติกรรมแตกต่างกัน โดย C3 มี AUC H = 11.3891 สูงกว่า C2 = 10.6724 และมี Recovery Time = 5.10 วินาที สูงกว่า C2 = 2.85 วินาที แสดงว่า C3 มีการสะสมฮอร์โมนตามเวลามากกว่าและฟื้นตัวช้ากว่า จึงควรใช้ AUC, Recovery Time และ Clamp Fraction ร่วมกับ Peak H ในการวิเคราะห์

    C0 ไม่มี Hormone State (H) เนื่องจากใช้การตอบสนองโดยตรงจาก Stimulus ด้วยสมการ R = S โดยไม่ผ่านกลไกสะสมฮอร์โมน ดังนั้นค่า Peak H และ AUC H จึงเป็น N/A ไม่ใช่ 0 เพราะไม่มีตัวแปร H ให้คำนวณ อย่างไรก็ตาม C0 ยังคงมีค่า Receptor (R) สำหรับการสร้าง Output

    ค่า Total Variation (TV) ที่ต่ำกว่าเพียงอย่างเดียวไม่สามารถพิสูจน์ว่าหุ่นยนต์มีคุณภาพการเดิน (Gait) ที่ดีกว่าได้ เพราะ TV วัดเพียงขนาดการเปลี่ยนแปลงสะสมของค่าควบคุมเท่านั้น จากผลทดลอง C3 มี Phi Total Variation ต่ำที่สุดประมาณ 0.3979 rad แต่ Lab 4 ยังไม่มีการจำลอง Gait จริง จึงไม่สามารถสรุปเรื่องการทรงตัว ความเร็ว หรือประสิทธิภาพการเดินได้ จำเป็นต้องทดสอบร่วมกับระบบการเดินในขั้นถัดไป

## D. Extreme tests และ handoff (10 นาที)

รัน zero,one,spike,fault ของ C1 ตรวจว่า spike มี on interval เพียงหนึ่งแถว fault มี flag และ state reset แล้วเริ่มกลับได้ แยก software bounds จาก actuator limits และ emergency stop

    ### Zero Test (C1)

    ทดลองโดยกำหนด Stimulus S = 0 ตลอด 30 วินาที

    ผลการทดลอง:
    - Peak H = 0.0
    - Peak R = 0.0
    - AUC H = 0.0
    - Phi Total Variation = 0.0 rad
    - Stride Total Variation = 0.0 m
    - Fault Count = 0
    - Timing Metrics = N/A

    วิเคราะห์ผล:
    เมื่อไม่มีสิ่งกระตุ้น ระบบไม่มีการสร้างฮอร์โมนเพิ่ม
    ทำให้ระดับ H และ R คงอยู่ที่ 0 ตลอดการทดลอง
    ค่าควบคุมจึงไม่มีการเปลี่ยนแปลง
    ผลสอดคล้องกับพฤติกรรมที่คาดการณ์จากสมการ
    ของ Proposed Course Model

    ### One Test (C1)

    ทดลองโดยกำหนด Stimulus S = 1 ตลอด 30 วินาที

    ผลการทดลอง:
    - Peak H ≈ 1.0000
    - Peak R ≈ 1.0000
    - AUC H = 28.7500
    - Upper Clamp Fraction = 0.0000
    - Phi Total Variation = 0.2000 rad
    - Stride Total Variation = 0.0100 m
    - Fault Count = 0
    - Timing Metrics = N/A

    วิเคราะห์ผล: เมื่อได้รับ Stimulus ต่อเนื่อง ระดับฮอร์โมน H เพิ่มขึ้นและค่อย ๆ เข้าใกล้ 1 ขณะที่ Clearance และ Binding เพิ่มตามระดับฮอร์โมน ทำให้อัตราการเพิ่มของ H ลดลง แม้ Peak H จะปัดเศษได้ 1.0000 แต่ Upper Clamp Fraction เป็น 0 แสดงว่าไม่มีค่า H_raw เกิน Upper Bound ในการทดลองนี้ ผลจึงสอดคล้องกับสมการของ Proposed Course Model

    ### Spike Test (C1)

    **เงื่อนไขการทดลอง:** กำหนด Stimulus S = 1 เพียงหนึ่ง Sample (0.05 วินาที) ที่เวลา 10.00 วินาที แล้วกลับเป็น S = 0

    **ผลการทดลอง**
    - Peak H = 0.0400
    - Peak R = 0.0400
    - AUC H = 0.0500
    - Upper Clamp Fraction = 0.0000
    - Fault Count = 0
    - Timing Metrics = N/A

    **การตรวจสอบข้อมูล CSV**

    พบ Stimulus = 1 เพียง 1 รอบอัปเดต จากทั้งหมด 600 รอบ โดยที่ input_time_s = 10.00 วินาที ทำให้ระดับ H เพิ่มเป็น 0.0400 ที่ time_s = 10.05 วินาที หลังจากนั้น H ลดลงเป็น 0.0384 ที่เวลา 10.10 วินาที และ 0.036864 ที่เวลา 10.15 วินาที

    **วิเคราะห์ผล**

    ระบบตอบสนองต่อสิ่งกระตุ้นระยะสั้นด้วยการเพิ่มระดับฮอร์โมน และเมื่อ Stimulus หยุดลง ระดับ H ค่อย ๆ ลดลงจาก Clearance และ Binding แทนที่จะกลับเป็นศูนย์ทันที แสดงถึงการเก็บสถานะเดิมของระบบ ผลการทดลองสอดคล้องกับ Proposed Course Model โดยไม่พบ Fault หรือ Clamp Event

    ### Fault Test (C1)

    **เงื่อนไขการทดลอง:** ใช้ Stimulus แบบ Step โดยกำหนดให้เกิด NaN จำนวน 1 Sample ที่ input_time_s = 15.00 วินาที ก่อนกลับสู่ Stimulus ปกติ

    **ผลการทดลอง**
    - Peak H = 0.9831
    - Peak R = 0.9831
    - AUC H = 8.7699
    - Fault Count = 1
    - Upper Clamp Fraction = 0.0000
    - Phi Total Variation = 0.7862 rad
    - Timing Metrics = N/A

    **การตรวจสอบข้อมูล CSV**

    ที่เวลา 15.05 วินาที พบว่า fault = 1 และระบบรีเซ็ต H, R, P, C, B และ H_raw เป็น 0 พร้อมกำหนด phi = 0.20 rad และ stride_half = 0.055 m

    เมื่อ Stimulus กลับมาปกติ ที่เวลา 15.10 วินาที พบว่า fault = 0 และ H เพิ่มขึ้นเป็น 0.0400 จากนั้นเพิ่มเป็น 0.0784 ที่เวลา 15.15 วินาที

    **วิเคราะห์ผล**

    ระบบตรวจจับค่า NaN และรีเซ็ตสถานะได้ตามข้อกำหนดของ Proposed Course Model จากนั้นสามารถกลับมาสะสมฮอร์โมนใหม่ได้เมื่อได้รับ Stimulus ปกติ โดยพบ Fault Count เพียง 1 ครั้ง และไม่พบ Clamp Event

    อย่างไรก็ตาม กลไกนี้เป็น Software Fallback ในการจำลอง ไม่ใช่หลักฐานยืนยันความปลอดภัยของฮาร์ดแวร์จริง

    ทดลองส่ง contract เดิมของ Week03 ถ้ามี แล้วตรวจ phi/stride ที่ R=0 และ R=1 ระบุสิ่งที่ Lab04 ยังไม่ได้ทดสอบ

    ผลการตรวจ Bounded Mapping

    |Receptor (R)| phi_rad | stride_half_m |
    | 0 | 0.20 rad | 0.055 m |
    | 1 | 0.40 rad | 0.065 m |

    ค่าที่ได้อยู่ภายในขอบเขตที่กำหนด แสดงว่าระบบสามารถสร้างค่าควบคุม phi และ stride ตามข้อตกลงของ Week 03 ได้

    ข้อจำกัดของ Lab 04

    Lab 04 เป็นการจำลองระบบ Artificial Hormone และการแปลงค่า Receptor ไปเป็นค่าควบคุม phi และ stride เท่านั้น ยังไม่ได้ทดสอบการทำงานร่วมกับ CPG เพื่อวัดการเคลื่อนไหวของหุ่นยนต์จริง

    การจำกัดค่าผ่าน Software Bounds และการรีเซ็ตสถานะเมื่อเกิด Fault ไม่สามารถใช้ยืนยันขีดจำกัดของ Actuator หรือความปลอดภัยของระบบ Emergency Stop ได้

    นอกจากนี้ยังไม่ได้ประเมิน Dynamic Stability, Contact Force, Friction, Motor Torque และ Energy Consumption จึงไม่สามารถสรุปว่าระบบที่มี Total Variation ต่ำที่สุดให้คุณภาพการเดินที่ดีที่สุด

    ใน Week 05 ควรนำสัญญาณจากระบบ Hormone ไปเชื่อมต่อกับ CPG แล้ววิเคราะห์ผลด้านการเดินและความมั่นคงเพิ่มเติม
     __________________________________

## E. สรุปส่งงาน (10 นาที)

เขียน 3 ย่อหน้า: (1) กลไกและ provenance (2) prediction ที่ข้อมูลสนับสนุน/ปฏิเสธพร้อมตัวเลข (3) ข้อจำกัดและการทดลอง Week05 ที่ต้องทำต่อ แนบคำสั่งรันและ config/hash ทุกข้อสรุปต้องอ้าง metric หรือ timestamp

Checklist: □ code □ comparison CSV 5 files □ extreme CSV 4 files □ config □ plots □ metrics □ report □ คำสั่งรันซ้ำ

# Lab 04 — Artificial Hormone Mechanism
**รายงานผลการจำลองระบบฮอร์โมนเทียมสำหรับการควบคุมหุ่นยนต์**

### 1. กลไกและที่มาของแบบจำลอง (Mechanism & Provenance)

        การทดลองนี้ศึกษากลไก Artificial Hormone ที่ใช้สิ่งกระตุ้น (Stimulus: S) ควบคุมระดับฮอร์โมน (H) ผ่านกระบวนการ Production (P), Clearance (C) และ Binding (B) โดยระดับฮอร์โมนใหม่คำนวณจากสมการ H_new = clamp(H_old + dt(P − C − B), 0, 1) จากนั้น Receptor แปลงระดับ H เป็นค่า R เพื่อสร้างค่าควบคุม phi และ stride_half ผ่าน Bounded Mapping ระบบเก็บสถานะฮอร์โมนจากรอบก่อนหน้า ทำให้ตอบสนองต่อสิ่งกระตุ้นอย่างต่อเนื่อง ไม่ได้เปลี่ยนทันทีเหมือน Direct Response แบบจำลองของรายวิชาอ้างอิงแนวคิดจากเอกสารการสอนที่กล่าวถึง Homchanthanakul et al. (2019) แต่สมการ Euler ค่าพารามิเตอร์และ Bounds เป็น Proposed Course Model สำหรับการศึกษา ไม่ใช่ค่าที่ได้รับการยืนยันจากหุ่นยนต์จริง

### 2. ผลการทดลองและการตรวจสอบ Prediction (Results & Analysis)

        การทดลอง Controlled Comparison พบว่า C0 ไม่มี Hormone State จึงมี Peak H และ AUC H เป็น N/A ส่วน C1 มี Peak H = 0.9997 และ AUC H = 9.9996 ขณะที่ C2 ซึ่งเพิ่ม Production มี Latency = 0.10 วินาที ต่ำกว่า C1 ที่ 0.15 วินาที จึงสนับสนุน Prediction ว่าตอบสนองได้เร็วขึ้น สำหรับ C3 ซึ่งลด Clearance มี AUC H = 11.3891 และ Recovery Time = 5.10 วินาที เทียบกับ C1 ที่ 2.85 วินาที แสดงว่าฟื้นตัวช้ากว่า 2.25 วินาที ส่วน C4 ที่ตัด Binding มี Recovery Time = 3.80 วินาที ซึ่งนานกว่า C1 เช่นกัน แม้ C2, C3 และ C4 มี Peak H เท่ากับ 1.0000 แต่ AUC และ Recovery Time แตกต่างกัน จึงไม่สามารถใช้ Peak เพียงอย่างเดียวสรุปพฤติกรรมระบบได้ ผล Extreme Tests พบว่า Zero Test มี Peak H = 0, One Test มี Peak H ใกล้ 1 โดยไม่มี Upper Clamp Event, Spike Test มี Peak H = 0.0400 จากสิ่งกระตุ้นเพียง 0.05 วินาที และ Fault Test ตรวจพบ Fault จำนวน 1 ครั้ง โดยรีเซ็ต H และ R เป็น 0 ที่เวลา 15.05 วินาที ก่อนกลับมาสะสมฮอร์โมนใหม่ที่เวลา 15.10 วินาที ผลโดยรวมสนับสนุน Prediction ที่ทำไว้ก่อนทดลอง

### 3. ข้อจำกัดและการทดลองในอนาคต (Limitations & Future Work)

        Lab 04 ตรวจสอบเฉพาะกลไกฮอร์โมน การตอบสนองของ Receptor และการสร้างค่า phi/stride_half ผ่าน Bounded Mapping เท่านั้น แม้ผล Handoff แสดงว่า Contract จาก Lab 3 และ Lab 4 มีค่าพารามิเตอร์ตรงกันและสามารถใช้งานร่วมกับตัวรันได้ แต่ยังไม่ได้ทดสอบการเชื่อมต่อ Hormone กับ CPG เพื่อจำลองการเดินแบบบูรณาการ ค่า Total Variation ที่ต่ำไม่สามารถยืนยันคุณภาพ Gait ได้ เนื่องจากยังไม่มีการประเมิน Dynamic Stability, Contact Force, Friction, Motor Torque หรือ Energy Consumption นอกจากนี้ Software Bounds และ Fault Handling ไม่สามารถใช้แทนการตรวจ Actuator Limits และ Emergency Stop ของหุ่นยนต์จริงได้ ใน Week 05 ควรเชื่อมผล Receptor เข้ากับ CPG แล้ววัดความเร็ว ความถี่การก้าว การทรงตัว และพฤติกรรมการเดินภายใต้เงื่อนไขเดียวกัน เพื่อประเมินผลของระบบฮอร์โมนต่อการเคลื่อนไหวอย่างมีหลักฐาน

        **แหล่งข้อมูลผลทดลอง:** `results/comparison/metrics.json`, `results/zero/metrics.json`, `results/one/metrics.json`, `results/spike/metrics.json`, `results/fault/metrics.json` และ CSV ที่เกี่ยวข้อง

        **เอกสารอ้างอิง:** เอกสาร Week 04 ของรายวิชา Bio Inspire; Homchanthanakul et al. (2019), DOI: 10.1109/IROS40897.2019.8968580 (ตามการอ้างอิงในคู่มือรายวิชา)

### 4. Experimental Configuration & Code Integrity

        การทดลอง Controlled Comparison ใช้ Proposed Course Model โดยกำหนด dt = 0.05 วินาที ระยะเวลาจำลอง 30 วินาที และระดับฮอร์โมนเริ่มต้น H0 = 0 ใช้ Controller Source = student และ Stimulus Profile = step สำหรับเปรียบเทียบ Conditions C0–C4

        **SHA-256 ของไฟล์ที่ใช้ในการทดลอง**

        **Controller (`student_hormone.py`):**

        `a34efb2edb9e83d43f953a109de21d7fd890757e199865f8696173b003ade2b7`

        **Runner (`lab04.py`):**

        `f66d8f29996644c2fb9cc3830298c3bd339080120e9eb657767eb7071b94e3cd`

        ตรวจสอบ SHA-256 ของไฟล์ปัจจุบันเทียบกับ `results/comparison/config.json` พบว่า Controller Match = True และ Runner Match = True แสดงว่าไฟล์โค้ดปัจจุบันตรงกับไฟล์ที่บันทึกไว้ในการทดลอง Comparison

        **หลักฐานประกอบผลทดลอง**

        - `results/comparison/metrics.json` — Metrics ของ C0–C4
        - `results/comparison/config.json` — Configuration และ SHA-256
        - `results/zero/metrics.json` — Zero Test
        - `results/one/metrics.json` — One Test
        - `results/spike/C1_spike.csv` — Spike ที่ input_time_s = 10.00 s
        - `results/fault/C1_fault.csv` — Fault ที่ time_s = 15.05 s
        - `results/handoff_from_lab03/config.json` — การทดลองโดยใช้ Contract จาก Lab 3

        ผลการตรวจ Hash ข้างต้นใช้ยืนยันไฟล์ของ Comparison โดยเฉพาะ ส่วนผลการทดลองอื่นมี Configuration แยกอยู่ในโฟลเดอร์ของตนเอง


### 5. Reproducibility Commands

        คำสั่งทั้งหมดรันด้วย Python 3 จากโฟลเดอร์ lab_04

        #### 1. ตรวจสอบโค้ดนักศึกษา

        ```powershell
        py -3 check_submission.py --student-file student_hormone.py
        ```

        #### 2. Controlled Comparison C0-C4

        ```powershell
        py -3 lab04.py --controller-source student --condition all --profile step --output-dir results/repro_comparison
        ```

        #### 3. Zero Test

        ```powershell
        py -3 lab04.py --controller-source student --condition C1 --profile zero --output-dir results/repro_zero
        ```

        #### 4. One Test

        ```powershell
        py -3 lab04.py --controller-source student --condition C1 --profile one --output-dir results/repro_one
        ```

        #### 5. Spike Test

        ```powershell
        py -3 lab04.py --controller-source student --condition C1 --profile spike --output-dir results/repro_spike
        ```

        #### 6. Fault Test

        ```powershell
        py -3 lab04.py --controller-source student --condition C1 --profile fault --output-dir results/repro_fault
        ```

        #### 7. Week 03 Handoff

        ```powershell
        py -3 lab04.py --controller-source student --condition C1 --profile zero --contract ..\lab_03\results\week4_bridge\handoff_contract.json --output-dir results/repro_handoff
        ```

        หมายเหตุ:
        - ใช้ Python 3.10 ขึ้นไป
        - ใช้โค้ดและ Contract ตาม SHA-256/Configuration ที่บันทึกไว้
        - ผลลัพธ์ถูกสร้างในโฟลเดอร์ results/repro_*
        - หากโฟลเดอร์ปลายทางมีไฟล์อยู่แล้ว ให้เลือกชื่อใหม่เพื่อไม่เขียนทับข้อมูล
        - ผลจาก Lab 04 เป็นการจำลอง subsystem ไม่ใช่การทดสอบ Gait หรือ Hardware Safety จริง
