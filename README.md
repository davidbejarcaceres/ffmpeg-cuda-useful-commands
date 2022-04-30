# Custom commands for ffmpeg video conversion (CUDA):
Here we can find a set of commands to use in video conversion with [ffmpeg](https://ffmpeg.org/) and common configurations (resolution, bitrate, encoding, etc), some custom commands to [transcode using cuda](https://developer.nvidia.com/blog/nvidia-ffmpeg-transcoding-guide/) and [AMD AMF](https://gpuopen.com/advanced-media-framework/)

## Codecs (CUDA):
### Decoders for cuda:
    1. h264_cuvid -> Nvidia CUVID H264 decoder (codec h264)
    2. hevc_cuvid -> Nvidia CUVID HEVC decoder (codec hevc)
    3. av1_cuvid -> Nvidia CUVID AV1 decoder (codec av1)
    4. mjpeg_cuvid -> Nvidia CUVID MJPEG decoder (codec mjpeg)
    5. mpeg1_cuvid -> Nvidia CUVID MPEG1VIDEO decoder (codec mpeg1video)
    6. mpeg2_cuvid -> Nvidia CUVID MPEG2VIDEO decoder (codec mpeg2video)
    7. mpeg4_cuvid -> Nvidia CUVID MPEG4 decoder (codec mpeg4)
    8. vc1_cuvid -> Nvidia CUVID VC1 decoder (codec vc1)
    9. vp8_cuvid -> Nvidia CUVID VP8 decoder (codec vp8)
    10. vp9_cuvid -> Nvidia CUVID VP9 decoder (codec vp9)

### Encoders for cuda:
    1. h264_nvenc
    2. hevc_nvenc
    3. libsvtav1 -> AV1 (SVT-AV1 Encoder and Decoder)

### Encoder for AMD (up to 2x faster compared to libx264 | libx265):
    1. h264_amf
    2. hevc_amf

### Encoder for AV1:
    1. libsvtav1 -> AV1 (SVT-AV1 Encoder and Decoder) No GPU support


#########################################################################


# Most used ffmpeg flags:
 1. `-c:v` -> To inidicate codec of the video track
 2. `-c:a copy` -> To copy audio from source
 3. `-vf scale=1280:720` -> To resize the video to new resolution
 4. `-vf scale_cuda=1280:720` -> To resize with CUDA
 5. `-hwaccel cuvid` -> Inidicates Hardware acceleration to use (cuda | cuvid)
 6. `-hwaccel_output_format cuvid` -> Inidicates acceleration mode to use (cuda | cuvid)
 7. `-noautorotate` -> Prevents from auto rotate, sometimes ffmpeg rotate portrait videos

#########################################################################
# Examples:
## Only CPU:
This uses encoder h264 for video, and copy audio from metadata, uses -noautorotate in case is a portrait video
```console
ffmpeg -noautorotate -i input.mp4 -c:a copy -c:v h264 output.mp4
```

## Encode CPU AMD:
This uses AMD custom encoder, only for AMD CPUs, can accelerate up to 2x:
```console
ffmpeg -noautorotate -i input.mp4 -c:a copy -c:v h264 output.mp4
```

## Resize video CPU:
Resize video with CPU and filter `-vf scale=1280:720`:
```console
ffmpeg -noautorotate -i input.mp4 -vf scale=1280:720 -c:a copy -c:v h264 output.mp4
```

## Encode using GPU CUDA:
Transcode using CUDA, with encoder `h264_nvenc`:
```console
ffmpeg -noautorotate -y -vsync 0 -hwaccel cuvid -hwaccel_output_format cuvid -i input.mp4 -c:a copy -c:v h264_nvenc output.mp4 
```

## Transcode using GPU CUDA (max performance):
Transcode using CUDA, using decoder cuda `hevc_cuvid` for input stream video, can accelerate 25% the process:
```console
ffmpeg -noautorotate -y -vsync 0 -hwaccel cuvid -hwaccel_output_format cuvid -c:v hevc_cuvid -i input.mp4 -c:a copy -c:v hevc_nvenc output.mp4
```

## Resize video on GPU CUDA:
Resize video using CUDA, using filter `-vf scale_cuda=1280:720` for input stream video:
```console
ffmpeg -y -vsync 0 -hwaccel cuvid -c:v hevc_cuvid -i input.mp4 -vf scale_cuda=1280:720 -c:a copy -c:v hevc_nvenc  output.mp4
```

## Decode with GPU CUDA + Encode CPU (AV1):
Decode using CUDA and encode using CPU for the new AV1 codec, decoder cuda `hevc_cuvid`, and `libsvtav1` for encoding ([more about AV1](https://github.com/AOMediaCodec/SVT-AV1)):
```console
ffmpeg -y -vsync 0 -noautorotate -y -vsync 0 -hwaccel cuvid -hwaccel_output_format cuvid -c:v hevc_cuvid  -i input.mp4 -c:a copy -c:v libsvtav1 output.mp4
```