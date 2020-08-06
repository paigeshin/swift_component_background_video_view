# swift_component_background_video_view

```swift

/** Set up Video Background **/
//arg1 - hosting view
//arg2 - network
//arg3 - local

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
    backgroundVideoPlayer =  AVPlayer(url: url)
    let playerLayer: AVPlayerLayer = AVPlayerLayer(player: backgroundVideoPlayer)
    playerLayer.frame = view.frame
    playerLayer.videoGravity = AVLayerVideoGravity.resizeAspectFill //full size
    view.layer.insertSublayer(playerLayer, at: 1)
    backgroundVideoPlayer?.play()
    NotificationCenter.default.addObserver(forName: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: nil, queue: .main) { notification in
        self.backgroundVideoPlayer?.seek(to: CMTime.zero)
        self.backgroundVideoPlayer?.play()
    }
}


```
