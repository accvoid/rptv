Docker（推荐同时安装 Docker Compose）项目指南

akiralereal/iptv 「直播」
cloudflare/cloudflared
cqshushu/iptv-spider 「直播」
deluan/navidrome
develop767/migu_video 「直播」
ghcr.nju.edu.cn/cqshushu/udp-stream 「直播」
guovern/iptv-api 「直播」
instituteiptv/iptv-trmas 「直播」
instituteiptv/smart-v1 「直播」
kakaxi088/zubo 「直播」
lampon/omnibox
metacubex/mihomo
sifan1/vodspider
xhongc/music_tag_web


1.omnibox （https://github.com/Silent1566/OmniBox-Spider）
services:
  omnibox:
    image: lampon/omnibox:latest
    container_name: omnibox
    restart: always
    environment:
      TZ: Asia/Shanghai
    ports:
      - "7023:7023"
    volumes:
      - ./vol1/1000/docker/smarttv/data:/app/data


2.udp-stream
version: '3.8'

services:
  udp-stream:
    container_name: udp-stream
    restart: always
    image: ghcr.nju.edu.cn/cqshushu/udp-stream:latest
    ports:
      - "5000:5000"          # 宿主机端口1977映射到容器5000
    volumes:
      - /vol1/1000/docker/udp-stream/config:/app/config          # 配置文件目录
      - /vol1/1000/docker/udp-stream/playlists:/app/playlists    # 播放列表目录

3.vodspider （不夜影视等源）
version: "3.8"

services:
  redis:
    image: redis:latest
    restart: always
    container_name: redis
    volumes:
      - ./redis/:/data
  clash:
    image: metacubex/mihomo:latest
    restart: always
    container_name: clash
    ports:
      - 9090:9090 # clash WEB UI
    volumes:
      - ./clash/:/root/.config/mihomo/
  vod:
    image: sifan1/vodspider:latest
    container_name: vod
    ports:
      - 8080:3000
    environment:
      REDIS_HOST: redis
      REDIS_PORT: 6379
      PROXY_HTTP: http://user:pwd@clash:7890 # 无鉴权可以删除user:pwd@
      PROXY_SOCKS5: socks5://user:pwd@clash:7890 # 无鉴权可以删除user:pwd@
    volumes:
      - ./vod/:/app
    depends_on:
      - redis
      - clash # 代理，网络已翻情况可以删除

4.smart （直播）
version: '3.8'

services:
  smart-v1:
    image: instituteiptv/smart-v1:latest
    container_name: smart-v1
    restart: always
    ports:
      - "5050:5050"
    environment:
      - SMART_FILE=/app/smart.txt
      - SMART_CUSTOM_TOKEN=abc123

5.iptv-trmas（直播汇总）
version: '3.8'
services:
  iptv-trmas:
    image:  instituteiptv/iptv-trmas:latest
    container_name: trmas_rust
    restart: unless-stopped
    ports:
      - "19890:19890"
    environment:

      - ADMIN_USER=myadmin
      - ADMIN_PASS=secret
      - PLAY_TOKEN=abc123

6.iptv-spider（直播）
version: '3'
services:
  iptv-spider:
    image: cqshushu/iptv-spider:latest
    container_name: iptv-spider
    restart: unless-stopped
    ports:
      - "50085:50085"
    volumes:
      - /vol1/1000/docker/iptv-spider data:/app/data

7.iptv-api（直播）
services:
  iptv-api:
    image: guovern/iptv-api:latest
    container_name: iptv-api
    restart: unless-stopped
    ports:
      - "81:8181"                     # 宿主机端口:容器端口，若80被占用可改为其他如8080
    volumes:
      - ./config:/iptv-api/config
      - ./output:/iptv-api/output
    environment:
      PUBLIC_SCHEME: "http"
      PUBLIC_DOMAIN: "192.168.110.32"  # 改为飞牛NAS的实际IP地址
      PUBLIC_PORT: "81"                # 与宿主机端口一致
      NGINX_HTTP_PORT: "8181"
      CDN_URL: ""
      HTTP_PROXY: ""

8.iptv （直播）
services:
  iptv:
    image: akiralereal/iptv:latest              # 使用最新版本镜像
    container_name: iptv                        # 自定义容器名称
    ports:
      - "1905:1905"                             # 宿主机:容器端口映射
    environment:
      - muserId=                                # 可选：咪咕账号ID（留空为游客模式）
      - mtoken=                                 # 可选：咪咕登录令牌（用于高画质/VIP）
      - mport=1905                              # 必须：容器监听端口，与 ports 对应
      - mhost=                                  # 可选：外部访问地址（如 http://test.com:1905）
      - mrateType=4                             # 画质：2=标清，3=高清，4=蓝光(需VIP)
      - mpass=                                  # 可选：访问密码（设置后访问: http://ip:port/密码/...）
    restart: always                             # 容器异常退出后自动重启

9.music-tag（无损音乐）
version: '3'

services:
  music-tag:
    image: xhongc/music_tag_web:latest
    container_name: music-tag-web
    ports:
      - "8008:8001"
    volumes:
      - /vol1/@team/music:/app/media:rw
      - /vol1/1000/docker/music_tag_web/config:/app/data
    command: /start
    restart: unless-stopped

10.navidrome（音乐软件）
version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    ports:
      - "4533:4533"  # 默认端口
    restart: unless-stopped
    volumes:
      - "/vol1/1000/docker/navidrome/data:/data"
      - "/vol1/@team/music:/music:ro"
    environment:
      - ND_SCANSCHEDULE=1h        # 每小时扫描一次音乐库
      - ND_LOGLEVEL=info
      - ND_SESSIONTIMEOUT=24h

