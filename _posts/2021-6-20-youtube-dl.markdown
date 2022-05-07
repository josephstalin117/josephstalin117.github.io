---
layout: post
title:  "youtube-dl"
date:   2021-6-20 20:35:35 +0800
categories: command
---

#### youtube-dl

```
youtube-dl.exe --proxy "socks5://127.0.0.1:1081/" -f 137+bestaudio --merge-output-format mkv "youtube url"
```

查看格式
```
youtube-dl.exe --proxy "socks5://127.0.0.1:1081/" -F --merge-output-format mkv "youtube url"
```


#### ffmpeg

```
# 查看格式
ffmpeg -formats

$ ffmpeg \
[全局参数] \
[输入文件参数] \
-i [输入文件] \
[输出文件参数] \
[输出文件]
```

mkv to mp4
```
ffmpeg -i LostInTranslation.mkv -codec copy LostInTranslation.mp4
```

烧入字幕
```
ffmpeg -i mymovie.mp4 -vf subtitles=subtitles.srt mysubtitledmovie.mp4
```
