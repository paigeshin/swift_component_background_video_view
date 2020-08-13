# swift_component_background_video_view

```swift



    func setUpVideoBackground(view: UIView, urlString: String? = nil, localFileName: String? = nil) {
        print("Video Background Started!")
        var videoPath: String?
        var videoURL: URL?
        if let urlString: String = urlString {
            print("\(TAG) - setUpVideoBackground(): Network Video Background SetUp")
            videoURL = URL(string: urlString)
        } else {
            print("\(TAG) - setUpVideoBackground(): Local Video Background SetUp")
            guard let localFileName: String = localFileName else {
                print("\(TAG) - setUpVideoBackground(): No Local Video Name is was provided")
                return
            }
            videoPath = Bundle.main.path(forResource: localFileName, ofType: "mp4")
            guard let unwrappedVideoPath: String = videoPath else {
                print("\(TAG) - setUpVideoBackground(): No Local Video Path is was found")
                return
            }
            videoURL = URL(fileURLWithPath: unwrappedVideoPath)
        }
        guard let url: URL = videoURL else {
            print("\(TAG) - setUpVideoBackground(): videoURL is not found! either for local or for network")
            return
        }
//        do {
//            try  AVAudioSession.sharedInstance().setCategory(AVAudioSession.Category.playback, options: [.allowAirPlay, .mixWithOthers])
//            try  AVAudioSession.sharedInstance().setActive(true)
//        } catch _ as NSError {
//            print("\(TAG) - setUpBackground(): Error happenened while setting audio session")
//        }
        let playerItem: AVPlayerItem = AVPlayerItem(url: url)
        playerItem.audioTimePitchAlgorithm = AVAudioTimePitchAlgorithm.lowQualityZeroLatency //로딩속도 줄이기
        backgroundVideoPlayer = AVPlayer(playerItem: playerItem)
        backgroundVideoPlayer?.rate = 0.50 //로딩속도 줄이기
        backgroundVideoPlayer?.automaticallyWaitsToMinimizeStalling = false //로딩속도 줄이기
        backgroundVideoPlayer?.playImmediately(atRate: 0) //로딩속도 줄이기
        playerLayer?.removeFromSuperlayer()
        playerLayer = AVPlayerLayer(player: backgroundVideoPlayer)
        playerLayer?.frame = view.frame
        playerLayer?.videoGravity = AVLayerVideoGravity.resizeAspectFill //full size
        guard let playerLayer: AVPlayerLayer = playerLayer else {
            print("\(TAG) - setUpVideoBackground(): playerLayer is not initialized!")
            backgroundVideoPlayer?.isMuted = true
            backgroundVideoPlayer = nil
            return
        }
        view.layer.insertSublayer(playerLayer, at: 1)
        backgroundVideoPlayer?.play()
        /***  Debugging, 2020.08.10 ***/
        //이 코드 때문에 사운드 믹스가 안되었음. 자꾸 sound interrupting 현상이 벌어짐
        //이유는 간단함. NSNotification.Name.AVPlayerItemDidPlayToEndTime은 모든 AVPlayer의 event를 관찰하는 애들임.
        //AVQueuePlayer가 끝날 때마다, background audio player도 영향을 받게 되어있다.
        //object를 지정해줘야한다. object는 item이여야 한다.
        NotificationCenter.default.addObserver(forName: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: backgroundVideoPlayer?.currentItem, queue: .main) { [weak self] notification in
            self?.backgroundVideoPlayer?.seek(to: CMTime.zero)
            self?.backgroundVideoPlayer?.play()
        }
    }

```
