# ── Stage duy nhất: dùng node 18 alpine trên linux/amd64 ──
# Chỉ định platform rõ ràng vì M4 Pro là ARM,
# nhưng server Linux thường là amd64
FROM --platform=linux/arm64 node:18-alpine

# Cài build tools cần thiết cho bcrypt (package native C++)
# bcrypt cần python3 + make + g++ để compile từ source
RUN apk add --no-cache python3 make g++

# Tạo và chuyển vào thư mục làm việc trong container
WORKDIR /app

# Copy package.json trước (tận dụng Docker layer cache)
# Nếu code thay đổi nhưng package.json không đổi,
# Docker bỏ qua bước npm install → build nhanh hơn
COPY package*.json ./

# Cài dependencies (chỉ production, không cần devDependencies)
RUN npm install --omit=dev

# Copy toàn bộ source code vào container
COPY . .

# Khai báo port backend lắng nghe (để Docker biết, không tự mở)
EXPOSE 8386

# Lệnh khởi động container
CMD ["node", "server.js"]
