---
layout: post
title: "Contemplating on making a real time dictionary"
date: 2022-09-26 4:48:49 +0900
categories: project
---

## Why

I like to read books. I am talking about reading a real, paperback book while lying in my bed. While reading, I often come across words I don't know. When I read articles with my mobile phone, I can just long-press the word right on the screen and look up its definition quite easily, but when I am reading an actual book, I can't do that. The best possible way for me to do is first, open google, and then type in the word by each character, and click search. This to me is just too long a process to continuously repeat while reading a book. This also breaks the flow of reading which sucks more than not knowing an exact meaning of the word. So, I plan to come up with a tool that can help me look up words while reading a book as easily as possible.

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

This is the bread and butter of our `Fast` implementation of vision framework. Let's look at it from the top:

#### #1

```swift
class ViewController: UIViewController {

  // MARK: - UI objects
  @IBOutlet weak var previewView: PreviewView! // #1
  @IBOutlet weak var cutoutView: UIView! // #2
  @IBOutlet weak var numberView: UILabel! // #3
  var maskLayer = CAShapeLayer() // #4
  var currentOrientation = UIDeviceOrientation.portrait // #5
...
```

This section is listing out `UI Objects` that will be used inside the `ViewController`.

1. An instance of `PreviewView`, meaning it is connected via the interface builder, or via `storyboard`.
1. An instance of `UIView` from the aforementioned `storyboard`
1. An instance of `UILabel`
1. An instance of `CAShapeLayer`. The purpose of `CAShapeLayer` according to Apple's documentation is: _A layer that draws a cubic Bezier spline in its coordinate space._ `cubic Bezier spline` is a type of a smooth curve. So basically, `CAShapeLayer` allows programmers to draw some nice shapes on top of it.
1. An instance to hold current device's orientation, which defaults to `portrait` right now

#### #2

```swift
// MARK: - Capture related objects
private let captureSession = AVCaptureSession() // #1
let captureSessionQueue = DispatchQueue(label: "com.example.apple-samplecode.CaptureSessionQueue") // #2
var captureDevice: AVCaptureDevice? // #3
var videoDataOutput = AVCaptureVideoDataOutput() // #4
let videoDataOutputQueue = DispatchQueue(label: "com.example.apple-samplecode.VideoDataOutputQueue") // #5
```

This section is related to making instances that are related to video capturing

1. An instance of `AVCaptureSession`. This session deals with capturing video stream from the input source and process them into programmable data within the app
1. `DispatchQueue` for capture session which allows parallel processing to not block a main thread.
1. An instance of `AVCaptureDevice`. An object that represents a hardware or virtual capture device like a camera or microphone.
1. An instance of `AVCaptureVideoDataOutput`. This variable holds captured data output from the camera.
1. `DispatchQueue` for video data output.

#### #3

```swift
// MARK: - Region of interest (ROI) and text orientation
var regionOfInterest = CGRect(x: 0, y: 0, width: 1, height: 1) // #1
var textOrientation = CGImagePropertyOrientation.up // #2
```

1. Region of video data output buffer that recognition should be run on. This part gets recalculated once the bounds of the preview layer are known. Currently defaults to a point.
1. Orientation of text to search for in the region of interest. Defaults to reading texts considering 0th row at top, 0th column on left. Simply put, regular orientation like a blank excel sheet.

#### #4

```swift
// MARK: - Coordinate transforms
var bufferAspectRatio: Double! // #1
var uiRotationTransform = CGAffineTransform.identity // #2
var bottomToTopTransform = CGAffineTransform(scaleX: 1, y: -1).translatedBy(x: 0, y: -1) // #3
var roiToGlobalTransform = CGAffineTransform.identity // #4

// Vision -> AVF coordinate transform.
var visionToAVFTransform = CGAffineTransform.identity // #5
```

1. An instance that stores `bufferAspectRatio` in `Double`. Aspect ratio meant in this context is expected width / height.
1. `uiRotationTransform` instance is used to keep track of the rotation of capturing device. When the device is rotated, this property gets updated and necessary calculation is made accordingly.
1. `bottomToTopTransform` is used to transform bottom-left coordinates to top-left, which is needed to handle rotated device. \* need further dissection of its usage
1. `roiToGlobalTransform` transforms coordinates in ROI to global coordinates (still normalized). \* need further dissection of its usage
1. `visionToAVFTransform` is used to transform full vision ROI to AVFoundation coordinate. \* need further dissection of its usage

Just by looking at the variables, we can't get a full grasp of what this `ViewController` does. Let's look at other functions one-by-one.

#### viewDidLoad()

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  previewView.session = captureSession // #1


  cutoutView.backgroundColor = UIColor.gray.withAlphaComponent(0.5) // #2
  maskLayer.backgroundColor = UIColor.clear.cgColor // #3
  maskLayer.fillRule = .evenOdd // #4
  cutoutView.layer.mask = maskLayer // #5

  // Starting the capture session is a blocking call. Perform setup using
  // a dedicated serial dispatch queue to prevent blocking the main thread.
  captureSessionQueue.async {
    self.setupCamera()

    // Calculate region of interest now that the camera is setup.
    DispatchQueue.main.async {
      // Figure out initial ROI.
      self.calculateRegionOfInterest()
    }
  }
}
```

`viewDidLoad()` gets called when the view is completely loaded from a `ViewController`. Swift programmers tend to put settings helper functions inside this part of the lifecycle.

1. first setup `previewView` by setting the instantiated `captureSession` to `previewView`'s `session`. So when we think about it visually, `previewView` is positioned on top of the `ViewController`'s view, and this line of code allows us to see the live preview that is being captured by the camera.
1. then we set a transparent gray colored `cutoutView` on top of the `previewView` so that ROI stands out in a clear view.
1. `maskLayer` is our region of interest which is set as clear color.
1. [fillRule](https://developer.apple.com/documentation/quartzcore/cashapelayerfillrule/) is set to `.evenOdd`. Which means that we use `even-odd winding rule` for this mask layer. So we can imagine that the mask layer will be clear view, emphasizing the ROI.

<img src="https://upload.wikimedia.org/wikipedia/commons/f/f8/Even-odd_and_non-zero_winding_fill_rules.png" width="300" height="300" style="display: block; margin: 0 auto"/>

<p style="text-align: center;">even-odd winding rule (left) none-zero winding rule (right)</p>

1. then we set the `maskLayer` to `cutoutView`'s masking layer.
