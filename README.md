#  部署一个简单的私人直播平台

#### 确保您的系统已安装 Docker 和 Docker Compose。


下载项目：
```
git clone https://gitee.com/wanfeng789/live-rtmp.git
```
### 步骤 1：在root目录下创建您的项目文件

 ```
 mkdir ~/live-rtmp && cd ~/live-rtmp
```


### 步骤 2: 设置 Nginx RTMP
在项目目录中，创建 ```docker-compose.yml``` 文件，并添加下面的内容：

```
touch docker-compose.yml
```

```
version: '3'

services:
  nginx-rtmp:
    image: tiangolo/nginx-rtmp
    ports:
      - "1935:1935" # RTMP 端口
      - "8080:8080" # HLS 流的 HTTP 端口
    volumes:
      - ./web/nginx.conf:/etc/nginx/nginx.conf # 映射自定义 Nginx RTMP 配置
      - /mnt/hls:/mnt/hls # 映射 HLS 目录
    restart: always

  web-server:
    image: nginx
    volumes:
      - ./web:/usr/share/nginx/html # web 目录包含您的 HTML 文件
    ports:
      - "80:80" # Web 服务器端口
    restart: always

```
创建一个用于网页服务的子目录和相关文件：

```
mkdir -p web && touch web/nginx.conf web/index.html && sudo mkdir -p /mnt/hls && sudo chmod -R 755 /mnt/hls
```
编辑 ```web/nginx.conf```添加以下内容：

```
worker_processes auto;
rtmp_auto_push on;

events {}

rtmp {
    server {
        listen 1935; # RTMP 监听端口
        listen [::]:1935 ipv6only=on;    

        application live {
            live on;
            record off;

            # HLS 设置
            hls on;
            hls_path /mnt/hls/; # HLS 文件存储位置
            hls_fragment 3s; # 每个 HLS 分段的长度
            hls_playlist_length 60s; # HLS 播放列表的总长度
        }
    }
}

http {
    server {
        listen 8080; # HTTP 服务监听端口

        # HLS 文件提供位置
        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8; # HLS 播放列表文件类型
                video/mp2t ts; # HLS 视频分段文件类型
            }

            root /mnt/; # HLS 文件的根目录
            add_header Cache-Control no-cache; # 防止 HLS 文件缓存
            add_header Access-Control-Allow-Origin *; # 允许所有域名跨域访问
        }
    }
}

```
编辑 ```web/index.html```添加您的网页内容和视频播放器代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>私人直播间</title>
    <!-- 引入 DPlayer CSS -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/dplayer/dist/DPlayer.min.css">
    <!-- 引入 DPlayer 和 HLS.js -->
    <script src="https://cdn.jsdelivr.net/npm/dplayer/dist/DPlayer.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        body {
            background-image: url('https://i.miji.bid/2024/01/09/29e7db3b0af393722f068942a17c2fd0.png');  // 网页背景图片URL
            background-size: cover;
            margin: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .video-title {
            font-family: 'Arial', sans-serif;
            color: #E6E6FA;
            font-size: 2em;
            text-align: center;
            margin-bottom: 20px;
        }
        #dplayer-container {
            width: 70%;
            max-width: 100%;
            height: auto;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        #dplayer {
            width: 100%;
            border-radius: 10px;
            overflow: hidden;
            border: 5px solid black;
            box-shadow: 0 0 15px 10px rgba(255, 255, 255, 0.5);
        }
        @media (max-width: 600px) {
            #dplayer-container {
                width: 90%;
            }
        }
    </style>
</head>
<body>
    <h1 class="video-title">私人直播间</h1>
    <div id="dplayer-container">
        <div id="dplayer"></div>
    </div>
    <script>
        const dp = new DPlayer({
            container: document.getElementById('dplayer'),
            theme: '#E6E6FA',
            video: {
                url: 'http://服务器IP:8080/hls/admin.m3u8',  // admin为串流密钥可自定义
                type: 'hls',
                pic: 'https://img.win3000.com/m00/2b/2c/a196e8b13619029ac87a09adc6990fd7.jpg',  // 播放器封面图片URL
            },
            p2pConfig: {
                live: true  // 如果不是直播设为 false
            }
        });
    </script>
</body>
</html>
```

### 步骤 3: 运行 Docker 容器
在 ```docker-compose.yml``` 同目录下运行以下命令来启动容器：

```
docker-compose up -d
```

### 配置直播软件
使用 OBS 或FFmpeg等软件推流直播。
推流 URL 设置为 ```rtmp://服务器IP:1935/live/admin``` &nbsp;&nbsp;&nbsp; # admin为串流密钥，可自定义。


### 访问直播

通过访问 ```http://服务器IP``` 来查看您的直播。或者用反代实现域名访问。


---


# docker版ffmpeg无人值守推流直播：

先把MP4视频放入`/home/videos`目录下


```
mkdir /home/videos && cd /home/videos
```



```
docker run -d --restart always \
  --network host \
  -v /home/videos:/tmp/video \
  linuxserver/ffmpeg \
  -re -stream_loop -1 -i /tmp/video/视频文件名称.mp4 \
  -c:v libx264 -preset veryfast -b:v 1500k \
  -c:a aac -b:a 92k \
  -f flv "推流地址"
```


---

##  带宽码率推荐:

| 视频清晰度    | 建议视频码率 (kbps) | 音频码率 (kbps) | 大约占用带宽 (Mbps) |
|-------------|-------------------|----------------|------------------|
| 标清 480p  | 500 - 1500        | 128            | 1 - 2     |
| 高清 720p  | 1500 - 4000       | 128            | 2 - 4      |
| 超清 1080p | 3000 - 6000       | 128            | 4 - 7      |
| 2K           | 8000 - 20000      | 128            | 9 - 20     |
| 4K           | 15000 - 50000     | 128            | 15 - 50    |



---



