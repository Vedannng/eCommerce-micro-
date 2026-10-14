FROM node:20-alpine
WORKDIR /app
COPY product-service/package*.json ./
RUN npm install --omit=dev
COPY product-service/ ./
ENV NODE_ENV=production
EXPOSE 4001
CMD ["node","server.js"]
