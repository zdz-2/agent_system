# ============================================================
# RuoYi 前端镜像（多阶段构建）
# 阶段一：Node 20 构建 Vite 静态资源
# 阶段二：Nginx 托管静态资源 + 反向代理后端
# ============================================================

# ---------- 构建阶段 ----------
FROM node:20-alpine AS builder

WORKDIR /app

# 依赖清单（lock 文件内已使用 npmmirror 源）
COPY package.json package-lock.json ./
RUN npm ci --registry=https://registry.npmmirror.com

# 拷贝源码并构建（使用 .env.production，接口前缀 /prod-api）
COPY . .
RUN npm run build:prod

# ---------- 运行阶段 ----------
FROM nginx:1.25-alpine

ENV TZ=Asia/Shanghai

# Nginx 站点配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 静态资源
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80
