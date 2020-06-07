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

## Add subtitles into video as an optional track

This solution adds the subtitles to the video as a separate optional (and user-controlled) subtitle track.

```bash
ffmpeg -i infile.mp4 -i infile.srt -c:v copy -c:a copy -c:s mov_text outfile.mp4
```

Source: https://stackoverflow.com/a/17584272

## Burns subtitles into the video

This solution "burns the subtitles" into the video, so that every viewer of the video will be forced to see them.

* First convert the subtitles to .ass format:

```bash
ffmpeg -i subtitles.srt subtitles.ass
```

* Then add them using a video filter:

```bash
ffmpeg -i mymovie.mp4 -vf ass=subtitles.ass mysubtitledmovie.mp4
```

Source: https://stackoverflow.com/a/13125122
