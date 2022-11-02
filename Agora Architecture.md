
## 整体结构

```cpp
IAgoraService {
	IRtcConnection {
		LocalUser {
			ILocalAudioTrack{Microphone}
			ILocalAudioTrack{PcmSender}
			ILOcalVideoTrack{Camera}
			ILOcalVideoTrack{VideoFrameSender}
			// ...
		}
		RemoteUsers {
			IRemoteAudioTrack
			IRemoteVideoTrack
		}
	}
	
	IRtmpConnection {
		LocalUser {
			ILocalAudioTrack{Microphone}
			ILOcalVideoTrack{Camera}
		}
	}
}
```

### IAgoraService

```cpp
// 全局服务类以及配置类
AudioEncoderConfiguration{}
AgoraServiceConfiguration{}
IAgoraService{
	createAgoraService
	initialize
	release
	setAudioSessionPreset
	// ...
	// connection
	createRtcConnection
	createRtmpConnection
	
	// audio track
	// build-in mic
	createLocalAudioTrack
	createCustomAudioTrack(IAudioPcmDataSender
						   | IRemoteAudioMixerSource
						   | IAudioEncodedFrameSender
						   | IMediaPacketSender)
	createMediaPlayerAudioTrack
	createRecordingDeviceAudioTrack
	
	// video track
	createCameraVideoTrack(ICameraCapturer)
	createScreenVideoTrack(videoSource)
	createMixedVideoTrack(IVideoMixerSource)
	createTranscodedVideoTrack(IVideoFrameTransceiver)
	createCustomVideoTrack(IVideoFrameSender
						   | IVideoEncodedImageSender
						   | IMediaPacketSender)
	createMediaPlayerVideoTrack
	
	// media
	createAudioDeviceManager
	createMediaNodeFactory
	
	// 其他服务
	createRtmpStreamingService
	createMediaRelayService
	createRtmService
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
	connect
	disconnect
	getRtmpLocalUser
	// ...
}
```

### User

```cpp
ILocalUser {
	setUserRole
	publishAudio(ILocalAudioTrack)
	publishVideo(ILocalVideoTrack)
	subscribeAudio(userId)
	subscribeVideo(userId, VideoSubscriptionOptions)
	// ...
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

agora {
	// 内存管理
	RefCounter{}
	RefCounterObject{}
	RefCountInterface{}
	agora_refptr{}
	// ...
}

agora::commons {
	log{};
	// ...
}

agora::base {
	// kv 工具类
	IAgoraParameter
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