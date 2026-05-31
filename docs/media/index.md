---
title: Media
icon: fontawesome/solid/play
#icon: material/stadia_controller
---

CLI player: `mpv`, otherwise `vlc`

Extract audio from video:

```
ffmpeg -i input_video.mp4 -vn -c:a copy output_audio.m4a
```

Crop video/audio:

```
ffmpeg -ss 00:02:00 -to 00:05:30 -i input.m4a -c copy output.m4a
```

We can skip start or end time and it will crop just start or end.
