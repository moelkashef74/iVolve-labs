FROM node:18-alpine AS builder
WORKDIR /app
COPY kubernetes-app/package.json ./
RUN npm install

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY kubernetes-app/. .
EXPOSE 3000
CMD ["node", "server.js"]


