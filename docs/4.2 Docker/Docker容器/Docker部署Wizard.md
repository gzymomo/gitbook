```bash
docker run -d --name wizard -e DB_HOST=xxx -e DB_PORT=3308 -e DB_DATABASE=wizard -e DB_USERNAME=root -e DB_PASSWORD=123456  -p 8080:80 -v /wizard:/webroot/storage/app/public mylxsw/wizard
```



Wizard上传附件大小配置文件地址：

```bash
/usr/local/etc/php/conf.d/upload-limit.ini
upload_max_filesize = 100M
post_max_size = 0
```

