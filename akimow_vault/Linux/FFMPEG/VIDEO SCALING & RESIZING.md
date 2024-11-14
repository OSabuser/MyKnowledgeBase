There is a difference between Sample Aspect Ratio (SAR) and Display Aspect Ratio (DAR). If you want to change the video to display at 4:3, you will either need to change the actual pixels in the image (by scaling the pixels and changing SAR), or by setting a metadata flag that at the container level that tells external media players to stretch the image to your desired DAR.

You will not be able to scale the pixels and change SAR without applying a video filter. If you choose this method, you will be required to transcode the file - since you cannot "stream copy" the video stream while applying a video filter.

To scale the image and change SAR (while transcoding), try:

```
ffmpeg -i <INPUT_FILE> -vf scale=720:540 -c:v <Video_Codec> <OUTPUT_FILE>
```

On the other hand, if you just want to change the metadata flag and adjust the DAR, you will be able to stream copy the video. To do this, try:

```
ffmpeg -i <INPUT_FILE> -aspect 720:540 -c copy [OUTPUT_FILE]
```


For scaling video with FFmpeg, you apply the -vf scale filter, which means "video filter." This filter assists in specifying the width and height that you want for your resulting video.

### Code Syntax with Example

To resize a video to a specific resolution, such as 1280x720 pixels, use the following FFmpeg command:

```
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4
```

In this command:

- -i input.mp4 specifies the input video file.
- -vf scale=1280:720 applies the scaling filter to resize the video to 1280 pixels in width and 720 pixels in height.
- output.mp4 is the name of the resized output video file.

This command will make the input video have a resolution of 1280x720 pixels, using FFmpeg to resize its [video resolution](https://www.gumlet.com/glossary/video-resolution/) to 720p. This is a handy method for converting videos into standard resolutions such as 720p, [1080p](https://www.gumlet.com/learn/4k-vs-1080p/#what-is-1080p-resolution) or your own desired size.

This scale filter helps FFmpeg scale more straightforwardly and simply, letting you adjust the size of your videos as needed. It also ensures that your videos can be used on various platforms and devices.

## How to Resize a Video and Keep Aspect Ratio Using FFmpeg?

Maintaining the [aspect ratio](https://www.gumlet.com/learn/video-aspect-ratio/) is crucial to avoid stretching or squashing the video. To ensure the original aspect ratio is preserved, you can use FFmpeg's -vf scale filter with a dynamic scaling approach.

### Code Syntax with Example

To resize a video to half its original width and height while maintaining the aspect ratio, use the following FFmpeg command:

```
ffmpeg -i input.mp4 -vf "scale=iw*0.5:ih*0.5" output.mp4
```

In this command:

- iw and ih represent the input video’s width and height.
- iw*0.5:ih*0.5 scales the width and height to 50% of their original values.

This method ensures that the video is resized without losing its original aspect ratio, making it ideal for various playback devices and screen sizes.

## How to Crop a Video Using FFmpeg?

Cropping a video involves selecting a specific rectangular region from the original video. This is useful for removing unwanted parts or focusing on a particular section of the video.

### Code Syntax with Example

To crop a video to a 640x480 rectangle starting at position (100, 100) from the top-left corner, use the following FFmpeg command:

```
ffmpeg -i input.mp4 -vf "crop=640:480:100:100" output.mp4
```

In this command:

- crop=640:480:100:100 defines the width, height, and top-left corner coordinates of the crop rectangle.

This command allows you to precisely control which part of the video to keep, making it a powerful tool for video editing and customization.

## How to Resize a Video with Black Bars Using FFmpeg?

Sometimes, you may want to resize a video but keep the original aspect ratio by adding black bars (letterboxing or pillarboxing). This ensures that the video fits within a specific resolution without distortion.

### Code Syntax with Example

To scale a video to 1280x720 and add black bars to preserve the aspect ratio, use the following FFmpeg command:

```
ffmpeg -i input.mp4 -vf "scale=1280:720, pad=1280:720:(ow-iw)/2:(oh-ih)/2" output.mp4
```

In this command:

- scale=1280:720 resizes the video to 1280x720 pixels.
- pad=1280:720:(ow-iw)/2:(oh-ih)/2 adds black bars to the top/bottom or sides to maintain the original aspect ratio.

This method ensures that the video fits perfectly within the specified resolution, making it suitable for various display formats.