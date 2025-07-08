---
title: "Ffmpeg"
date: 2023-02-12T07:56:49+08:00
author: ["loveyu"]
draft: false
categories: 
- toher
tags: 
- ffmpeg
- other
---

1. ```shell
   ffprobe -i test.mp4 -v quiet -print_format json -show_format -show_streams
   ```

   >```go
   >type VideoInfoData struct {
   >	Stream []Streams `json:"streams"` //视频数据流,包括视频音频字幕
   >	Format Format    `json:"format"`  //格式化
   >}
   >
   >type Streams struct {
   >	CodecName string `json:"codec_name"`        //编码
   >	Width     int    `json:"width,omitempty"`   //宽
   >	Height    int    `json:"height,omitempty"`  //高
   >	PixFmt    string `json:"pix_fmt,omitempty"` // 分辨率
   >	Duration  string `json:"duration"`          //视频时长
   >}
   >
   >type Format struct {
   >	BitRate string `json:"bit_rate"` //速率
   >}
   >```
   >
   >把json数据反序列化为结构体

2. ```shell
   提取test.mp4的音频文件保存为audio.m4a
   ffmpeg -hide_banner -i test.mp4 -vn -c copy audio.m4a
   ```

3. 

3. ```shell
   视频分辨率转换,把test.mp4转化为1920x1080分辨率3000k码率的视频
   ffmpeg -hide_banner -i test.mp4 -crf 20 -c:v libx264 -an -s 1920x1080 -r 300
   00/1001 -b:v 3000k tmp_1080p_3000k.mp4
   ```

4. >分辨率对应:
   >
   >1080: "1920x1080", "-r", "30000/1001", "-b:v", "3000k"
   >
   >720: "1080x720", "-r", "30000/1001", "-b:v", "2000k"
   >
   >480: "854x480", "-r", "30000/1001", "-b:v", "900k"
   >
   >360: "640x360", "-r", "30000/1001", "-b:v", "500k"

4. ```shell
   ffmpeg -i ./upload/video/v_cr41aouei3zc847/tmp_360p_500k.mp4 -i ./upload/video/v_cr41aouei3zc847/tmp_480p_900k.mp4 -i ./upload/video/v_cr41aouei3zc847/tmp_720p_2000k.mp4 -i ./upload/video/v_cr41aouei3zc847/tmp_1080p_3000k.mp4 -i ./upload/video/v_cr41aouei3zc847/audio.m4a -c copy -map 0 -map 1 -map 2 -map 3 -map 4 -f dash -init_seg_name v_cr41aouei3zc847-init-$RepresentationID$.m4s -media_seg_name v_cr41aouei3zc847-$RepresentationID$-$Number%05d$.m4s ./upload/video/v_cr41aouei3zc847/index.mpd
   
   
   ffmpeg -i tmp_1080p_3000k.mp4 -i audio.m4a -c copy -map 0 -f dash -init_seg_name v_cr41aouei3zc847-init-$RepresentationID$.m4s -media_seg_name v_cr41aouei3zc847-$RepresentationID$-$Number%05d$.m4s index.mpd
   ```

5. 
