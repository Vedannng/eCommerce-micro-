FROM node:20-alpine
WORKDIR /app
COPY api-gateway/package*.json ./
RUN npm install --omit=dev
COPY api-gateway/ ./
COPY public ./public
ENV NODE_ENV=production
EXPOSE 3000
CMD ["node","server.js"]
