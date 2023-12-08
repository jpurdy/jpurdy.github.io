---
layout: post
title: Upsert Issue with SwiftData's Attribute Schema Macro using the Unique Option 
date: 2023-12-08 07:07:07 +0300
description: Fixing SwiftData crashes attributes marked as unique.
img: 2023-12-08.jpg
fig-caption:
tags: [Swift, SwiftData]
---
I recently encountered a puzzling crash while using SwiftData. The result? A cryptic `Thread 1: EXC_BAD_INSTRUCTION (code=EXC_I386_INVOP, subcode=0x0)` error message. The catalyst? Updating a record in the DB and then (simply) reading it. To be clear, this couldn't be reproduced when the record was first created, only after it had been **updated** and then read.

My data model was very simple; it looked like so:

```swift
@Model
class Event {
    @Attribute(.unique)
    var id: Int

    var type: String
    var date: Date
        
    init(id: Int, type: String, date: Date) {
        self.id = id
        self.type = type
        self.date = date
    }
}

```

While experimenting, I found the catalyst of this crash to be calling `save()` on the `ModelContext`. `save()`, however, is not neccessary since [SwiftData will autosave an updated context](https://developer.apple.com/documentation/swiftdata/adding-and-editing-persistent-data-in-your-app).  I had missed the note on that one 😅. I'm surprised it was detrimental, though.

Removing the `.save()` fixed the issue. Alternatively, making the entity field marked with the `@Attribute(.unique)` as optional solved the issue as well. In my scenario, I was able to just remove the `.save()` call on the `ModelContext`.

Some sample code demonstrating the problem can be found below:

```swift
import SwiftUI
import SwiftData

struct ContentView: View {
    @State var readValue = "<none>"
    var body: some View {
        VStack(spacing: 15) {
            Button(action: {
                readValue = TestBed.crashMe()
            }, label: {
                Text("Tap Twice to Crash")
            })
            
            Button(action: {
                readValue = TestBed.worksFine()
            }, label: {
                Text("Will Never Crash")
            })
            
            Text("Read Value: \(readValue)")
        }
    }

    enum TestBed {
        @MainActor static func crashMe() -> String {
            let result = insertItem()

            // This save operation creates a crash if the item is read
            do { try result.context.save() } catch { print(error) }  
            return result.item.name
        }
        
        @MainActor static func worksFine() -> String {
            let result = insertItem()
            return result.item.name
        }
        
        @MainActor static func insertItem() -> (item: Item, context: ModelContext) {
            let container = try! ModelContainer(for: Item.self)
            let context = container.mainContext
            let item = Item(name: "Hello World")
            context.insert(item)
            return (item: item, context: context)
        }
    }
}

@Model
final class Item {
    @Attribute(.unique) var name: String

    init(name: String) {
        self.name = name
    }
}

```
