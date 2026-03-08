# Chạy Camunda 8 trên Docker

## Giới thiệu
Camunda 8 là nền tảng tự động hóa quy trình (process automation) hiện đại, hỗ trợ BPMN 2.0, DMN và Forms. Docker giúp bạn nhanh chóng khởi chạy Camunda 8 trên môi trường local để phát triển và testing.

## Yêu cầu
- Docker Desktop hoặc Docker Engine (20.10+)
- Docker Compose (v2.0+)
- Tối thiểu 4GB RAM khả dụng
- Tối thiểu 10GB dung lượng ổ đĩa

## Cài đặt nhanh với Docker Compose

### Tải Camunda 8 Docker Compose
Clone repository chính thức:

```bash
git clone https://github.com/camunda/camunda-platform.git
cd camunda-platform
```

### Khởi chạy Camunda 8
Chạy lệnh sau để start tất cả services:

```bash
docker compose up -d
```

Các services sẽ được khởi động:
- **Zeebe**: Process engine (port 26500)
- **Operate**: Monitoring và operations UI (port 8081)
- **Tasklist**: User task management UI (port 8082)
- **Optimize**: Analytics và reporting (port 8083)
- **Identity**: User management (port 8084)
- **Elasticsearch**: Data storage
- **Keycloak**: Authentication

### Kiểm tra trạng thái
Kiểm tra các container đang chạy:

```bash
docker compose ps
```

## Truy cập các ứng dụng

Sau khi khởi động thành công, truy cập:

| Ứng dụng | URL | Mô tả |
|----------|-----|-------|
| Operate | http://localhost:8081 | Giám sát process instances |
| Tasklist | http://localhost:8082 | Quản lý user tasks |
| Optimize | http://localhost:8083 | Phân tích và báo cáo |
| Identity | http://localhost:8084 | Quản lý users và permissions |

**Thông tin đăng nhập mặc định:**
- Username: `demo`
- Password: `demo`

## Deploy Process đầu tiên

### Sử dụng zbctl CLI
Cài đặt zbctl (Zeebe CLI):

```bash
# macOS
brew install zbctl

# Linux/Windows
# Download từ https://github.com/camunda/zeebe/releases
```

Deploy một BPMN process:

```bash
zbctl deploy process.bpmn --insecure
```

### Sử dụng Modeler
1. Tải [Camunda Modeler](https://camunda.com/download/modeler/)
2. Tạo BPMN diagram
3. Click "Deploy" và configure connection:
   - Cluster endpoint: `http://localhost:26500`
   - Authentication: None (hoặc OAuth nếu đã cấu hình)

## Cấu hình nâng cao

### Giới hạn tài nguyên
Chỉnh sửa `docker-compose.yml` để giới hạn tài nguyên:

```yaml
services:
  zeebe:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

### Persistence data
Tạo volumes để giữ dữ liệu:

```yaml
volumes:
  zeebe_data:
  elasticsearch_data:
```

### Environment variables
Customize qua file `.env`:

```properties
ZEEBE_BROKER_EXPORTERS_ELASTICSEARCH_ARGS_INDEX_PREFIX=camunda
ZEEBE_BROKER_CLUSTER_PARTITIONS_COUNT=3
```

## Quản lý và bảo trì

### Xem logs
Xem logs của một service cụ thể:

```bash
docker compose logs -f zeebe
```

### Dừng và khởi động lại

```bash
# Dừng tất cả services
docker compose down

# Dừng và xóa volumes (mất dữ liệu)
docker compose down -v

# Khởi động lại
docker compose up -d

# Restart một service
docker compose restart zeebe
```

### Health check
Kiểm tra sức khỏe của Zeebe:

```bash
curl http://localhost:9600/ready
```

## Troubleshooting

### Container không start
- Kiểm tra logs: `docker compose logs [service-name]`
- Xác nhận ports không bị xung đột
- Tăng memory cho Docker Desktop

### Kết nối bị từ chối
- Đảm bảo services đã ready (check logs)
- Đợi 2-3 phút để Elasticsearch hoàn tất khởi động
- Kiểm tra firewall settings

### Performance chậm
- Tăng memory allocation cho Docker
- Giảm số partitions trong Zeebe config
- Disable Optimize nếu không cần thiết

## Sử dụng trong Development

### Spring Boot Integration
Thêm dependency vào `pom.xml`:

```xml
<dependency>
    <groupId>io.camunda</groupId>
    <artifactId>spring-zeebe-starter</artifactId>
    <version>8.5.0</version>
</dependency>
```

Cấu hình trong `application.yml`:

```yaml
zeebe:
  client:
    broker:
      gateway-address: localhost:26500
    security:
      plaintext: true
```

### Worker đơn giản

```java
@JobWorker(type = "payment-service")
public void processPayment(final JobClient client, final ActivatedJob job) {
    Map<String, Object> variables = job.getVariablesAsMap();

    // Xử lý logic
    System.out.println("Processing payment: " + variables);

    // Complete job
    client.newCompleteCommand(job.getKey())
        .variables(Map.of("paid", true))
        .send()
        .join();
}
```

## Best Practices

1. **Development vs Production**: Không dùng Docker Compose setup này cho production
2. **Data backup**: Thường xuyên backup Elasticsearch volumes
3. **Security**: Đổi password mặc định khi expose ra internet
4. **Resource monitoring**: Theo dõi CPU/Memory usage
5. **Version pinning**: Lock versions trong docker-compose.yml

## Dọn dẹp

Xóa hoàn toàn Camunda 8:

```bash
docker compose down -v --remove-orphans
docker system prune -a
```

## Tham khảo
- [Camunda 8 Docs](https://docs.camunda.io/)
- [Docker Compose GitHub](https://github.com/camunda/camunda-platform)
- [Zeebe Documentation](https://docs.camunda.io/docs/components/zeebe/zeebe-overview/)
- [Community Forum](https://forum.camunda.io/)
