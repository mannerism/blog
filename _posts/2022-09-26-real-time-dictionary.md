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

Using Apple's `Vision` framework will definitely make my life easy, but unfortunately I have iPhone 8+, [which isn't supported](https://nerdschalk.com/does-live-text-work-on-iphone-6-7-8-x-and-xs/). I should look into using other OCR frameworks like `Tesseract` or any other [open-source OCR](https://www.hitechnectar.com/blogs/open-source-ocr-tools/).

Let's see these options first and see what I can use.

## Tesseract
