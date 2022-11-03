### 整体结构
```c++
class Miku {
    class MikuConnection {
    }

    class RtcConnection extends MikuConnection {
        // 一个 client 表示一个连接上的会话
        MikuRtcClient connect(token)        
    }

    class RtmpConnection extends MikuConnection {
        MikuRtmpClient connect(url)        
    }

    class MikuClient {
        getLocalUser()        
        publish(tracks)
    }

    class MikuRtcClient extends MikuClient {
        // 用户体现会话上的资源从属关系
        getRemoteUser()        
        subscribe()
    }

    class MikuRtmpClient extends MikuClient {
    }

    RtcConnection createRtcConnect()
    RtmpConnection createRtmpConnection()

    createTrack()
}
```

### Miku
```c++
Miku {
   ScreenTrack createScreenTrack()
   MicrophoneTrack createMicrophoneTrack()
   CameraTrack createCameraTrack()

    // audio track
    // build-in mic
    LocalAudioTrack createLocalAudioTrack()
    LocalAudioTrack createRecordingDeviceAudioTrack(RecordingDeviceSource audioSource, bool enableAec)
    LocalAudioTrack createCustomAudioTrack(AudioPcmDataSender audioSource)
    LocalAudioTrack createCustomAudioTrack(AudioEncodedFrameSender audioSource)
    LocalAudioTrack createCustomAudioTrack(MediaPacketSender source)
    LocalAudioTrack createMediaPlayerAudioTrack(MediaPlayerSource audioSource)

    // video track
    LocalVideoTrack createCameraVideoTrack(CameraCapturer videoSource)
    LocalVideoTrack createScreenVideoTrack(ScreenCapturer videoSource)
    LocalVideoTrack createCustomVideoTrack(VideoFrameSender videoSource)
    LocalVideoTrack createCustomVideoTrack(VideoEncodedImageSender videoSource)
    LocalVideoTrack createCustomVideoTrack(MediaPacketSender source)
    LocalVideoTrack createMediaPlayerVideoTrack(MediaPlayerSource videoSource)

    // connection
    RtcConnection createRtcConnection()
    RtmpConnection createRtmpConnection()

    // plugin
    getMediaSourcePlugin()
    getMediaSinkPlugin()
    getMediaFilterPlugin()
}
```

### Connection
```c++
Connection {
   setConnectionStateListener()
}

RtmpConnection extends Connection {
    MikuClient connect(url)
}
 
RTCConnection extends Connection {
    MikuClient connect(token)  
}
```

### Client
```c++
MikuClient {
    void publish(QNPublishResultCallback callback, List<QNLocalTrack> trackList)
    unpublish(tracks)
    List<QNLocalTrack> getPublishedTracks()
}
 
MikuRTMPClient extends MikuClient {

}
 
MikuRTCClient extends MikuClient {
    RTCClientListener {
        void onUserJoined(String remoteUserID, String userData);
        void onUserLeft(String remoteUserID);
        void onUserRejoining(String remoteUserID);
        void onUserRejoined(String remoteUserID);

        void onUserPublished(String remoteUserID, List<QNRemoteTrack> trackList);
        void onUserUnpublished(String remoteUserID, List<QNRemoteTrack> trackList);
        void onSubscribed(String remoteUserID, List<QNRemoteAudioTrack> remoteAudioTracks, List<QNRemoteVideoTrack> remoteVideoTracks);
        void onMessageReceived(QNCustomMessage message);
    }

    setRTCClientListener()
    subscribe(tracks)
    unsubscribe(tracks)
    getRemoteUsers()
}
```

### Tracks
```c++
Track {
  isAudio()
  isVideo()
  getUserId()
}

LocalTrack extends Track {

}

LocalVideoTrack extends LocalTrack {
  addRenderer(VideoRenderer)
  int setEncoderConfiguration(const VideoEncoderConfiguration& config)
  int enableSimulcastStream(bool enabled, const SimulcastStreamConfig& config)
  void setVideoFrameListener(QNVideoFrameListener listener)
}

LocalAudioTrack extends LocalTrack {
    void setFrameListener(QNAudioFrameListener listener)
}

RemoteTrack extends Track {
    bool isSubscribed()
}

RemoteVideoTrack extends RemoteTrack {
    void setFrameListener(QNVideoFrameListener listener)
}

RemoteAudioTrack extends RemoteTrack {
    void setFrameListener(QNAudioFrameListener listener)
}
```

### media plugin
```c++
AudioFrame {}
EncodedAudioFrame {}
VideoFrame {}
EncodedVideoFrame {}
MediaPacket{}

// Sender
AduioPcmFrameSender {
	sendAudioPcmFrame(AudioFrame)
}

AudioEncodedFrameSender {
	sendAudioEncodedFrame(EncodedAudioFrame)
}

VideoFrameSender {
	sendVideoFrame(VideoFrame)
}

VideoEncodedFrameSender {
	sendVideoEncodedFrame(EncodedVideoFrame)
}

MediaPacetSender {
	sendMediaPacket(MediaPacket)
}

// Source
ScreenCapturer {
    int initWithDisplayId(view_t displayId, const Rectangle& regionRect)
    int initWithScreenRect(const Rectangle& screenRect,
                                 const Rectangle& regionRect)
}

CameraCapturer {
    CameraCaptureListener {
        void onError(int errorCode, String description);
    }

	// mobile only
	switchCamera()
	setCameraZoom(float zoomValue)
	setCameraFocus(float x, float y)
	// pc only
	initWithDeviceId(char* deviceId)
	setCameraListener(CameraCaptureListener listener)
	// ...
}

// Filter
AudioFilter {
	processAudioFrame(AudioFrame inAudioFrame, AudioFrame adaptedFrame)
}

VideoFilter {
	processVideoFrame(VideoFrame)
}

// Sink & Renderer
AudioSink {
	onAudioFrame(AudioFrame)
}

VideoSink {
	onVideoFrame(VideoFrame)
}

VideoRenderer extends VideoSink {
	RenderMode {}
	setRenderMode(RenderMode renderMode)
	setMirror(bool mirror)
	setView(void* view)
	unsetView()
}

MediaSourcePlugin {
	AduioPcmFrameSender createAudioPcmSender()
	AudioEncodedFrameSender createAudioEncodedFrameSender()
	VideoFrameSender createVideoFrameSender()
	VideoEncodedFrameSender createVideoEncodedFrameSender()
	MediaPacketSender createMediaPacketSender()
}

MediaFilterPlugin {
    AudioFilter createAudioFilter()
    VideoFilter createVideoFilter()
}

MediaSinkPlugin {
	VideoRenderer createVideoRenderer()
    AudioSink createAudioSink(char* name, char* vendor)
}
```
