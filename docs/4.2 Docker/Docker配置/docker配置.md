```bash
vi /etc/docker/daemon.json
{
  "registry-mirrors": ["https://bk6kzfqm.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.0.241"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

