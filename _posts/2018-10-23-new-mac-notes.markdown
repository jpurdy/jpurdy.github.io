---
layout: post
title: New Mac Setup Notes
date: 2018-10-23 07:07:07 +0300
description: New Apple Mack Setup Notes
img: 2018_10_23-1.jpg
fig-caption:
tags: [iOS, Mac]
---
1. Install Xcode and the Xcode Command Line Tools.

1. Upgrade the shell to [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh).

   ```
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
   Open ~/.zshrc
   ```

   Change:  
   **ZSH_THEME="robbyrussell"**  
   To:  
   **ZSH_THEME=“gnzh”**

   *Note: zsh aliases git to ‘g’*
1. Setup Git Aliases

   Open **~/.gitconfig** (or create it if it doesn’t exist).  Add:
```
[alias]
aliases = !git config –list | grep ^alias\\. | cut -c 7- | grep -Ei –color \”$1\” “#”
co = checkout
st = status
adds = “!f() { git ls-files –modified | grep “\\.swift$” | xargs git add; }; f”
addm = “!f() { git ls-files –modified | grep “\\.m$” | xargs git add; }; f”
addh = “!f() { git ls-files –modified | grep “\\.h$” | xargs git add; }; f”
addsb = “!f() { git ls-files –modified | grep “\\.storyboard$” | xargs git add; }; f”
addproj = “!f() { git ls-files –modified | grep “\\.pbxproj$” | xargs git add; }; f”
appcode = “!git ls-files –modified | grep “\\.xcscheme$” | xargs git co”
l = log –name-status
```

1. Setup git ignore file list (Usually **~/.gitignore_global** but check via `git config --list --show-origin`)

   If no ignore files are present create one and link it to git:
   i.e. `git config --global core.excludesfile ~/.gitignore_global`

   Edit the file to include the following lines:

   ```
   *~
   .DS_Store

   ##Cocoapods
   Pods/*
   Podfile.lock

   ## User settings
   xcuserdata/

   ```

1. Install [Homebrew](https://brew.sh/)

   ```
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```

1. Install CocoaPods
   ```
   brew install ruby
   sudo gem install cocoapods
   pod setup
   pod repo update
   ```

1. Install Xcode Code Formatter
   * Objective-C

      ```
      brew install clang-format
      ```

      [Download](https://github.com/mapbox/XcodeClangFormat/releases/tag/v1.0.0) the Xcode code formatting plugin and open it. 
      Set the formatting of your choice within the tool. Mine (similar to a workplace who’s coding convention I liked) is here:
      ```
      AlignAfterOpenBracket: true
      AlignConsecutiveAssignments: false
      AlignConsecutiveDeclarations: false
      AlignEscapedNewlinesLeft: true
      AlignOperands: true
      AlignTrailingComments: true
      AllowAllParametersOfDeclarationOnNextLine: true
      AllowShortBlocksOnASingleLine: false
      AllowShortCaseLabelsOnASingleLine: false
      AllowShortFunctionsOnASingleLine: false
      AllowShortIfStatementsOnASingleLine: false
      AllowShortLoopsOnASingleLine: false
      AlwaysBreakBeforeMultilineStrings: false
      AlwaysBreakTemplateDeclarations: true
      BraceWrapping : {
        AfterClass: true,
        AfterControlStatement: false,
        AfterFunction: false,
        AfterObjCDeclaration: false,
        AfterStruct: true,
        AfterUnion: true,
        BeforeCatch: false,
        BeforeElse: false,
      }
      BinPackArguments: false
      BinPackParameters: false
      BreakBeforeBraces : Custom
      BreakBeforeTernaryOperators: false
      BreakConstructorInitializersBeforeComma: false
      BreakStringLiterals: true
      ColumnLimit: 0
      IndentWidth: 4
      IndentCaseLabels: true
      KeepEmptyLinesAtTheStartOfBlocks: false
      MaxEmptyLinesToKeep: 1
      ObjCBlockIndentWidth: 4
      ObjCSpaceAfterProperty: true
      ObjCSpaceBeforeProtocolList: true
      PointerAlignment: Right
      ReflowComments: true
      SortIncludes: false
      SpaceAfterCStyleCast: false
      SpaceBeforeAssignmentOperators: true
      SpaceBeforeParens: ControlStatements
      SpaceInEmptyParentheses: false
      SpacesBeforeTrailingComments: 1
      SpacesInAngles: false
      SpacesInCStyleCastParentheses: false
      SpacesInContainerLiterals: false
      SpacesInParentheses: false
      SpacesInSquareBrackets: false
      TabWidth: 4
      UseTab: Never
      ```
      Go to your system’s System Preferences → Extensions

      Verify that clang-format in the Xcode Source Editor section is checked.

      To setup a keyboard shortcut, open System Preferences, click Keyboard, and switch to the Shortcuts tab.

      On the left column, select App Shortcuts, then hit the + button. Select Xcode, enter Format Source Code, and define a shortcut. I use *option-command-l*.
