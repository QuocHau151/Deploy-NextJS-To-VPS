# Deploy NextJs to VPS with Docker and CI/CD

## 1. Tạo Dockerfile và .dockerignore

### Dockerfile

```bash
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN \
  if [ -f yarn.lock ]; then yarn run build; \
  elif [ -f package-lock.json ]; then npm run build; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm run build; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

# ĐƯA TẤT CẢ ENV ENVIRONMENT VÀO THEO KIỂU `ENV NODE_ENV production`

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/next-config-js/output
CMD HOSTNAME="0.0.0.0" node server.js

```

### .dockerignore

```bash
Dockerfile
.dockerignore
node_modules
npm-debug.log
README.md
.next
.git
```

## 2. Tạo access token SSH

- Tạo Personal access tokens (classic) trên github.com
- Copy và nhớ access token để sau còn đăng nhập

## 3. Tạo Docker Build

```bash
 docker build -t ghcr.io/{USERNAME}/{NAME_PROJECT}:latest .
```

- Nếu không có lỗi thì xoá Docker Image đó

## 4. Đăng nhập docker github container registry (ghcr.io) trên máy

```bash
docker login ghcr.io -u {USERNAME} -p {PASS_IS_ACCESS_TOKEN}
```

## 5. Push Docker Image

```bash
docker push ghcr.io/{USERNAME}/{NAME_PROJECT}:latest
```

- push project lên package của github

## 6. Mở Terminal và login VPS

```bash
ssh root@id_vps
```

## 7. Cài Docker cho VPS

- Tại root@ip_vps cài đặt docker

```bash
sudo apt-get update
```

```bash
sudo apt install docker.io
```

```bash
docker -v
```

- Nếu docker not found thì:

```bash
systemctl is-active docker
```

- = Active là ok - Nếu = Inactive thì:

```bash
apt-get install docker.io
```

```bash
apt-get update
```

- Thử lại

```bash
systemctl is-active docker
```

```bash
systemctl enable docker
```

```bash
systemctl start docker
```

- Verify docker

```bash
sudo docker run hello-world
```

- Logic Docker

```bash
docker login ghcr.io -u {USERNAME} -p {PASS_IS_ACCESS_TOKEN}
```
- Docker pull

  ``` bash
  docker pull ghcr.io/quochau151/fptsmarthome:latest
  ```
- Chạy Docker trên VPS

```bash
docker run -d -p 3000:3000 ghcr.io/{USERNAME}/{NAME_PROJECT}:latest
```

- Kiểm tra container id, volume image, status

```bash
docker ps
```

- Kiểm tra code đã oke chưa bằng cách search địa chỉ ip

```bash
 ip_vps:3000
```

## 8. Trỏ Domain về Vps
## 9. Cài Đặt NginX và PM2 cho VPS
