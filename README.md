#  部署一个简单的私人直播平台

#### 确保您的系统已安装 Docker 和 Docker Compose。


### 1：下载项目：
```
git clone https://gitee.com/wanfeng789/live-rtmp.git
```


### 2: 运行

进入项目目录```cd /root/live-rtmp```

修改服务器IP为你的实际服务器IP


启动：

```
docker-compose up -d
```

访问你的`服务器IP地址`查看前端页面

### 3：配置直播软件
使用 OBS 或FFmpeg等软件推流直播。
推流地址为 ```rtmp://服务器IP:1935/live/admin``` &nbsp;&nbsp;&nbsp;  提示：admin为串流密钥，可在配置文件中修改。


### 4：查看直播

浏览器地址输入你的`服务器IP`查看您的直播。或者用反代实现域名访问。


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



