# Docker部署

```kotlin
docker run -p 9000:9000 --name myminio \
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -v /mnt/data:/data \
  -v /mnt/config:/root/.minio \
  minio/minio server /data
```

## S3网关

```bash
docker run -p 9000:9000 --name minio-s3 \
 -e "MINIO_ACCESS_KEY=access_key" \
 -e "MINIO_SECRET_KEY=secret_key" \
 minio/minio gateway s3 https://s3_compatible_service_endpoint:port
```



# docker-compose.yml

```kotlin
version: '3'
services:
  minio:
    image: minio/minio:latest
    container_name: myminio
    ports:
      - 9000:9000
    volumes:
      - /var/minio/data:/data
      - /var/minio/config:/root/.minio
    environment:
      MINIO_ACCESS_KEY: "root"
      MINIO_SECRET_KEY: "password"
    command: server /data
    restart: always
```