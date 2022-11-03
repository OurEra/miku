
## 整体结构

```cpp
IAgoraService {
	IRtcConnection {
		LocalUser {
            ILocalUserObserver {
                onUserVideoTrackSubscribed(IRemoteVideoTrack)
            }

            publishVideo(ILocalVideoTrack videoTrack)
            ...

            subscribeVideo(user_id_t userId)
            ...

            registerLocalUserObserver(ILocalUserObserver)
		}
	}

	IRtmpConnection {
		RtmpLocalUser {
            publishVideo(ILocalVideoTrack videoTrack)
            ...
		}
	}

    IMediaNodeFactory {
        ICameraCapturer createCameraCapturer()
        ...

        IVideoFilter createVideoFilter()
        ...

        IVideoFrameSender createVideoFrameSender()
        ...
    }

    createCameraVideoTrack(ICameraCapturer)
    ...

    IRtcConnection createRtcConnection()
    ...
}
```

### IAgoraService

```cpp
// 全局服务类以及配置类
IAgoraService {
	// connection
    IRtcConnection createRtcConnection(RtcConnectionConfiguration cfg )
    IRtmpConnection createRtmpConnection(RtmpConnectionConfiguration cfg)

	// audio track
	// build-in mic
    ILocalAudioTrack createLocalAudioTrack()
    ILocalAudioTrack createRecordingDeviceAudioTrack(IRecordingDeviceSource audioSource, bool enableAec)
    ILocalAudioTrack createCustomAudioTrack(IAudioPcmDataSender audioSource)
    ILocalAudioTrack createCustomAudioTrack(IAudioEncodedFrameSender audioSource)
    ILocalAudioTrack createCustomAudioTrack(IMediaPacketSender source)
    ILocalAudioTrack createMediaPlayerAudioTrack(IMediaPlayerSource audioSource)

	// video track
    ILocalVideoTrack createCameraVideoTrack(ICameraCapturer videoSource)
    ILocalVideoTrack createScreenVideoTrack(IScreenCapturer videoSource)
    ILocalVideoTrack createCustomVideoTrack(IVideoFrameSender videoSource)
    ILocalVideoTrack createCustomVideoTrack(IVideoEncodedImageSender videoSource)
    ILocalVideoTrack createCustomVideoTrack(IMediaPacketSender source)
    ILocalVideoTrack createMediaPlayerVideoTrack(IMediaPlayerSource videoSource)

	// media
    IAudioDeviceManager createAudioDeviceManagerComponent(IAudioDeviceManagerObserver *observer)
    IMediaNodeFactory createMediaNodeFactory()
	
	// 转推服务
    IRtmpStreamingService createRtmpStreamingService(IRtcConnection rtcConnection, const char* appId)
    // 跨房服务
    IMediaRelayService createMediaRelayService(IRtcConnection rtcConnection const char* appId)
    // 实时消息服务
    IRtmService createRtmService()
}
```

### Connection

```cpp
IRtcConnection {
	connect
	disconnect
	getLocalUser
	getRemoteUsers
	// ...
}

IRtmpConnection {
    RTMP_CONNECTION_STATE {
    	STATE_DISCONNECTED = 1,
    	STATE_CONNECTING = 2,
    	STATE_CONNECTED = 3,
    	STATE_RECONNECTING = 4,
    	STATE_FAILED = 5
    };
    RtmpConnectionInfo {
    	RTMP_CONNECTION_STATE state;
    };
    IRtmpConnectionObserver {
    	void onConnected(RtmpConnectionInfo connectionInfo)
    	void onDisconnected(RtmpConnectionInfo connectionInfo)
    	void onReconnecting(RtmpConnectionInfo connectionInfo)
    	void onConnectionLost(RtmpConnectionInfo connectionInfo)
    	void onConnectionFailure(RtmpConnectionInfo connectionInfo,
                                       RTMP_CONNECTION_ERROR errCode)
    }

	int connect(url)
	int disconnect()
	IRtmpLocalUser getRtmpLocalUser()
	int registerObserver(IRtmpConnectionObserver observer)
}
```

### User

```cpp
ILocalUser {
    void setUserRole(rtc::CLIENT_ROLE_TYPE role)
    int publishVideo(ILocalVideoTrack videoTrack)
    int subscribeVideo(user_id_t userId)

    registerLocalUserObserver(ILocalUserObserver* observer)

    ILocalUserObserver {
        enum STREAM_PUBLISH_STATE {
          PUB_STATE_IDLE = 0,
          PUB_STATE_NO_PUBLISHED = 1,
          PUB_STATE_PUBLISHING = 2,
          PUB_STATE_PUBLISHED = 3
        }

        enum STREAM_SUBSCRIBE_STATE {
          SUB_STATE_IDLE = 0,
          SUB_STATE_NO_SUBSCRIBED = 1,
          SUB_STATE_SUBSCRIBING = 2,
          SUB_STATE_SUBSCRIBED = 3
        }

        void onVideoTrackPublishSuccess(ILocalVideoTrack> videoTrack)
        void onUserVideoTrackSubscribed(user_id_t userId, IRemoteVideoTrack> videoTrack)
        void onVideoPublishStateChanged(const char* channel, STREAM_PUBLISH_STATE oldState, STREAM_PUBLISH_STATE newState)
        void onVideoSubscribeStateChanged(const char* channel, uid_t uid, STREAM_SUBSCRIBE_STATE oldState, STREAM_SUBSCRIBE_STATE newState)
    }
}


IRtmpLocalUser {
	setAudioStreamConfiguration
	setVideoStreamConfiguration
	publishAudio
	publishVideo
	// ...
}
```

### Track

```cpp
// Audio Track
IAudioTrack{
	adjustPlayoutVolume
	addAudioFilter
	removeAudioFilter
	// ...
}
ILocalAudioTrack : IAudioTrack{
	setEnabled
	getState
	getStats
	enableLocalPlayback
	enableEarMonitor
	// ...
}
IRemoteAudioTrack : IAudioTrack{
	getStatistics
	getState
	// ...
}

// Video Track
IVideoTrack {
	addVideoFilter
	addRenderer
	// ...
}
ILocalVideoTrack : IVideoTrack{
	setEnabled
	getState
	enableSimulcastStream
	// ...
}
IRemoteVideoTrack : IVideoTrack{
	getStatistics
	getTrackInfo
	// ...
}
```

### Media Node

```cpp
// Sender
IAudioPcmDataSender{
	sendAudioPcmData
}
IAudioEncodedFrameSender {
	sendEncodedAudioFrame
}
IVideoFrameSender {
	sendVideoFrame
}
IVideoEncodedImageSender {
	sendEncodedVideoImage
}
IMediaPacketSender {
	sendMediaPacket
}
IMediaControlPacketSender {
	sendPeerMediaControlPacket
	sendBroadcastMediaControlPacket
}
// Sink
IAudioSinkBase {
	onAudioFrame
}
IVideoSinkBase {
	setProperty
	onFrame
	// ...
}
IVideoRenderer : IVideoSinkBase {
	setRenderMode
	// ...
}
// Filter
IAudioFilterBase {
	adaptAudioFrame
}
IAudioFilter : IAudioFilterBase {
	setEnabled
	setProperty
	// ...
}
IVideoFilterBase {
	adaptVideoFrame
}
IVideoFilter : IVideoFilterBase {
	setEnabled
	setProperty
	// ...
}
IVideoBeautyFilter : IVideoFilter {
	setBeautyEffectOptions
}
// Receiver
IMediaPacketReceiver {
	onMediaPacketReceived
}
IMediaControlPacketReceiver {
	onMediaControlPacketReceived
}
// Tranceiver
IVideoFrameTransceiver {
	getTranscodingDelayMs
	addVideoTrack
	// ...
}
// Factory
IMediaNodeFactory {
	createAudioPcmDataSender
	createAudioEncodedFrameSender
	// ... 上面的各个 node 都有静态创建方法
}
```

### Misc

```cpp
rtc::media::base {
	AudioFrame{}
	VideoFrame{}

	VIDEO_PIXEL_FORMAT
	RENDER_MODE_TYPR
	// ...
}

agora::rtm {
	IChannel{
		join
		leave
		sendMessage
	}
	IRtmService{
		initialize
		release
		login
		logout
		sendMessageToPeer
		createChannel
	}
}

agora::rtc {
	IVideoFrame {
		enum Type
		enum Format
		width
		height
		textureId
		resize
		//...
	}
	
	// 管理跨房
	IMediaRelayService{
		startChannelMediaRelay
		updateChannelMediaRelay
		stopChannelMediaRelay
		// ...
	}
	
	// 管理 rtmp 转发
	IRtmpStreamingService{
		addPublishStreamUrl
		removePublishStreamUrl
		setLiveTranscoding
	}
	
	// 管理音频设备
	INGAudioDeviceManager{
		setMicrophoneVolume
		getSpeakerVolume
		// ...
	}
	
	// 自定义扩展
	IExtensionProvider {
		createAudioFilter
		createVideoFilter
		createVideoSink
		// ...
	}
	
	// Source
	IMediaPlayerSource{
		play;
		pause;
		stop;
		seek;
		// ...
	}
	ICameraCapturer{
		switchCamera
		setCameraZoom
		// ...
	}
	IScreenCapturer {
		initWithScreenRect
		// ...
	}
	IRemoteAudioMixerSource {
		addAudioTrack
		// ...
	}
	IVideoMixerSource {
		addVideoTrack
		setStreamLayout
		// ...
	}
}

```
