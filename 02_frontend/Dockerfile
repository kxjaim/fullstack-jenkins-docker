# ---------- Stage 1: Build the Next.js application ----------
FROM node:20-alpine AS builder
ARG API_HOST=http://localhost:3001
WORKDIR /app

# ⬇️ เพิ่มบรรทัดนี้ (ลด memory usage)
ENV NODE_OPTIONS="--max-old-space-size=1024"

COPY package*.json ./

# ⬇️ เพิ่ม --maxsockets=1 (ลด concurrent downloads)
RUN npm ci --maxsockets=1 --legacy-peer-deps || \
    npm install --legacy-peer-deps && \
    npm cache clean --force
    
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1 \
    API_HOST=${API_HOST}
RUN npm run build

# ---------- Stage 2: Production runtime ----------
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production \
    NEXT_TELEMETRY_DISABLED=1
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]