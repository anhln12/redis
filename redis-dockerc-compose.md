docker-compose.yml
```
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis_app
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: >
      redis-server
      --requirepass "Thay_Mat_Khau_Manh_Vao_Day"
      --appendonly yes
      --protected-mode yes
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "Thay_Mat_Khau_Manh_Vao_Day", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
    driver: local
```

Truy cập CLI inside container để verify
```
docker exec -it redis_app redis-cli -a "Thay_Mat_Khau_Manh_Vao_Day" ping
# Trả về: PONG
```
