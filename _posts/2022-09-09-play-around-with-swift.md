---
layout: post
title: "Getting my hands dirty with Swift"
date: 2022-09-03 4:48:49 +0900
categories: Ideas
---

## Introduction

---

Introduce what you are going to explain in this article.

## View (or any other topic you want to write about)

```swift
  var a: Int = 1  // 1. describe what this line does in your own words
  var b = 2       // 2. describe what this line does in your own words
  var c: Int      // 3. describe what this line does in your own words
  c = 3           // 4. describe what this line does in your own words
```

write summary and overall goal of the codes written above

## Image

```swift
  import SwiftUI

  struct ContentView: View {
      var body: some View {
          VStack { // 1. explain this line
              Image(systemName: "globe")
                  .imageScale(.large)
                  .foregroundColor(.accentColor) // 2. explain this line
              Text("Hello, world!") // 3. explain this line
              Image(uiImage: #imageLiteral(resourceName: "test"))
                  .resizable()
                  .scaledToFill()
                  .frame(width: 300, height: 300)
                  .background(Color.blue)
                  .clipShape(Circle()) // 4. explain this line
          }
      }
  }

```

write summary and overall goal of the codes written above

## Shape

```swift

  Enter swift code here

```

write summary and overall goal of the codes written above

---

## Conclusion

wrap up what you have learned today. 4-5 sentences.
