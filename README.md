sudo apt update
sudo apt install python3-opencv espeak -y

# ใช้ torch เวอร์ชัน CPU ที่เล็กที่สุด + ultralytics ล่าสุด
pip install torch==2.4.0+cpu torchvision==0.19.0+cpu torchaudio==2.4.0+cpu --index-url https://download.pytorch.org/whl/cpu

pip install ultralytics==8.3.8 opencv-python-headless
