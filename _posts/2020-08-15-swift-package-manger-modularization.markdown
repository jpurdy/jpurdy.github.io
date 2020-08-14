---
layout: post
title: Modularization with Swift Package Manager 
date: 2020-08-01 07:07:07 +0300
description: Guide to distributing a GUI inside a Swift Package in Xcode 12
img: bbb_spm.png
fig-caption:
tags: [iOS]
published: false
---

Begining in Swift 5.3, the Swift Package Manager (SPM) gained the ability ([SE-0271](https://github.com/apple/swift-evolution/blob/master/proposals/0271-package-manager-resources.md)) to include and utilize resources. Adding things like plists, images, storyboards, and xibs became trivial.

In the context of this enhancment to SPM and Xcode 12 beta's support of Swift 5.3, it was a good time to explore the possibilites of using the SPM to modularize portions of an App's GUI. Fortunately I had a pretty decent hobby code base to ~mutilate~ experiment with; the code base to [Boom Boom Bros](https://itunes.apple.com/us/app/boom-boom-bros./id961455539).

So, zooming out a bit, here are some of the upsides and downsides to this sort of general modularization effort.

#### Upsides:
* Encourages decoupling of components. Boundaries are clearly defined.
* Purposely crafting the public interfaces for modules reduces the tendency for consumers of the module to rely (or care) about implementation details.  Can be easier to integrate.
* Seperation of concerns must be purposely considered when building a codebase made up of modules.
* Module versioning allows for ease of intergration.
* Has the potential to reduces build times.
* Introduces the option to test and develop the module in isolation (i.e. via a playground or seperate project)

#### Downsides:
* Project organization and workflow becomes significantly more complex as developers have to deal with multiple repos as well as different versions of those repos.
* The relationships between these modules can become complex.

A modularized approach can scale up in a more straighforward way as a team does, whereas only so many developers can pound on a monolith before they are consistently stepping on other's toes.  I do not have any data to back up this assertion other than to say I've witnessed a monolith reach it's breaking point and the subsequent dictation from on high to begin modularization of the code base.

*Note: In the context of the codebase I chose to experiment on, there is not a good argument for modularizing it. It's small and it's a one dev affair with no need to scale.*

Now the question is why use SPM instead of Cocoapods or Carthage?

### Upsides
* SPM is skating to where Apple is taking the puck.
* SPM creation and integration is built into Xcode (see the previous bullet point)
* Packages are not tied to Xcode project files and are simple to write.

### Downsides
* Bound to find snags as SPM is still a relatively new initiative.
* Doesn't support targets with mixed Objc and Swift source.
* Cocoapods is a more dominant, mature, battle hardened third party dependency manager.

If should be noted that choosing to utilize SPM does not preclude the use of Cocoapods/Carthage.  All of these can coexist if need be.

## Planning

Ok, so those are the "whys" of using modularization and SPM. Now let's get into how we are going to restructure the project.

The flow of the Boom Boom Bros app looks like the following:

**pic of game flow**

The goal of this exercise was to extract the screens cicled below, and then re-insert a similar flow as a package.

**pic of the screens we want to extract**

To aid in this endevour, I ended up migrating this.

**pic here of code monolith**

to this

**pic of monolith split out into modules**


### Workflow

Initially the plan was to develop the flows via a playground. This is technically doable, and here (<- Link to new post) is sort post on the nuts and bolts of setting this up.  What I quickly discovered is that you can't use breakpoint debuggin in a playground. You can't do this from the package or the playground using it. There may have been other issues, but this lack of debugging tools was a dealbreaker; there was no reason to explore this option further.

The workflow I ended adopting was to creat a small enivornment to developer the package within. Here's how you do this.

1. First step in this process
2. Second step in this process


Note that the project file is saved in the repo but is ignored when it comes time to compile the package as a library.

Some other notes about working with the Swift Package Manager:
1. Versioning of packages is done via git tags.

        git commit -m "Version Complete"
        git tag 1.0.0
        git push origin main --tags


### References

https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package



