FROM node:20-alpine
WORKDIR /app
COPY order-service/package*.json ./
RUN npm install --omit=dev
COPY order-service/ ./
ENV NODE_ENV=production
EXPOSE 4002
CMD ["node","server.js"]
