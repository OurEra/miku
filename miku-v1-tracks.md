Track {
  State getState()
  Stats getStats()
}

VideoTrack {
     addRender(RenderInterface)
     
}

AudioTrack {
  setVolume(float volume)
}

LocalVideoTrack : VideoTrack {
    addEncoder(EncoderInterface)
}

LocalAudioTack : AudioTrack {
}

ScreenTrack : LocalVideoTack {

}

MicrophoneTrack : LocalAudioTrack {

}

CameraTrack : LocalVideoTack {

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

