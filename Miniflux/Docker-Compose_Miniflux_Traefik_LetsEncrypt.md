# 使用 Docker 自建 Miniflux 服务

在 VPS 自建 RSS 阅读服务，浏览第一手信息资讯。搭建服务用到的一些服务：

- VPS 主机，操作系统为 Debian 11 (bullseye) 或其他 Linux 发行版；
- Docker-Compose: 在 Docker 容器中运行，配置文件中 yml 文件中直观方便；
- Miniflux: Miniflux 是最优秀的 RSS 阅读工具之一，开源、轻量、稳定；
- Traefik: 反向代理工具，与 Docker 是很好的组合，方便解决网站证书；
- Let's Encrypt: 实现网站的加密访问；
- Domain: 拥有自己可以设置 DNS 的域名，可以申请 Let's Encrypt 证书。

## 配置文件示例```traefik.yml```，按下方提示修改配置文件的内容

```version: '3.4'
services:
  traefik:
    image: "traefik:v2.3"
    container_name: traefik
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=postmaster@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    depends_on:
      - miniflux
    ports:
      - "443:443"
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
  miniflux:
    image: ${MINIFLUX_IMAGE:-miniflux/miniflux:latest}
    container_name: miniflux
    restart: always
    depends_on:
      - db
    expose:
      - "8080"
    environment:
      - DEBUG=0
      - LOG_DATE_TIME=1
      - POLLING_FREQUENCY=15
      - CLEANUP_FREQUENCY_HOURS=876000
      - CLEANUP_ARCHIVE_READ_DAYS=36500
      - CLEANUP_REMOVE_SESSIONS_DAYS=36500
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
      - RUN_MIGRATIONS=1
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=webLogintest123
      - BASE_URL=https://miniflux.example.org
    healthcheck:
      test: ["CMD", "/usr/bin/miniflux", "-healthcheck", "auto"]
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.miniflux.rule=Host(`miniflux.example.org`)"
      - "traefik.http.routers.miniflux.entrypoints=websecure"
      - "traefik.http.routers.miniflux.tls.certresolver=myresolver"
  db:
    image: postgres:12
    container_name: postgres
    environment:
      - POSTGRES_USER=miniflux
      - POSTGRES_PASSWORD=secret
    volumes:
      - miniflux-db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "miniflux"]
      interval: 10s
      start_period: 30s
volumes:
  miniflux-db:
```
- 修改第 25 行中 ```postmaster@example.com```  字段为自己的邮箱；
- 修改第 35 行中 ```secret```  字段为数据库密码；
- 修改第 39 行中 ```webLogintest123```  Web 登录的密码，用户名默认 ```admin``` ；
- 修改第 40 行中 ```miniflux.example.org```  字段为一级或二级域名，Web 访问；
- 修改第 45 行中 ```miniflux.example.org```  字段为一级或二级域名，申请证书；
- 修改第 53 行中 ```secret```  字段为数据库密码。

```
mkdir miniflux && cd miniflux
wget https://raw.githubusercontent.com/aturl/awesome-anti-gfw/master/Miniflux/traefik.yml
```
按上面建议的修改说明将 ```traefik.yml``` 改成自己的设定，拉取容器镜像然后运行服务。

```
docker-compose -f traefik.yml up -d db
docker-compose -f traefik.yml up
```
参考 Miniflux v2 项目文档 https://github.com/miniflux/v2/tree/master/contrib/docker-compose
