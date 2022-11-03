```c++
Miku {
   ScreenTrack createScreenTrack()
   MicrophoneTrack createMicrophoneTrack()
   CameraTrack createCameraTrack()

   // YUV
   ExternalVideoTrack createExternalVideoTrack(VideoFrameSender sender)
   ExternalAudioTrack createExternalAudioTrack(AudioFrameSender sender)

   // H264/G711
   ExternalVideoTrack createExternalVideoTrack(VideoPacketSender sender)
   ExternalAudioTrack createExternalAudioTrack(AudioPacketSender sender)

   // sender
   VideoFrameSender createVideoFrameSender()
   AudioFrameSender creaetAudioFrameSender()
   VideoPacketSender createVideoPacketSender
   AudioPacketSender createAudioPacketSender

   RtcConnection createRtcConnection()
   RtmpConnection createRtmpConnection()
   GbConnection createGbConnection()
}
```

[Tracks](./miku-v1-tracks.md)


```c++
Connection {
   setConnectionStateListener()
}

RtmpConnection : Connection {
    MikuClient connect(url)
}
 
RTCConnection : Connection {
    MikuClient connect(token)  
}

// 国标对接推流部分都是集成设备内部，优先级低
GBConnection : Connection {
    MikuClient connect(device id)  
}

MikuClient {
    publish(tracks)
}

FilterInterface {
  setEnabled(bool enabled)
  isEnabled()
}

VideoFilterInterface {
  adaptVideoFrame(VideoFrame)
}

AudioFilterInterface {
  adaptAudioFrame(AudioFrame)
}

RenderInterface {
  onVideoFrame(VideoFrame)
}

EncoderInterface {
    initEncode()
    EncodedImage encodeFrame(VideoFrame)
    deinitEncode();
}

MediaPluginFactory {
  creaetAudioFilter()
  creaetVideoFilter()
  createEncoder()
  createRender()
}
```

source track 分离
