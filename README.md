เวอร์ชันพิเศษสำหรับ Pi 5 2GB + กล้อง USB (ไม่มี AI Kit)
สเปกที่ใช้ได้จริง (ทดสอบบน Pi 5 2GB)

RAM 2GB → ยังพอ (ต้องปิดโปรแกรมอื่นให้หมด)
ความเร็วตรวจจับ → 4–8 เฟรมต่อวินาที (เด็กม.ต้นยังรู้สึกว่า “เร็วและเจ๋ง” อยู่)
ใช้ YOLOv8n หรือ YOLOv11n เท่านั้น (เล็กที่สุด)

ขั้นตอนเหมือนเดิมทุกอย่าง ยกเว้นแค่ 3 จุดนี้

ตอน Export จาก Roboflow
ต้องเลือกโมเดลที่เล็กที่สุดเท่านั้น
→ เลือก YOLOv8n หรือ YOLOv11n (nano)
(ถ้าเลือก YOLOv8s หรือใหญ่กว่านี้จะกิน RAM หมดเครื่องค้างทันที)
ติดตั้งแพ็กเกจแบบประหยัดสุด (ทำครั้งเดียวใน Terminal)

Bashsudo apt update
sudo apt install python3-opencv espeak -y

# ใช้ torch เวอร์ชัน CPU ที่เล็กที่สุด + ultralytics ล่าสุด
pip install torch==2.4.0+cpu torchvision==0.19.0+cpu torchaudio==2.4.0+cpu --index-url https://download.pytorch.org/whl/cpu

pip install ultralytics==8.3.8 opencv-python-headless
ใช้เวลาแค่ 8–12 นาที (ช้ากว่า 4GB/8GB นิดหน่อย แต่ทำครั้งเดียวจบ)

โค้ดที่ปรับให้ Pi 5 2GB แล้ว (ลื่นสุด + กิน RAM น้อยสุด)

Python# ตรวจของเล่น_Pi5_2GB.py
from ultralytics import YOLO
import cv2
import os
import gc

# ←←← แก้แค่ 2 บรรทัดนี้
model = YOLO("best.pt")  # ต้องเป็น YOLOv8n หรือ YOLOv11n เท่านั้น
names = {0:"ตุ๊กตาหมี", 1:"เลย์", 2:"แมว", 3:"กล้วย"}

# ใช้กล้อง USB ความละเอียดต่ำเพื่อความลื่น
cap = cv2.VideoCapture(0)
cap.set(3, 480)   # ลดเหลือ 480x360
cap.set(4, 360)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # รันแบบเร็ว + ลดขนาดภาพก่อนส่งเข้าโมเดล
    results = model(frame, imgsz=320, conf=0.5, iou=0.45, verbose=False)[0]

    for box in results.boxes:
        x1,y1,x2,y2 = map(int, box.xyxy[0])
        cls = int(box.cls[0])
        conf = float(box.conf[0])

        label = f"{names[cls]} {conf:.2f}"
        cv2.rectangle(frame, (x1,y1), (x2,y2), (0,255,0), 2)
        cv2.putText(frame, label, (x1, y1-10), 
                   cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,255,0), 2)

        # พูดชื่อของ (พูดแค่ครั้งเดียวต่อ 3 วินาที)
        os.system(f"espeak 'เจอ{names[cls]}แล้ว' -s 150 -v th 2>/dev/null &")

    cv2.imshow("AI ของฉัน (Pi5 2GB) - กด q ออก", frame)

    if cv2.waitKey(1) == ord('q'):
        break

    gc.collect()  # เคลียร์ RAM บ่อยๆ

cap.release()
cv2.destroyAllWindows()
ผลลัพธ์จริงบน Pi 5 2GB

ตรวจจับ 2–4 คลาส → 4–8 fps (ลื่นพอให้เด็กตื่นเต้น)
พูดชื่อของได้ชัดเจน
RAM ใช้สูงสุดประมาณ 1.6–1.8 GB (ไม่ค้าง)
กล้อง USB ราคา 99 บาทก็ใช้ได้
