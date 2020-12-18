---
title: "ブラックジャックの必勝法「カウンティングアプリ」を作った"
emoji: "🃏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SwiftUI]
published: false
---

:::message
これは[CAMPFIRE Advent Calendar 2020](https://qiita.com/advent-calendar/2020/campfire)の19日目の記事です。
:::

# 使用技術

- SwiftUI
- Combine
- The Composable Architecture

# ブラックジャックとは

## ルール

- ディーラー対プレイヤーのゲーム
- 手札の合計点数が21に近い方勝ちです
- もし手札の合計点数が21を超えた場合負けです

## 点数の付け方

- 2〜9はそのままの数字として扱い
- 10,J,Q,Kはすべて10として扱い
- Aは1もしくは11として扱います

## 流れ

- デッキの中からディーラーとプレイヤーに2枚ずつ配ります。ただしディーラーの2枚目のトランプは非公開
- プレイヤーは配られた2枚の合計点数を見て、更にトランプを引くかそこで止めるか選べます
- ディーラーは合計点数が17を超えるまで引き続けます
- 最後にディーラーとプレイヤーの合計点数を見て勝敗が決まります

## 配当

- 最初に配られる2枚の合計点数が21のとき、掛け額の1.5倍もらえます
- ディーラーに勝った場合、掛け額の1.0倍もらえます
- ディーラーに負けた場合、掛け額がすべて没収されます

## ブラックジャックで勝率を上げるには

ブラックジャックでは10,J,Q,K,Aが重要になってきます。なぜならAと10,J,Q,Kの組み合わせになったとき合計点数が21となり、掛け額の1.5倍をもらうことが出来るからです。

なので10,J,Q,K,Aが残りデッキの中にどれぐらいあるのか？が分かると、それらのトランプが多いときに掛け額を上げてより多くの配当を得ることが可能になります。

# カウンティング

## カウンティングとは

カウンティングとは場に出たトランプが何かをカウントし、残りのデッキにどれぐらい10,J,Q,K,Aが残っているのか把握する手法です。

## カウンティングの方法

場に出たトランプをすべて記憶することができれば正確に把握することができますが記憶力には限界があります。
すべてにカードを記憶することは不可能に近いのでトランプを3つのグループに分け計算をしていきます。

具体的なグループとしては

- 10, J, Q, K, A
- 7, 8, 9
- 2, 3, 4, 5, 6

|  |  |
| --- | ----- |
| -1 | 10, J, Q, K, A |
| 0 | 7, 8, 9 |
| +1 | 2, 3, 4, 5, 6 |

上記のリストの通り「10, J, Q, K, A」のトランプを-1、「7, 8, 9」のトランプを±0、「2, 3, 4, 5, 6」のトランプを+1として計算します。

場に出たすべてのトランプを上記の法則に従い計算していきます。

例えば+に数字が大きい場合はブラックジャックでは重要なトランプである「10, J, Q, K, A」がデッキにたくさんあることが想定できます。

例えば
- 残りのトランプが100枚あり、カウントが+20
- 残りのトランプが40枚あり、カウントが+20

の場合では後者の方が「10, J, Q, K, A」が出てくる確率が高いことが分かります。

なので単純なカウントがいくつなのかも重要ながら、残りのトランプ総数で割ったときにカウントも重要になってきます。


# 作ったアプリについて

## 何を作ったのか

カウンティングの方法で述べたとおりにカウントをすればいいのですが、カウントを記憶することや瞬時に計算することは難しいので、簡単にカウンティングを行えるアプリを作りました。

| | | |
|-|-|-|
| ![](https://storage.googleapis.com/zenn-user-upload/avke1v03swch7g4dihqztbllnnkc) | ![](https://storage.googleapis.com/zenn-user-upload/3yns1xjt4sohjb7x2i8ehc20ah10) | ![](https://storage.googleapis.com/zenn-user-upload/49htbe92wn5v0r2eayr6kuiv19cp) |

場に出たトランプをタップすると自動でカウントをしてくれるようになっています。

UIで誤魔化していますが基本的にはただのカウントアプリになっています。

## どう作ったのか

今から新規でアプリを作るならSwiftUI一択だと考えていたのでSwiftUIを使用しました。

アーキテクチャとしてはThe Composable Architectureを使用しています。

トランプに関するEntityは[自作でライブラリ](https://github.com/tomoki69386/PokerHands)として公開しているものがあるのでそれを使用しています。

カウンティングアプリ自体の実装はただのカウントアプリですごく簡単なものなので今回は触れません。気になる人はコードを見てみてください

## おもしろかったところ

カウントをリセットしてくれる機能を作ったのですが、誤って押してしまう可能性のある場所にボタンを配置したので「本当にリセットしますか？」とアラートで確認を取るようにしました。

<img src='https://storage.googleapis.com/zenn-user-upload/ni9vo6ivhg960tnkgpajdbx2a3my' width=300 />

SwiftUIでは以下のようにしてアラートを出すことができます

```Swift
struct ContentView: View {
    @State private var isPresented = false
    
    var body: some View {
        Button("Tapped") {
            self.showingAlert = true
        }
        .alert(isPresented: $isPresented) {
            Alert(title: Text("Alert title"))
        }
    }
}
```

ですがThe Composable Architectureの場合は以下のようにアラートを表示します。

```Swift
import SwiftUI
import ComposableArchitecture

struct CounterState: Equatable {
    var alert: AlertState<CounterAction>?
}

enum CounterAction: Equatable {
    case tappedResetCountButton
    case alertDismissed
    case acceptResetting
    case cancelResetting
}

struct CounterEnvironment { }

let counterReducer = Reducer<CounterState, CounterAction, CounterEnvironment> { state, action, _ in
    
    switch action {
    case .tappedResetCountButton:
        state.alert = .init(
            title: "カウンティングをリセットしますか？",
            primaryButton: .cancel(
                send: CounterAction.cancelResetting
            ),
            secondaryButton: .default("リセットする",
                send: CounterAction.acceptResetting
            )
        )
        return .none
    case .alertDismissed:
        state.alert = nil
        return .none
    case .acceptResetting:
        state.count = 0
        return .none
    case .cancelResetting:
        return .none
    }
}

struct CounterView: View {
    
    let store: Store<CounterState, CounterAction>
    
    var body: some View {
        WithViewStore(self.store) { viewStore in 
          Button(action: { 
              viewStore.send(.tappedResetCountButton)
          }, lael: { 
              Text("Reset")
          })
          .alert(self.store.scope(state: { $0.alert }), dismiss: .alertDismissed)
        }
    }
}
```

これを利用することでActionを簡単に渡すことができ、Stateの変更も簡単におこなうことができました。

# その他

- 今回作ったアプリのソースコードはGitHubで公開しています。機能の追加や修正点などあればPRお待ちしてます。
  - https://github.com/tomoki69386/BlackjackCounting

- 今回作ったアプリはAppStoreにて配信中です。気になった方はダウンロードしてレビュー書いてください！！！！
  - https://apps.apple.com/us/app/blackjack-counting/id1544949227

- またアプリの中で利用しているPokerHandsというライブラリもGitHubで公開しています。
  - https://github.com/tomoki69386/PokerHands
