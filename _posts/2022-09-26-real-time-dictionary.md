---
layout: post
title: "Contemplating on making a real time dictionary"
date: 2022-09-26 4:48:49 +0900
categories: project
---

## Why

I like to read books. I am talking about reading a real, paperback book while lying in my bed. While reading, I often come across words I don't know. When I read articles with my mobile phone, I can just long-press the word right on the screen and look up its definition quite easily, but when I am reading an actual book, I can't do that. The best possible way for me to do is first, open google, and then type in the word by each character, and click search. This to me is just too long a process to continuously repeat it while reading a book. This also breaks the flow of reading which sucks more than not knowing an exact meaning of the word. So, I plan to come up with a tool that can help me look up words while reading a book as easily as possible.

## Possiblilty

For now, the best solution is using my mobile phone. Apple recently announced `live text` functionality. [HERE](https://support.apple.com/en-us/HT212630). This allows real-time recognition of text from photos. Gotcha is, according to Apple, `To use Live Text, you need an iPhone XS, iPhone XR, or later with iOS 15 or later.`

I thought about using different open-source softwares like [Tesseract](https://github.com/gali8/Tesseract-OCR-iOS) or [SwiftOCR](https://github.com/NMAC427/SwiftOCR), but boosting up the accuracy of `Tesseract` requires several layers of image processing and `SwiftOCR` is no longer maintained. `SwiftOCR` even recommends people to use Apple's `Vision` framework instead:

> Please use Apple's Vision framework instead of SwiftOCR. It is very fast, accurate and much less finicky.

So let's just not reinvent the wheel and go with the simplest solution. If this straight path won't get me the solution, I will have to dig up some additional, maybe a bit of hacky, ways to come up with a solution.

## Apple's Vision Framework

Let'a have a deeper look at what we can do with Vision Framework.

Video: [Apple WWDC Talk on Vision Framework](https://developer.apple.com/videos/play/wwdc2021/10041/).
Article: [Recognizing Text in Images](https://developer.apple.com/documentation/vision/reading_phone_numbers_in_real_time).

There are two types of text-recognition in the framework:

1. Fast
   > The fast path uses the framework’s character-detection capabilities to find individual characters, and then uses a small machine learning model to recognize individual characters and words. This approach is similar to traditional optical character recognition (OCR).
1. Accurate
   > The accurate path uses a neural network to find text in terms of strings and lines, and then performs further analysis to find individual words and sentences. This approach is much more in line with how humans read text.

### Fast

We'll checkout some of the sample codes for fast recognition provided by Apple: [Sample Code](https://developer.apple.com/documentation/vision/reading_phone_numbers_in_real_time)

This is the project structure:

```md
project
| AppDelegate.swift
| PreviewView.swift
| ViewController.swift
| VisionViewController.swift
| StringUtils.swift
```

We will go over each swift file and try to dissect their functionalities in depth. Again, this is the `fast` version of text-recognition and it's programmatic approach is fundamentally different from `accurate` version.

#### AppDelegate.swift

```swift
  import UIKit

  @UIApplicationMain
  class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
  }
```

it's just an entry point of the app. Not much is there to dissect.

#### PreviewView.swift

```swift
  import UIKit
  import AVFoundation // #1

  class PreviewView: UIView { // #2

    var videoPreviewLayer: AVCaptureVideoPreviewLayer { // #3
      guard let layer = layer as? AVCaptureVideoPreviewLayer else {
        fatalError("Expected `AVCaptureVideoPreviewLayer` type for layer. Check PreviewView.layerClass implementation.")
      }
      return layer
    }

    var session: AVCaptureSession? { // #4
      get {
        return videoPreviewLayer.session
      }
      set {
        videoPreviewLayer.session = newValue
      }
    }

    // MARK: UIView
    override class var layerClass: AnyClass { // #5
      return AVCaptureVideoPreviewLayer.self
    }
  }
```

1. import `AVFoundation` framework to use `audio` and `video` capabilities of the mobile phone
1. make a new class named `PreviewView` and inherit from `UIView` so that we can use all the `UIView` functionalities. On top of it we add `#3` ~ `#5` variables.
1. create a variable `videoPreviewLayer` of type `AVCaptureVideoPreviewLayer`. It is a computed property that only renders if the layer is of type `AVCaptureVideoPreviewLayer`.
1. create a variable `session` of type `AVCaptureSession` that returns `videoPreviewLayer`'s session when read, and write to `videoPreviewLayer`'s session when written.
1. Default `CALayer` class is overridden with `AVCaptureVideoPreviewLayer`'s class. [Reference](https://developer.apple.com/documentation/uikit/uiview/1622626-layerclass)

Basically, `PreviewView` class is creating a custom `UIView` class that uses `AVCaptureVideoPreviewLayer` as a main `layerClass`. [Reference](https://developer.apple.com/documentation/avfoundation/avcapturevideopreviewlayer)

#### ViewController.swift

```swift
// #1
import UIKit
import AVFoundation
import Vision

// #2
class ViewController: UIViewController {
  ...
}

// #3
// MARK: - AVCaptureVideoDataOutputSampleBufferDelegate
extension ViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
  ...
}

// #4
// MARK: - Utility extensions
extension AVCaptureVideoOrientation {
...
}
```

Let's begin dissecting `ViewController` file. First, a bird-eye view of the whole file:

1. We'll be using three frameworks: `UIKit`, `AVFoundation`, `Vision`
1. All the global variables and functions will be located inside the main body
1. `ViewController` extends `AVCaptureVideoDataOutputSampleBufferDelegate` what it does, according to the [Reference](https://developer.apple.com/documentation/avfoundation/avcapturevideodataoutputsamplebufferdelegate) is this:

   > This protocol defines an interface for delegates of an AVCaptureVideoDataOutput **object to receive captured video sample buffers and be notified of late sample buffers that were dropped.**

   In another words, this is where we receive captured video streams from the camera.

1. Utility function that handles video orientation. Doesn't know fully until we dissect the inside logic.

#### class ViewController: UIViewController {}

Let's start with the `ViewController` class:

```swift
class ViewController: UIViewController {

  // MARK: - UI objects
  @IBOutlet weak var previewView: PreviewView!
  @IBOutlet weak var cutoutView: UIView!
  @IBOutlet weak var numberView: UILabel!
  var maskLayer = CAShapeLayer()
  var currentOrientation = UIDeviceOrientation.portrait

  // MARK: - Capture related objects
  private let captureSession = AVCaptureSession()
  let captureSessionQueue = DispatchQueue(label: "com.example.apple-samplecode.CaptureSessionQueue")
  var captureDevice: AVCaptureDevice?
  var videoDataOutput = AVCaptureVideoDataOutput()
  let videoDataOutputQueue = DispatchQueue(label: "com.example.apple-samplecode.VideoDataOutputQueue")

  // MARK: - Region of interest (ROI) and text orientation
  var regionOfInterest = CGRect(x: 0, y: 0, width: 1, height: 1)
  var textOrientation = CGImagePropertyOrientation.up

  // MARK: - Coordinate transforms
  var bufferAspectRatio: Double!
  var uiRotationTransform = CGAffineTransform.identity
  var bottomToTopTransform = CGAffineTransform(scaleX: 1, y: -1).translatedBy(x: 0, y: -1)
  var roiToGlobalTransform = CGAffineTransform.identity

  // Vision -> AVF coordinate transform.
  var visionToAVFTransform = CGAffineTransform.identity

  // MARK: - View controller methods
  override func viewDidLoad() { ... }

  override func viewWillTransition(
    to size: CGSize,
    with coordinator: UIViewControllerTransitionCoordinator
  ) { ... }

  override func viewDidLayoutSubviews() { ... }

  // MARK: - Setup
  func calculateRegionOfInterest() { ... }
  func updateCutout() { ... }
  func setupOrientationAndTransform() { ... }
  func setupCamera() { ... }

  // MARK: - UI drawing and interaction
  func showString(string: String) { ... }
  @IBAction func handleTap(_ sender: UITapGestureRecognizer) { ... }
}
```

This is the bread and butter of our `Fast` implementation of vision framework.
