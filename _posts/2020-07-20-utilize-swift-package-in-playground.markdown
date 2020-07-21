---
layout: post
title: Utilize a Swift Package inside a Swift Playground
date: 2020-07-20 07:07:07 +0300
description: Import a local swift package for use in a playground
img: 2020-07-20.png
fig-caption:
tags: [iOS]
---

New in Swift 5.3 is the ability to embed and use resources (plists, images, xibs, etc..) inside a Swift Package. Additionally Xcode 12 affords us the ability to drag packages into a playground and utilize them. In this tutorial we leverage both of these new features to **build a Swift Package that encompasses a GUI and then present that GUI within a Swift Playground**.


# Part 1: Building a Storyboard GUI in a Swift Package

### Create a new Swift Package :package: with Xcode 

1. In Xcode **File->New->Swift Package**.
2. In the **Save As** text box, name the project something. In this example I name my package **MyExampleGUI**
3. Xcode will build a skeleton package that looks something like below:

    ![Xcode 12 - Create New Swift Package]({{site.baseurl}}/assets/img/2020-07-20/01_xcode12_create_package.png)

4. Update the Package.swift so that iOS 13 (or whatever version makes sense) is required. Note the addition of the `platforms` field.

    ```
    // swift-tools-version:5.3
    // The swift-tools-version declares the minimum version of Swift required to build this package.
    
    import PackageDescription
    
    let package = Package(
        name: "MyExampleGUI",
        platforms: [.iOS(.v13)],
        products: [
            .library(
                name: "MyExampleGUI",
                targets: ["MyExampleGUI"]),
        ],
        dependencies: [ ],
        targets: [
            .target(
                name: "MyExampleGUI",
                dependencies: []),
            .testTarget(
                name: "MyExampleGUITests",
                dependencies: ["MyExampleGUI"]),
        ]
    )
    ```

5. Set the active scheme to an iPhone simulator.

    ![Xcode 12 - Set the Scheme to iOS]({{site.baseurl}}/assets/img/2020-07-20/02_xcode12-set-scheme.png)

### Generate a GUI from our package :package:

1. Create a new swift file in **Sources/MyExampleGUI/** named **SimpleFlowViewController.swift**. Replace the contents with the following:

    ```swift
    import UIKit
    
    class SimpleFlowViewController: UIViewController {
    
      @IBOutlet var myLabel: UILabel!
      @IBOutlet var myButton: UIButton!
      
      override func viewDidLoad() {
            super.viewDidLoad()
        }
        
      @IBAction func didTapMyButton(_ sender: Any) {
        myLabel.text = "did get tapped"
      }
    }
    ```
3. Create a new storyboard file in **Sources/MyExampleGUI/** named **SimpleFlow.Storyboard**. Add a View Controller to the storyboard. Set the **Storyboard ID** and **Class** of that View Controller to `SimpleFlowViewController`.

    ![Xcode 12 - Create New Swift Package]({{site.baseurl}}/assets/img/2020-07-20/03_xcode12_storyboard.png)

4. Update the package manifest to add this new storyboard resource. **Package.swift** should look similar to the following. Note the additional `resources` line.
    ```
    // swift-tools-version:5.3
    // The swift-tools-version declares the minimum version of Swift required to build this package.
    
    import PackageDescription
    
    let package = Package(
        name: "MyExampleGUI",
        platforms: [.iOS(.v13)],
        products: [
            .library(
                name: "MyExampleGUI",
                targets: ["MyExampleGUI"]),
        ],
        dependencies: [ ],
        targets: [
            .target(
                name: "MyExampleGUI",
                dependencies: [],
                resources: [.copy("SimpleFlow.storyboard")]),
            .testTarget(
                name: "MyExampleGUITests",
                dependencies: ["MyExampleGUI"]),
        ]
    )
    ```
5. Add a button and a label to the storyboard. 

6. Connect the button and label on the storyboard to the outlets in the **SimpleFlowViewController.swift** code. If you do this from the package, Xcode will crash. To complete this step do the following:
    1. Close Xcode and open the terminal.  Navigate to the root of the package. Create a project file from the swift package with the following command.

        ```shell
        $ swift package generate-xcodeproj
        generated: ./MyExampleGUI.xcodeproj
        ````
    2. Open the newly created **MyExampleGUI.xcodeproj**. Note: you may have to re-add the storyboard to the project (probably a beta bug).
    3. Select the Project in the navigator, and then select the **build settings**. Inside the **build settings** set the **Base SDK** to `iOS`
    4. Hook the button and label on the storyboard to the code by ctrl dragging to the SimpleFlowViewController.swift IBOutlets.  Ctrl drag from the button to the `didTapMyButton` IBAction method as well.

7. Create a function in **MyExampleGUI.swift** which returns a SimpleFlowViewController to the caller.
    ```swift
      import UIKit
      
      public struct MyExampleGUI {
        public static func fetchViewController() -> UIViewController {
          let sb = UIStoryboard(name: "SimpleFlow", bundle: Bundle.module)
          return sb.instantiateViewController(identifier: "SimpleFlowViewController")
        }
      }
    ```

8. Replace the automatically generated code in **MyExampleGUITests.swft** with the following (or just remove the file).

    ```swift
    import XCTest
    @testable import MyExampleGUI
    
    final class MyExampleGUITests: XCTestCase {
    
    }
    ````

# Part 2: Importing our local Swift Package into a Swift Storyboard

1. In Xcode create a new playground **File->New->Playground**. From the playground template options select **Single View**.
2. In the **Save As** text box, name the playground something. In this example I named my playground **MyPlayground.playground**
3. Open up finder, and drag the root folder of your Package into the project navigator. Make sure to place it at the same level as the playground, and not nested within the playground. Note the heiarchy below:

    ![Xcode 12 - Create a new Playground]({{site.baseurl}}/assets/img/2020-07-20/05_xcode12_import_swift_package.png)

4. Save the playground as a workspace. **File -> Save as Workspace**.
5. If it is not already, set the target to **MyExampleGUI > Any iOS Device**
6. From the project navigator select **MyPlayground**, and replace the text in the editor with the following:

    ```
    import UIKit
    import PlaygroundSupport
    import MyExampleGUI
    
    PlaygroundPage.current.liveView = MyExampleGUI.fetchViewController()
    
    ```

7. Run the playground to observe your GUI built inside your swift package being presented in a Swift Playground.

    ![Xcode 12 - Create a new Playground]({{site.baseurl}}/assets/img/2020-07-20/06_xcode12_complete.png)


## Notes
This is likely not an ideal workflow for developing Swift Packages, as the debugging tools are not functional.