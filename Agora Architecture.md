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
IRTCConnection {
    IRtcConnectionObserver{
        onConnected(TConnectionInfo connectionInfo)
        onDisconnected(TConnectionInfo connectionInfo)
        onConnecting(TConnectionInfo connectionInfo)
        onReconnecting(TConnectionInfo connectionInfo).
        onReconnected(TConnectionInfo connectionInfo)
        onConnectionLost(TConnectionInfo connectionInfo) 
        onConnectionFailure(TConnectionInfo connectionInfo)
        onUserJoined(user_id_t userId)
        onUserLeft(user_id_t userId, USER_OFFLINE_REASON_TYPE reason)
        onTransportStats(RtcStats stats);
        onChangeRoleSuccess(CLIENT_ROLE_TYPE oldRole, CLIENT_ROLE_TYPE newRole)
        onChangeRoleFailure(CLIENT_ROLE_CHANGE_FAILED_REASON reason, CLIENT_ROLE_TYPE currentRole) 
        onUserNetworkQuality(user_id_t userId, QUALITY_TYPE txQuality, QUALITY_TYPE rxQuality)
        onChannelMediaRelayStateChanged(int state, int code)
    }
    
    TConnectionInfo {
        conn_id_t id;
        util::AString channelId;
        CONNECTION_STATE_TYPE state;
        util::AString localUserId;
        util::AString connectionIp;
    }
    
    CONNECTION_STATE_TYPE {
        CONNECTION_STATE_DISCONNECTED = 1,
        CONNECTION_STATE_CONNECTING = 2,
        CONNECTION_STATE_CONNECTED = 3,
        CONNECTION_STATE_RECONNECTING = 4,
        CONNECTION_STATE_FAILED = 5,
    }

    connect(const char* token, const char* channelId, user_id_t userId)
    disconnect() 
    ILocalUser getLocalUser()
    getRemoteUsers(UserList& users)
    registerObserver(IRtcConnectionObserver* observer)
    unregisterObserver(IRtcConnectionObserver* observer)
    registerNetworkObserver(INetworkObserver* observer)
    unregisterNetworkObserver(INetworkObserver* observer)
    RtcStats getTransportStats()
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

    void setUserRole(rtc::CLIENT_ROLE_TYPE role)
    int publishVideo(ILocalVideoTrack videoTrack)
    int subscribeVideo(user_id_t userId)

    registerLocalUserObserver(ILocalUserObserver* observer)
    int registerAudioFrameObserver(agora::media::IAudioFrameObserver * observer)
}

IRtmpLocalUser {
    RtmpStreamingAudioConfiguration {
        int sampleRateHz
        int bytesPerSample
        int numberOfChannels
        int bitrate
    }
    RtmpStreamingVideoConfiguration {
        int width
        int height
        int framerate
        int bitrate
        int maxBitrate
        int minBitrate
        ORIENTATION_MODE orientationMode
    }
    PublishAudioError {
        PUBLISH_AUDIO_ERR_OK
        PUBLISH_AUDIO_ERR_FAILED
    }
    PublishVideoError {
        PUBLISH_VIDEO_ERR_OK
        PUBLISH_VIDEO_ERR_FAILED
    }
    VideoBitrateAdjustType {
        None
        Increasing
        Decreasing
    }
    IRtmpLocalUserObserver {
        void onAudioTrackPublishSuccess(ILocalAudioTrack audioTrack)
        void onAudioTrackPublicationFailure(ILocalAudioTrack audioTrack, PublishAudioError error)
        void onVideoTrackPublishSuccess(ILocalVideoTrack videoTrack)
        void onVideoTrackPublicationFailure(ILocalVideoTrack videoTrack, PublishVideoError error)
    }
    
    int setAudioStreamConfiguration(RtmpStreamingAudioConfiguration config)
    int setVideoStreamConfiguration(RtmpStreamingVideoConfiguration config)
    
    void adjustVideoBitrate(VideoBitrateAdjustType type)
    
    int publishAudio(ILocalAudioTrack audioTrack)
    int publishVideo(ILocalVideoTrack videoTrack)
    
    int registerRtmpUserObserver(IRtmpLocalUserObserver observer)
    int registerAudioFrameObserver(IAudioFrameObserver observer)
    int registerVideoFrameObserver(IVideoFrameObserver observer)
}
```

### Track

```cpp

// Audio Track
IAudioTrack {
    int adjustPlayoutVolume(int volume)
    bool addAudioFilter(IAudioFilter filter, AudioFilterPosition position)
    bool removeAudioFilter(IAudioFilter filter)
    bool addAudioSink(IAudioSinkBase sink, AudioSinkWants wants)
    bool removeAudioSink(IAudioSinkBase sink)
}

ILocalAudioTrack : IAudioTrack{
    struct LocalAudioTrackStats {
      uint32_t source_id;
      uint32_t missed_audio_frames;
      uint32_t sent_audio_frames;
      uint32_t pushed_audio_frames;
      uint32_t dropped_audio_frames;
      ...
    };

    LocalAudioTrackStats GetStats()
    int adjustPublishVolume(int volume)
    int enableEarMonitor(bool enable, bool includeAudioFilter)
}

IRemoteAudioTrack : IAudioTrack{
    enum REMOTE_AUDIO_STATE {
      REMOTE_AUDIO_STATE_STOPPED = 0,  // Default state, audio is started or remote user disabled/muted audio stream
      REMOTE_AUDIO_STATE_STARTING = 1,  // The first audio frame packet has been received
      REMOTE_AUDIO_STATE_DECODING = 2,  // The first remote audio frame has been decoded or fronzen state ends
    
      REMOTE_AUDIO_STATE_FROZEN = 3,    // Remote audio is frozen, probably due to network issue
      REMOTE_AUDIO_STATE_FAILED = 4,    // Remote audio play failed
    };

    struct RemoteAudioTrackStats {
      uid_t uid;
      int quality;
      uint32_t jitter_buffer_delay;
      int audio_loss_rate;
      int received_bitrate;
      int total_frozen_time;
      int64_t received_bytes;
      ...
    }

    bool getStatistics(RemoteAudioTrackStats& stats)
    REMOTE_AUDIO_STATE getState()
    int registerMediaPacketReceiver(IMediaPacketReceiver* packetReceiver)
}

// Video Track
IVideoTrack {
    enum VIDEO_MODULE_POSITION {
      POSITION_POST_CAPTURER = 1 << 0,
      POSITION_PRE_RENDERER = 1 << 1,
      POSITION_PRE_ENCODER = 1 << 2,
      POSITION_POST_FILTERS = 1 << 3,
    };

    bool addVideoFilter(IVideoFilter filter, VIDEO_MODULE_POSITION position)
    bool addRenderer(VideoSinkBase>videoRenderer, VIDEO_MODULE_POSITION position)
}

ILocalVideoTrack : IVideoTrack{
    struct VideoEncoderConfiguration {
        VIDEO_CODEC_TYPE codecType;
        VideoDimensions dimensions;
        int frameRate;
        int bitrate;  
        int minBitrate;
        ORIENTATION_MODE orientationMode;
        DEGRADATION_PREFERENCE degradationPreference;
        VIDEO_MIRROR_MODE_TYPE mirrorMode;
    }

    struct SimulcastStreamConfig {
        VideoDimensions dimensions;
        int bitrate;
        int framerate;
    };

    enum LOCAL_VIDEO_STREAM_STATE {
      LOCAL_VIDEO_STREAM_STATE_STOPPED = 0,
      LOCAL_VIDEO_STREAM_STATE_CAPTURING = 1,
      LOCAL_VIDEO_STREAM_STATE_ENCODING = 2,
      LOCAL_VIDEO_STREAM_STATE_FAILED = 3
    };

    struct LocalVideoTrackStats {
        uint64_t number_of_streams 
        uint32_t frames_encoded 
        int input_frame_rate 
        int encode_frame_rate 
        int render_frame_rate;
        int target_media_bitrate_bps 
        int media_bitrate_bps 
        int total_bitrate_bps  // Include FEC
        int width 
        int height
        uint32_t encoder_type
    };

    int setVideoEncoderConfiguration(const VideoEncoderConfiguration& config)
    int enableSimulcastStream(bool enabled, const SimulcastStreamConfig& config)
    LOCAL_VIDEO_STREAM_STATE getState()
    bool getStatistics(LocalVideoTrackStats& stats)
}

IRemoteVideoTrack : IVideoTrack {
    enum REMOTE_VIDEO_STATE {
        REMOTE_VIDEO_STATE_STOPPED = 0,
        REMOTE_VIDEO_STATE_STARTING = 1,
        REMOTE_VIDEO_STATE_DECODING = 2,
        REMOTE_VIDEO_STATE_FROZEN = 3,
        REMOTE_VIDEO_STATE_FAILED = 4,
    };

    struct VideoTrackInfo {
        uid_t ownerUid;
        track_id_t trackId;
        conn_id_t connectionId;
        VIDEO_CODEC_TYPE codecType;
        // camera/screen
        VIDEO_SOURCE_TYPE sourceType;
    };

    struct RemoteVideoTrackStats {
        uid_t uid;
        int delay;
        int width;
        int height;
        int receivedBitrate;
        int decoderOutputFrameRate;
        int rendererOutputFrameRate;
        int frameLossRate;
        int packetLossRate;
        int totalFrozenTime;
        int frozenRate;
    };

    REMOTE_VIDEO_STATE getState()
    bool getTrackInfo(VideoTrackInfo& info)
    bool getStatistics(RemoteVideoTrackStats& stats)
    int registerVideoEncodedImageReceiver(IVideoEncodedImageReceiver* videoReceiver)
    int registerMediaPacketReceiver(IMediaPacketReceiver* videoReceiver)
}
```

### Media Node
```c++
// Sender
IAudioPcmDataSender{
    int sendAudioPcmData(void* audio_data, 
                         uint32_t capture_timestamp, 
                         size_t samples_per_channel, 
                         size_t bytes_per_sample, 
                         size_t number_of_channels, 
                         uint32_t sample_rate)
}

IAudioEncodedFrameSender {
    bool sendEncodedAudioFrame(uint8_t* payload_data, 
                               size_t payload_size, 
                               EncodedAudioFrameInfo audioFrameInfo)
}

IVideoFrameSender {
    int sendVideoFrame(ExternalVideoFrame videoFrame)
}

IVideoEncodedImageSender {
    bool sendEncodedVideoImage(uint8_t* imageBuffer, 
                               size_t length, 
                               EncodedVideoFrameInfo videoEncodedFrameInfo)
}

IMediaPacketSender {
    int sendMediaPacket(uint8_t *packet, 
                        size_t length, 
                        PacketOptions options)
}

IMediaControlPacketSender {
    int sendPeerMediaControlPacket(user_id_t userId,
                                   uint8_t *packet,
                                   size_t length)
    sendBroadcastMediaControlPacket(uint8_t *packet, size_t length)
}

// Sink
IAudioSinkBase {
    bool onAudioFrame(AudioPcmFrame audioFrame)
}

IVideoSinkBase {
    int setProperty(char* key, void* buf, int buf_size)
    int onFrame(VideoFrame videoFrame)
}

IVideoRenderer : IVideoSinkBase {
    setRenderMode
}

// Filter
IAudioFilterBase {
    bool adaptAudioFrame(AudioPcmFrame inAudioFrame, AudioPcmFrame adaptedFrame)
}

IAudioFilter : IAudioFilterBase {
    void setEnabled(bool enable)
    int setProperty(char* key, void* buf, int buf_size)
}

IVideoFilterBase {
    bool adaptVideoFrame(VideoFrame capturedFrame, VideoFrame adaptedFrame)
}

IVideoFilter : IVideoFilterBase {
    void setEnabled(bool enable)
    int setProperty(char* key, void* buf, size_t buf_size)
}

IVideoBeautyFilter : IVideoFilter {
    BeautyOptions {
        LIGHTENING_CONTRAST_LEVEL {
            LIGHTENING_CONTRAST_LOW = 0,
            LIGHTENING_CONTRAST_NORMAL,
            LIGHTENING_CONTRAST_HIGH
        };
        LIGHTENING_CONTRAST_LEVEL lighteningContrastLevel;
        float lighteningLevel;
        float smoothnessLevel;
        float rednessLevel;
    };
    int setBeautyEffectOptions(bool enabled, BeautyOptions options)
}

// Receiver
IMediaPacketReceiver {
    bool onMediaPacketReceived(uint8_t *packet, 
                               size_t length, 
                               PacketOptions options)
}

// Factory
IMediaNodeFactory {
    MEDIA_PLAYER_SOURCE_TYPE {
        MEDIA_PLAYER_SOURCE_DEFAULT,
        MEDIA_PLAYER_SOURCE_FULL_FEATURED,
        MEDIA_PLAYER_SOURCE_SIMPLE,
    };

    IAudioPcmDataSender createAudioPcmDataSender()
    IAudioEncodedFrameSender createAudioEncodedFrameSender()
    IVideoFrameSender createVideoFrameSender()
    IVideoEncodedImageSender createVideoEncodedImageSender()
    IMediaPacketSender createMediaPacketSender()
    IMediaPlayerSource createMediaPlayerSource(MEDIA_PLAYER_SOURCE_TYPE type)
    
    IAudioFilter createAudioFilter(char* name, char* vendor)
    IVideoFilter createVideoFilter(char* name, char* vendor)
    
    IVideoRenderer createVideoRenderer()
    IVideoSinkBase createVideoSink(char* name, char* vendor)

    ICameraCapturer createCameraCapturer()
    IScreenCapturer createScreenCapturer()
}
```


## Capture
```c++
ICameraCapturer {
    IDeviceInfo {
      void release()
      uint32_t NumberOfDevices() 
      int32_t GetDeviceName(uint32_t deviceNumber, deviceNameUTF8,
                                    uint32_t deviceNameLength, deviceUniqueIdUTF8,
                                    uint32_t deviceUniqueIdLength, productUniqueIdUTF8,
                                    uint32_t productUniqueIdLength) 
      int32_t NumberOfCapabilities(deviceUniqueIdUTF8)
      int32_t GetCapability(deviceUniqueIdUTF8,
                                    uint32_t deviceCapabilityNumber,
                                    VideoFormat& capability) 
    }

    enum CAMERA_SOURCE {
      CAMERA_BACK,
      CAMERA_FRONT,
    };

    VideoFormat {
      OPTIONAL_ENUM_SIZE_T {
        kMaxWidthInPixels = 3840,
        kMaxHeightInPixels = 2160,
        kMaxFps = 60,
      };
      int width; 
      int height;
      int fps;
    }

    ICameraCaptureObserver {
      void onCameraFocusAreaChanged(int imageWidth, int imageHeight, int x, int y) 
      void onFacePositionChanged(int imageWidth, int imageHeight, Rectangle vecRectangle,int vecDistance, int numFaces)
      void onCameraExposureAreaChanged(int x, int y, int width, int height)
      void onCameraStateChanged(CAMERA_STATE state, CAMERA_SOURCE source) 
    }

    IDeviceInfo createDeviceInfo()

    int setCameraSource(CAMERA_SOURCE source) 
    CAMERA_SOURCE getCameraSource()
    void switchCamera()
    bool isZoomSupported()

    int initWithDeviceId(deviceId)
    int initWithDeviceName(deviceName) 
    void setCaptureFormat(const VideoFormat& format) 
    VideoFormat getCaptureFormat()
    int registerCameraObserver(ICameraCaptureObserver observer) 
    int unregisterCameraObserver(ICameraCaptureObserver observer)
}

IScreenCapturer{
    int initWithDisplayId(uint32_t displayId, Rectangle regionRect)
    int updateScreenCaptureRegion(Rectangle regionRect)
    void setScreenOrientation(VIDEO_ORIENTATION orientation)
    void setFrameRate(int rate) 
}
```

### Misc
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
