# ใช้ python image เป็น base image
FROM python:3.10

# ตั้งค่า working directory
WORKDIR /app

# คัดลอกไฟล์ requirements.txt
COPY requirements.txt .

# ติดตั้ง dependencies
RUN pip install --no-cache-dir -r requirements.txt

# คัดลอกโค้ด backend ทั้งหมด
COPY . .

# เปิดพอร์ตที่ FastAPI ใช้งาน
EXPOSE 8000

# รัน FastAPI
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
