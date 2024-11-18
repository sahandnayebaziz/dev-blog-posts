---
title: "Syntax Highlighting SwiftUI Code with Swift Syntax"
slug: syntax-highlighting-swiftui-with-swift-syntax
image: https://assets.sahandnayebaziz.org/swift-syntax/running-on-mac.png
date: 2024-11-18
category: swift
---

I recently updated [DetailsPro](https://detailspro.app) to include syntax highlighting in the "Copy Code" section. This was a long-standing request I've had since I first released DetailsPro, which I kept putting off because I thought it was going to be too complex to implement.

Turns out... it's not! So I thought I'd write up a post about how I used [@swiftlang/swift-syntax](https://swiftpackageindex.com/swiftlang/swift-syntax) to natively parse and syntax-highlight the SwiftUI code that DetailsPro generates.

![Designers and developers use DetailsPro to create simple designs like the one seen here and often export or copy code straight into Xcode.](https://assets.sahandnayebaziz.org/swift-syntax/end-goal.jpg)

Users create SwiftUI designs in DetailsPro by arranging and styling SwiftUI views to their liking. At any point, they can copy the SwiftUI code that represents their design. My end goal was to dispay this code in DetailsPro with the same light and dark color schemes as Xcode. Ideally, I wanted a solution that was native Swift, reliable, and worked instantly. It also has to work on iOS, macOS, and visionOS where the DetailsPro editor runs.

Enter: swift-syntax.

## HOW SWIFT-SYNTAX WORKS

The good news is that swift-syntax is quite powerful. The bad news is (also) that swift-syntax is quite powerful. At a high level, this library is able to take Swift code you give it and turn it into an abstract representation that can be traversed and manipulated.

So, we can use it for syntax highlighting by using just two of its many abilities: parsing and traversal.

Swift-syntax will take valid code you give it and very quickly identify what is basically a super-nested tree of where things are. For example, something simple like "import SwiftUI" is identified as an `ImportDecl`, an strongly-typed identifier built-in to swift-syntax, and then within that strongly typed `ImportDecl`, you can access the part that says "import" and the part that says "SwiftUI". You can see the text contents, the range numbers, and more.

![Just this VStack init has many key ranges that are identified and broken down.](https://assets.sahandnayebaziz.org/swift-syntax/highlighting-breakdown.jpg)

There's a great resource made by Kishikawa Katsumi at [swift-ast-explorer.com](https://swift-ast-explorer.com) that lets you visualize what is happening. I definitely recommend pasting in your code and using this to understand how swift-syntax identifies any particular range of a given input.

![Even with just a short and simple file, you get a long abstract tree representation to look at.](https://assets.sahandnayebaziz.org/swift-syntax/ast-explorer.png)

## HOW I CUSTOMIZED SWIFT-SYNTAX FOR DETAILSPRO

Swift-syntax lets you create your own "SyntaxVisitor" you can use to go through parsed code and do something whenever you're at some part of code that you care about. For example, you can stop whenever you're at a string, a function, a colon, and other landmarks in Swift code that are nodes in the tree. Every Syntax node has properties you can access like text content, child nodes, and most important to us, the range of this node in the original code string.

To start, I needed a function that would take a string of code as input and output a string with attributes that I could directly display in my UI.

For my use case, I created a SyntaxVisitor that would stop at the kinds of nodes that I cared about. In my case, it was only the types of nodes that appear in simple SwiftUI view declarations. Then, as my visitor encountered one of these nodes, I used the range to add an attribute to my AttributedString. 


```swift
import SwiftParser
import SwiftSyntax
import SwiftUI
import UIKit

struct SwiftParser {
    static func makeHighlighted(for code: String) -> NSAttributedString {
        let attributedString = NSMutableAttributedString(string: code)

        let sourceFile = Parser.parse(source: attributedString.string)
        let visitor = MyVisitor(attributedString: attributedString)
        visitor.walk(sourceFile)

        return attributedString
    }
}

class MyVisitor: SyntaxVisitor {
    let attributedString: NSMutableAttributedString

    init(attributedString: NSMutableAttributedString) {
        self.attributedString = attributedString
        super.init(viewMode: .all)
    }

    // Override for the types of nodes you care about
    override func visit(_ node: LabeledExprSyntax) -> SyntaxVisitorContinueKind
    {
        if let trailingComma = node.trailingComma {
            highlight(syntax: trailingComma, color: .label)
        }
        if let colon = node.colon {
            highlight(syntax: colon, color: .label)
        }
        if let label = node.label {
            highlight(syntax: label, color: .codeOtherTypes)
        }
        return .visitChildren
    }

    // Override this general function for visiting common smaller pieces of code
    override func visit(_ token: TokenSyntax) -> SyntaxVisitorContinueKind {
        if token.leadingTrivia.contains(where: \.isWhitespace) {
            highlight(
                startPosition: token.position,
                endPosition: token.positionAfterSkippingLeadingTrivia,
                color: .gray)
        }
        if token.trailingTrivia.contains(where: \.isWhitespace) {
            highlight(
                startPosition: token.endPositionBeforeTrailingTrivia,
                endPosition: token.endPosition, color: .gray)
        }

        switch token.tokenKind {
        case .binaryOperator(_):
            highlight(syntax: token, color: .label)
        case .stringQuote:
            highlight(syntax: token, color: .codeString)
        case .stringSegment(_):
            highlight(syntax: token, color: .codeString)
        case .integerLiteral(_), .floatLiteral(_):
            highlight(syntax: token, color: .codeNumbers)
        case .leftParen,
            .rightParen,
            .period:
            highlight(syntax: token, color: .label)
        case .leftBrace,
            .rightBrace,
            .leftSquare,
            .rightSquare:
            highlight(syntax: token, color: .label)
        default:
            break
        }

        return .skipChildren
    }

    // I created multiple highlight functions to act as convenience methods
    private func highlight(syntax: SyntaxProtocol, color: UIColor) {
        highlight(
            startPosition: syntax.positionAfterSkippingLeadingTrivia,
            endPosition: syntax.endPositionBeforeTrailingTrivia,
            color: color)
    }

    private func highlight(
        startPosition: AbsolutePosition, endPosition: AbsolutePosition,
        color: UIColor
    ) {
        let code = attributedString.string

        let tokenStart = code.utf8.index(
            code.utf8.startIndex, offsetBy: startPosition.utf8Offset)
        let tokenEnd = code.utf8.index(
            code.utf8.startIndex, offsetBy: endPosition.utf8Offset)
        let tokenRange: Range<String.Index> = tokenStart..<tokenEnd

        let nsRange = NSRange(tokenRange, in: code)
        highlight(color: color, range: nsRange)
    }

    // This is the main highlight method that does the highlighting
    private func highlight(color: UIColor, range: NSRange) {
        let font = UIFont.monospacedSystemFont(ofSize: 12, weight: .regular)
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.lineSpacing = 6
        paragraphStyle.lineBreakMode = .byCharWrapping

        attributedString.addAttributes(
            [
                .font: font,
                .foregroundColor: color,
                .paragraphStyle: paragraphStyle,
            ], range: range)
    }
}
```

## Tips I Wish I Knew

First, you'll encounter a concept in called "trivia" which is the name swift-syntax gives whitespaces, newlines, tabs, and comments. It took me a while to figure out that through trivia is how you can highlight something like a line comment. And, I needed to make sure I was formatting the invisible whitespace between nodes so that my code still displayed proper spacing. 

Second, the SyntaxVisitor functions expect a return value like the ".skipChildren" and ".visitChildren" you see above. It took me a while to figure out exactly what was going on here, and basically what you're doing is telling the visitor whether or not it needs to go deeper after whatever you've identified. In some situations, you don't need to, for example if you've already identified an "import SwiftUI" statement and highlighted the appropriate parts, there's likely nothing more in there. With other more general nodes you'll visit, you'll likely want to keep going.

## The End Results

Today, syntax highlighting is running great in DetailsPro and I'm happy to have added a what is surely a reliable dependency that will enjoy continued support by the Swift community. 

![DetailsPro now shows live-updating, syntax-highlighted SwiftUI code when users select any part of their design.](https://assets.sahandnayebaziz.org/swift-syntax/running-on-mac.png)