# Hướng dẫn kết nối Docker Stack với các dịch vụ chia sẻ

## Tổng quan
File `docker-compose.yml` đã được cấu hình để sử dụng external network, cho phép các stack Docker khác kết nối vào các dịch vụ MySQL, Redis và RabbitMQ.

## Cấu hình đã thay đổi

### Network Configuration
```yaml
networks:
  vinhtanfarm-network:
    driver: bridge
    external: true
    name: vinhtanfarm-shared-network
```

**Giải thích:**
- `external: true`: Cho phép network này được sử dụng bởi các stack khác
- `name: vinhtanfarm-shared-network`: Tên network thực tế sẽ được tạo

## Cách sử dụng

### 1. Tạo network chia sẻ (chỉ cần làm một lần)
```bash
docker network create vinhtanfarm-shared-network
```

### 2. Khởi động stack chính
```bash
docker-compose up -d
```

### 3. Kết nối từ stack khác

Trong file `docker-compose.yml` của stack khác, thêm cấu hình network:

```yaml
version: "3.8"

services:
  your-app:
    image: your-app:latest
    networks:
      - vinhtanfarm-shared-network
    # Có thể kết nối đến:
    # - MySQL: mysql:3306
    # - Redis: redis:6379  
    # - RabbitMQ: rabbitmq:5672

networks:
  vinhtanfarm-shared-network:
    external: true
```

## Thông tin kết nối

### MySQL
- **Host**: `mysql`
- **Port**: `3306`
- **User**: `root`
- **Password**: (empty)

### Redis
- **Host**: `redis`
- **Port**: `6379`

### RabbitMQ
- **Host**: `rabbitmq`
- **Port**: `5672` (AMQP)
- **Management UI**: `15672`

## Ví dụ kết nối từ ứng dụng

### Node.js với MySQL
```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({
  host: 'mysql',
  user: 'root',
  password: '',
  database: 'your_database'
});
```

### Python với Redis
```python
import redis
r = redis.Redis(host='redis', port=6379, db=0)
```

### Java với RabbitMQ
```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("rabbitmq");
factory.setPort(5672);
```

## Lưu ý quan trọng

1. **Network phải được tạo trước**: Chạy `docker network create vinhtanfarm-shared-network` trước khi khởi động bất kỳ stack nào
2. **Tên service**: Sử dụng tên service (mysql, redis, rabbitmq) làm hostname khi kết nối
3. **Firewall**: Đảm bảo các port cần thiết không bị chặn
4. **Health checks**: Các service đã có health check, đợi chúng ready trước khi kết nối

## Troubleshooting

### Kiểm tra network
```bash
docker network ls
docker network inspect vinhtanfarm-shared-network
```

### Kiểm tra containers trong network
```bash
docker network inspect vinhtanfarm-shared-network --format='{{range .Containers}}{{.Name}} {{end}}'
```

### Test kết nối
```bash
# Từ container khác, test ping
docker exec -it your-container ping mysql
docker exec -it your-container ping redis
docker exec -it your-container ping rabbitmq
```
