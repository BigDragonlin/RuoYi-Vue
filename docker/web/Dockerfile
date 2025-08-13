# ---------- Stage 1: Build frontend with Node 16 ----------
FROM node:16-alpine AS builder

WORKDIR /app

# 可通过 --build-arg 覆盖，默认 /prod-api
ARG VUE_APP_BASE_API=/prod-api
ARG NPM_REGISTRY=

ENV VUE_APP_BASE_API=${VUE_APP_BASE_API}

# 安装依赖（使用国内源可选）
COPY ruoyi-ui/package*.json ./
RUN if [ -n "$NPM_REGISTRY" ]; then npm config set registry "$NPM_REGISTRY"; fi \
    && npm ci

# 拷贝前端源码并构建
COPY ruoyi-ui/ ./
RUN npm run build:prod


# ---------- Stage 2: Serve with Nginx ----------
FROM nginx:1.25-alpine

ENV TZ=Asia/Shanghai

# 站点配置与静态资源
COPY docker/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80


