# ใช้ node image เป็น base image
FROM node:18 AS build

# ตั้งค่า working directory
WORKDIR /app

# คัดลอกไฟล์ package.json และ package-lock.json
COPY package*.json ./

# ติดตั้ง dependencies
RUN npm install

# คัดลอกโค้ด frontend ทั้งหมด
COPY . .

# สร้าง production build
RUN npm run build

# ใช้ nginx เพื่อ serve build
FROM nginx:alpine

# คัดลอก build จากขั้นตอนก่อนหน้า
COPY --from=build /app/dist /usr/share/nginx/html

# เปิดพอร์ต 80
EXPOSE 80

# รัน nginx
CMD ["nginx", "-g", "daemon off;"]
