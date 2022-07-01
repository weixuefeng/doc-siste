# start

```
docker run -d -p 80:80 docker/getting-started
```

-  -d: run the container in detached mode (in the background) 后台运行
-  -p: 80:80 - map port 80 of the host to port 80 in the container,端口映射 主机:容器
-   docker/getting-started - the image to use, 使用镜像

> tip: docker run -dp 80:80 docker/getting-started

## 持久化数据
```
docker volume create todo-db
```

使用
```
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```