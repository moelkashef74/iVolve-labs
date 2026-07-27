FROM node:18-alpine AS builder
WORKDIR /app
COPY CI-CD/lab22/kubernetes-app/package.json ./
RUN npm install

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY CI-CD/lab22/kubernetes-app/. .
EXPOSE 3000
CMD ["node", "server.js"]


