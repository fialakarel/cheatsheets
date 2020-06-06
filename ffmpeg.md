# ffmpeg

## Extract one frame from video at the given time using ffmpeg
```bash
ffmpeg -ss 00:03:00 -i invideo.mp4 -vframes 1 -q:v 5 output.jpg
```

## Extract cropped video
```bash
ffmpeg \
  -i invideo.mp4 \
  -ss 13 \
  -t 00:41:10 \
  -loglevel error \
  -stats \
  -threads 0 \
  -f mp4 \
  -c:v libx264 \
  -preset medium \
  -crf 26 \
  -tune fastdecode \
  -movflags faststart \
  -pix_fmt yuv420p \
  -filter:v "crop=w=1920:h=1025:x=0:y=150" \
  -c:a copy \
  output.mp4
```
