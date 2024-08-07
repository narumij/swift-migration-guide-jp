# 移行戦略

プロジェクトのSwift 6言語モードへの移行を始めましょう。

|原文|[https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/migrationstrategy](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/migrationstrategy)|
|---|---|
|更新日|2024/6/20(翻訳を最後に更新した日付)|
|ここまで反映|[https://github.com/apple/swift-migration-guide/commit/40b11e0f54b6d35345d005511e013c230a520d26](https://github.com/apple/swift-migration-guide/commit/40b11e0f54b6d35345d005511e013c230a520d26)|

モジュールで完全な並行性の確認を有効にすると、コンパイラによって報告されるデータ競合安全性の問題が大量に発生する可能性があります。数百、場合によっては数千の警告は珍しくありません。そのような膨大な問題に直面したとき、特にSwiftのデータ隔離モデルについて学び始めたばかりであった場合、これは難しすぎると感じるかもしれません。

**慌てないで**

移行を進めていくと、実際にはたいていの場合、ほんの少し変更を加えるだけで移行を進められることに気づくでしょう。そして、その過程を通して、Swiftの並行処理システムがどのように動作するかについてのメンタルモデルがあなたの頭の中に急速に作られていくでしょう。

> 重要：このガイダンスが唯一の推奨される方法だと思わないでください。他のアプローチも自信を持って試してみてください。

## 戦略

このドキュメントでは、良いスタート地点となる一般的な戦略の概要を説明します。すべてのプロジェクトに有効な単一のアプローチはありません。

このアプローチには、次の3つの主要なステップがあります:
- モジュールを選択する
- Swift 5でより厳密な並行性の確認を有効にする
- 警告に対処する

このプロセスは _繰り返し_ 行なうものです。1つのモジュールのたった1つの変更であっても、プロジェクト全体の状態に大きな影響を与える可能性があります。

## 外側から始める

プロジェクトの最も外側にある最上位モジュールから始める方が簡単な場合があります。このモジュールは、(最も外側で最上位であるという)定義上、他のモジュールから依存されていません。このモジュールへの変更は局所的にしか影響を及ぼさないので、作業量を抑えられます。

しかし、変更はそのモジュールだけで終える**必要はありません**。あなたがコントロールできる、[安全でないグローバルな状態][Global]や[自動で`Sendable`に準拠している型][Sendable]が存在する依存関係は、プロジェクト全体の中の多くの警告の根本原因になる場合があります。多くの場合、これらは最初に焦点を当てると最も効果的です。

[Global]: <doc:CommonProblems##安全でないグローバルおよび静的変数>
[Sendable]: <doc:CommonProblems#暗黙的なSendable型>

## Swift5言語モードを使う

何も確認せず、Swift 5からSwift 6言語モードにいきなりプロジェクトを移行させるのは、とても難しいと感じるかもしれません。その代わりに、Swift 5言語モードのままで、Swift 6の確認メカニズムの多くを段階的に有効にできます。これは、問題のある部分を警告としてのみ表示し、ビルドとテストをこれまで通り行なえるようにします。

まず、Swift Concurrencyのupcoming featureを1つだけ有効にしましょう。こうすることで、一度に1つの*特定の種類*の問題に集中できます。

プロポーザル    | 内容 | upcoming featureフラグ 
:-----------|-------------|-------------
[SE-0401][] | プロパティラッパーが行なうアクター隔離の推論の削除 | `DisableOutwardActorInference`
[SE-0412][] | グローバル変数に対する厳密な並行性の確認 | `GlobalConcurrency`
[SE-0418][] | メソッドとkey pathリテラルに対する`Sendable`の推論 | `InferSendableFromCaptures`

[SE-0401]: https://github.com/swiftlang/swift-evolution/blob/main/proposals/0401-remove-property-wrapper-isolation.md
[SE-0412]: https://github.com/swiftlang/swift-evolution/blob/main/proposals/0412-strict-concurrency-for-global-variables.md
[SE-0418]: https://github.com/swiftlang/swift-evolution/blob/main/proposals/0418-inferring-sendable-for-methods.md

これらは独立しており、どの順番でも有効にできます。

upcoming featureフラグによって明らかになった問題に対処したら、次のステップは、そのモジュールに対して、[完全な並行性の確認を有効にする][CompleteChecking]ことです。これでコンパイラによるすべてのデータ隔離チェックが有効になります。

[CompleteChecking]: <doc:CompleteChecking>

## 警告に対処する

警告を調査する際には、1つの指針で行なうことを強くおすすめします。つまり、**今のコードの状態で正しくする**ということです。問題に対処するためにコードをリファクタリングしたいという衝動を抑えてください。

完全な並行性の確認を有効にしても警告のない状態にするためには、必要なコードの変更量を最小限に抑えることが効果的です。これが完了したら、警告を抑制するために適用した安全でないオプトアウトを、より安全な隔離メカニズムに置き換えるためのリファクタリングを行なう機会として利用してください。

> 重要: 頻出のコンパイルエラーに対処する方法については、<doc:CommonProblems>を参照してください。

## 繰り返す

最初のうちは、データ隔離の問題を無効にしたり回避したりするテクニックを使うことになるでしょう。上位モジュールで、これ以上問題を無効にしたり回避したりする必要がないと感じたら、ワークアラウンドが必要な依存関係のうちの1つを次のターゲットにしてください。

先に進むためにすべての警告を排除する必要はありません。とても些細な変更でさえもが大きな影響を与える可能性があることを覚えておいてください。依存関係の1つを更新できれば、いつでも元のモジュールを更新する作業に立ち戻ることができます。