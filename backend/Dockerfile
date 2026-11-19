# Build stage
FROM node:20-slim AS builder

WORKDIR /app

# Install OpenSSL & certificates for Prisma query engine
RUN apt-get update -y && apt-get install -y openssl ca-certificates && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
COPY prisma ./prisma/

RUN npm install

COPY . .

# Generate Prisma client for all target platforms
RUN npx prisma generate
RUN npm run build

# Production runtime stage
FROM node:20-slim AS runner

WORKDIR /app

ENV NODE_ENV=production

# Install OpenSSL for runtime schema engine
RUN apt-get update -y && apt-get install -y openssl ca-certificates && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

EXPOSE 5001

CMD ["sh", "-c", "npx prisma db push && node dist/server.js"]
