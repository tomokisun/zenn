---
title: "Poker用のEntityをまとめたライブラリを作った"
emoji: "♠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift]
published: true
---

# はじめに

Poker関係のアプリを作る際に型を再利用可能にするためにライブラリ化したので、それについて解説します。

今回作ったライブラリは以下

https://github.com/tomoki69386/PokerHands

# Suit

Suitとはトランプのマークのことです。

ローカルルールでマークに強さが決まってる場合があるので一応その順番にしてます。

```Swift
public enum Suit {
    case spades
    case hearts
    case diamonds
    case clubs
}
```

# Number

Pokerでは数字に強さの順番が決まっています2が最弱でAが最強というルールなので、その順番で作っています。

```Swift
public enum Number {
    case two
    case three
    case four
    case five
    case six
    case seven
    case eight
    case nine
    case ten
    case eleven
    case twelve
    case thirteen
    case one
    
    public var string: String {
        switch self {
        case .two: return "2"
        case .three: return "3"
        case .four: return "4"
        case .five: return "5"
        case .six: return "6"
        case .seven: return "7"
        case .eight: return "8"
        case .nine: return "9"
        case .ten: return "T"
        case .eleven: return "J"
        case .twelve: return "Q"
        case .thirteen: return "K"
        case .one: return "A"
        }
    }
}
```

# Trump

Trumpには`suit`,`number`を持たせています。

```Swift
public struct Trump {
    public let suit: Suit
    public let number: Number
    
    public init(suit: Suit, number: Number) {
        self.suit = suit
        self.number = number
    }
}
```

あとは特に複雑なことはしておらず、`Comparable` の `<` の定義をしています

```Swift
extension Trump: Comparable {
    public static func < (lhs: Trump, rhs: Trump) -> Bool {
        return lhs.number < rhs.number
            && lhs.suit < rhs.suit
    }
}
```

# Hand

これはプレイヤーに配られるトランプ2枚の型です。

2枚のトランプが全く同じものになる可能性はないので、そうなった場合は例外を返すようにしています。

ここはinit?(){}でも良かったかなと思っています。

```Swift
public struct Hand {
    public let firstTrump: Trump
    public let secondTrump: Trump
    
    public init(first: Trump, second: Trump) throws {
        if first == second {
            throw NSError(domain: "poker.hands", code: 100, userInfo: nil)
        }
        
        self.firstTrump = first
        self.secondTrump = second
    }
}
```

あとはポケットペアやスーテッドと呼ばれる組み合わせか否かを判定するプロパティを生やしています。

```Swift
extension Hand {
    public var isSuited: Bool {
        return firstTrump.suit == secondTrump.suit
    }
    
    public var isPocketPair: Bool {
        return firstTrump.number == secondTrump.number
    }
}
```

# 番外

このライブラリを作っているときに知ったのですが、SwiftのEnumは定義するcase順に順番が決まっているようです。

例えば以下のように定義されたenumの場合は

```Swift
enum Animal: Comparable {
    case cat
    case dog
}
```

以下のように判定されます。

```Swift
XCTAssertTrue(Animal.cat < Animal.dog)
```
