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
  import AVFoundation

  class PreviewView: UIView {
    // #1
    var videoPreviewLayer: AVCaptureVideoPreviewLayer {
      guard let layer = layer as? AVCaptureVideoPreviewLayer else {
        fatalError("Expected `AVCaptureVideoPreviewLayer` type for layer. Check PreviewView.layerClass implementation.")
      }
      return layer
    }
    // #2
    var session: AVCaptureSession? {
      get {
        return videoPreviewLayer.session
      }
      set {
        videoPreviewLayer.session = newValue
      }
    }

    // #3
    // MARK: UIView
    override class var layerClass: AnyClass {
      return AVCaptureVideoPreviewLayer.self
    }
  }
```
