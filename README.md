# E-commerce Microservices — no Docker Compose

Services: API Gateway (port 3001 on host), Product Service, Order Service, separate MySQL containers and named volumes.

Run every command from the project root in Windows PowerShell. Docker Desktop must be running.

## 1. Create network and volumes
```powershell
docker network create ecommerce-ms-network
docker volume create ecommerce_products_data
docker volume create ecommerce_orders_data
```

## 2. Build images
```powershell
docker build -t ecommerce-api-gateway:1.0 -f api-gateway/Dockerfile .
docker build -t ecommerce-product-service:1.0 -f product-service/Dockerfile .
docker build -t ecommerce-order-service:1.0 -f order-service/Dockerfile .
```

## 3. Run product database
```powershell
docker run -d --name product-db --network ecommerce-ms-network -e MYSQL_ROOT_PASSWORD="ProductsPass_123!" -e MYSQL_DATABASE="products_db" --mount "type=volume,source=ecommerce_products_data,target=/var/lib/mysql" --mount "type=bind,source=$((Get-Location).Path)\product-service\schema.sql,target=/docker-entrypoint-initdb.d/01-schema.sql,readonly" mysql:8.4
```

## 4. Run order database
```powershell
docker run -d --name order-db --network ecommerce-ms-network -e MYSQL_ROOT_PASSWORD="OrdersPass_123!" -e MYSQL_DATABASE="orders_db" --mount "type=volume,source=ecommerce_orders_data,target=/var/lib/mysql" --mount "type=bind,source=$((Get-Location).Path)\order-service\schema.sql,target=/docker-entrypoint-initdb.d/01-schema.sql,readonly" mysql:8.4
```

Wait for databases to be ready:
```powershell
docker logs -f product-db
```
When it says `ready for connections`, press Ctrl+C, then do the same for:
```powershell
docker logs -f order-db
```

## 5. Run Product Service
```powershell
docker run -d --name product-service --network ecommerce-ms-network -e DB_HOST="product-db" -e DB_USER="root" -e DB_PASSWORD="ProductsPass_123!" -e DB_NAME="products_db" ecommerce-product-service:1.0
```

## 6. Run Order Service
```powershell
docker run -d --name order-service --network ecommerce-ms-network -e DB_HOST="order-db" -e DB_USER="root" -e DB_PASSWORD="OrdersPass_123!" -e DB_NAME="orders_db" -e PRODUCT_SERVICE_URL="http://product-service:4001" ecommerce-order-service:1.0
```

## 7. Run API Gateway
```powershell
docker run -d --name ecommerce-gateway --network ecommerce-ms-network -p 3001:3000 -e PRODUCT_SERVICE_URL="http://product-service:4001" -e ORDER_SERVICE_URL="http://order-service:4002" ecommerce-api-gateway:1.0
```

Open http://localhost:3001.

## Check status/logs
```powershell
docker ps
docker logs ecommerce-gateway
docker logs product-service
docker logs order-service
docker logs product-db
docker logs order-db
docker volume ls
```

## Stop services
```powershell
docker stop ecommerce-gateway order-service product-service order-db product-db
```

Volumes persist when containers are removed. Do not remove `ecommerce_products_data` or `ecommerce_orders_data` if you need to keep the database data. SQL initialization scripts only run when the corresponding volume is empty.

Demo limitations: checkout is simulated; no authentication or real payment. Order storage and product stock updates are across services and are not one atomic distributed transaction, so this is for learning rather than production.
