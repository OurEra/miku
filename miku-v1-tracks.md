```c++
Track {
  State getState()
  Stats getStats()
}

VideoTrack {
    addRender(RenderInterface)
    addFilter(VideoFilterInterface)
}

AudioTrack {
  setVolume(float volume)
  addFilter(AudioFilterInterface)
}

LocalVideoTrack : VideoTrack {
    addEncoder(EncoderInterface)
}

LocalAudioTack : AudioTrack {
    addEncoder(EncoderInterface)
}

ScreenVideoTrack : LocalVideoTack {
    bool isSupported()    
    startShare()
    stopShare()
}

MicrophoneTrack : LocalAudioTrack {
    startRecord()
    stopRecord()
}

CameraTrack : LocalVideoTack {
    startCapture();
    stopCapture();
}

ExternalVideoTrack : LocalVideoTack {

}

ExternalAudioTrack : LocalAudioTrack {

}

RemoteVideoTrack : VideoTrack {

}

RemoteAudioTrack : AudioTrack {

}

VideoFrameSender {
    sendVideoFrame(VideoFrame frame)
}

AudioFrameSender {
    sendAudioFrame(AudioFrame frame)
}

VideoPacketSender {
    sendVideoPacket(VideoPacket packet)
}

AudioPacketSender {
    sendAudioPacket(AudioPacket packet)
}

```
