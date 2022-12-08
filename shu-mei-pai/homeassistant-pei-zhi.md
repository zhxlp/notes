# HomeAssistant 配置

## HomeKit

* 编辑`/home/homeassistant/.homeassistant/configuration.yaml`配置文件,添加\*\*`homekit:`\*\*,重启服务

## 海康摄像头

*   安装依赖

    ```bash
    apt install ffmpeg libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libavfilter-dev libswscale-dev libswresample-dev
    ```
*   编辑`/home/homeassistant/.homeassistant/configuration.yaml`配置文件,添加如下内容

    ```properties
    stream:
    camera:
      - platform: generic
        name: Streaming Enabled
        username: admin
        password: Password
        authentication: basic
        still_image_url: http://192.168.2.212:80/streaming/channels/1/picturec
        stream_source: rtsp://admin:Password@192.168.2.212/h264/ch1/main/av_stream
    ```
