# 脚本

## mp4 转 m3u8

```bash
ffmpeg -i test.mp4 \
  -forced-idr 1 -force_key_frames "expr:gte(t,n_forced*2)" \
  -strict -2 -c:a aac -c:v libx264 \
  -f hls -hls_list_size 0 -hls_time 2 .\m3u8\index.m3u8
```

`-forced-idr 1`    强行设置关键帧为IDR帧

`-force_key_frames "expr:gte(t,n_forced*2)"`   每2秒插入关键帧

`-hls_time 2` 2秒分割

`-hls_list_size 0`  ts文件数量不限制

`-strict -2` 使用FFmpeg自带的aac音频编码

`-c:v`  与 `-codec:v` 和 `-vcodec` 等价，视频编解码器

`-c:a` 音频编解码器







