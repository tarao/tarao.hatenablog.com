---
Title: なぜ型ファーストで考えるのか
Category:
- article
- lambda
- lang
- scala
- javascript
Date: 2020-02-18T10:23:37+09:00
URL: https://tarao.hatenablog.com/entry/type-first
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/26006613515666318
---

><blockquote class="epigraph">
  <p>How do you imagine a building?  You consciously create each aspect, puzzling over it in  stages.</p>
  <cite><a href="https://www.imdb.com/title/tt1375666/">Inception</a></cite>
</blockquote><

型なし言語に馴染みはあるものの型付言語をいざ使ってみたらどういう気持ちで書いたらいいのかわからなかったと同僚から相談があり, それをきっかけにして社内の勉強会で以下の話をしました.

[https://speakerdeck.com/tarao/computation-first-vs-type-first:embed]

><div style="display:none">
[f:id:tarao:20200210222444p:image:w320:right]
</div><

よく型なし vs. 型付の文脈では「型を書くのは面倒だ」「安全の方が大事だ」「でも面倒だ」「それは型推論を前提にしていないからだ」などの議論になりがちな気がしますが、これはあくまで「計算ありきの型」を考えているからで, 「型ありきの計算」だと全く見え方が違います.  「型はある種の仕様」とおもえば, 型ファーストであることと, 型なし言語でテスト駆動開発(TDD)するときに最初にテストを書くこととは, 同じなんだよなぁと思っていました.  そして型ファーストが重要な理由をつきつめていくと, Curry-Howard同型対応があるからだなぁという結論に至ったので, それを言語化しました.  せっかくなのでスライドを公開するとともに、口頭で補ったところも含めて丁寧に記事として書き起こしました。

====
TDDとの共通性からもわかる通り, 型なし言語を腐す話ではありません.  筆者も, 型付と型なしを混ぜる漸進的型付け(gradual typing)について, 型なし言語に型を導入するのではなく[https://www.jstage.jst.go.jp/article/jssst/26/2/26_2_2_18/_pdf/-char/ja:title=型付言語から型を取り除くアプローチで研究していた]人間なので, 型のない言語に恨みはなく, メリットも十分に理解しています.  単に, 型付言語でばりばり書いていて何も困っていない人たちはこういう気持ちで書いています, という話です.

[:contents]
><div style="display:none">
[tex:
    \require{enclose}
    \def\textsc#1{\dosc#1\csod}
    \def\dosc#1#2\csod{{\rm #1{\scriptsize #2}}}
    \def\tinysc#1{\tinydosc#1\tinycsod}
    \def\tinydosc#1#2\tinycsod{{\rm #1{\tiny #2}}}
    \def\sout#1{\enclose{horizontalstrike}{#1}}
    \def\code#1{\texttt{#1}}
    \def\infrule(#1)#2#3{\displaystyle\frac{#2}{#3}\,(\textsc{#1})}
    \def\infrulew#1#2(#3)#4#5{\displaystyle\frac{#1}{#2}(\textsc{#3})\frac{#4}{#5}}
    \def\andalso{\quad\quad}
    \def\ty|#1|{\left\lfloor #1 \right\rfloor}
    \def\infer(#1)#2#3{\displaystyle\frac{#3}{#2}\,{\scriptsize (\tinysc{#1})}}
    \def\uninfer(#1)#2#3{\displaystyle\frac{#3}{#2}{\scriptsize \hl{\sout{(\tinysc{#1})}}}}
    \def\hl#1{{\color{red}{#1}}}
]
</div><
** 計算ファーストで考える型

「型ファースト」について考える前に, 「計算ファースト」で考えた場合に型をつけるとはどういうことなのか, 整理しておきましょう.

*** 型がない場合

型がない場合, 意図とは違う誤った使い方を防げなかったり, 誤った使い方をしていても実行してみるまで誤りに気づけなったりします.  例を2つほど見てみましょう(例はJavaScriptで書いています).

>|typescript|
let apply2 = (f, n) => f(f(n))
apply2(1, 3)
||<

><div class="error">
>||
Uncaught TypeError: f is not a function
||<
</div><

本来は関数を渡すべきところに関数以外のものを渡してしまった例です.  こんな単純なミスはしないかもしれませんが, 呼び出し関係が複雑になってくれば「関数が渡ってくると思っていたら意外とそうでもなかった」となり, 関数でないものを「関数しか渡してはいけないところ」に渡してしまうかもしれません.

実行時エラーなので<code>apply2</code>の関数本体がエラー発生箇所になってしまっている点は特筆すべきです.  悪いのは呼び出し側のはずですが, あたかも<code>apply2</code>の実装が悪いかのようなエラーメッセージになってしまっていますね.

>|typescript|
let omega = (x) => x(x)
omega(omega)
||<

><div class="error">
>||
Uncaught RangeError: Maximum call stack size exceeded
||<
</div><

この例はなんかめちゃくちゃですね.  なんの役に立つのかよくわからないものがよくわからない使われ方をしています.

型がないとこういったものを書こうと思えば書けてしまいます.  いい意味では自由になんでも書けますが, 自由には代償をともないます.

*** 書いてよい式を型で制限する

計算ありきで考えたときの型の役割は「めちゃくちゃなことができないように制限をする」ことです.  先程の例の場合は型をつけると以下のようになります(例はTypeScriptで書いています).

>|typescript|
let apply2 = <A>(f: (a: A) => A, n: A) => f(f(n))
apply2(1, 3)
||<

><div class="error">
>||
Argument of type '1' is not assignable to parameter of type '(a: 3) => 3'
||<
</div><

この例では重要なことが2つあります.

- エラーは実行時ではなくコンパイル時に出ている
- <code>apply2</code>の関数本体ではなく呼び出し側のエラーになっている

>|typescript|
let omega = <A, B>(x: (a: A) => B) => x(x)
||<

><div class="error">
>||
Argument of type '(a: A) => B' is not assignable to parameter of type 'A'
||<
</div><

今度は<code>omega</code>の定義の時点でエラーになります.  「なんの役に立つのかよくわからないもの」のうち, 明らかに危険なものは型があればそもそも書けなくなる例です.

*** 型検査

現代の処理系で, このような「型がつく」ことの検査をどのようにやっているか少し話しておきましょう.  型を検査するためには, あらかじめ言語仕様として型付け規則(typing rule)が定義されていて, それらの規則だけを使って「ある式にある型がつく」ことを(自動)証明する手続きを踏みます.  難しいことを言っていますね.  ひとつずつ見ていきましょう.

まず型付け規則とはどういうものか.  実際のプログラミング言語のフルの仕様の型付け規則を見るのは非常にたいへんなので, 型理論の源流となっている単純型付ラムダ計算(以下STLC)の場合で見てみましょう(式の部分はTypeScriptの構文にしてあります).

[tex:
    \infrule(VAR)
    {\code{x}: \code{T} \in \Gamma}
    {\Gamma \vdash \code{x}: \code{T}}
    \qquad
] [tex:
    \infrule(ABS)
    {\Gamma, \code{x}: \ty|\code{S}| \vdash \code{m}: \code{T}}
    {\Gamma \vdash (\code{(x:S) => m}): \ty|\code{S}| \to \code{T}}
]

[tex:
    \infrule(APP)
    {
        \Gamma \vdash \code{m}: \code{S} \to \code{T}
        \andalso
        \Gamma \vdash \code{n}: \code{S}
    }
    {\Gamma \vdash \code{m(n)}: \code{T}}
    \qquad
] [tex:
    \left\{ \begin{array}{rcl}
        \ty|\code{(x:S) => T}| & = & \ty|\code{S}| \to \ty|\code{T}| \\
        \text{(otherwise)} \quad
        \ty|\code{T}|          & = & \code{T} \\
    \end{array} \right.
]

STLCの型付け規則[tex:\textsc{VAR}], [tex:\textsc{ABS}], [tex:\textsc{APP}]は, ラムダ計算の構文要素である変数(<strong>var</strong>iable), 関数抽象(<strong>abs</strong>traction), 関数適用(<strong>app</strong>lication)にそれぞれ対応して定義されています.  正確を期すため, TypeScriptの構文での型を, 型の表記として一般的な形に変換する操作[tex:\ty|\code{T}|]も定義しています.  これは単に[tex:\code{(x:S) => T}]の形のTypeScriptの関数型をここでは[tex:\code{S}\to\code{T}]と読み替えたいだけで, あまり重要ではありません.

いくつか読み方の約束を書いておきます.  [tex:\code{x}]は変数を表すためのメタ変数で, 任意の識別子が入ると思ってください.  [tex:\code{m}], [tex:\code{n}]は式を表すためのメタ変数で任意の式が入り, [tex:\code{S}], [tex:\code{T}]は型を表すためのメタ変数で任意の型が入ります.  [tex:\Gamma]は型環境と言って, 基本的には変数と型の組を[tex:\code{x}:\code{T}]の形で要素に持つ集合です.  ただし, [tex:\{ \code{x}_1:\code{T}_1, \ldots, \code{x}_n:\code{T}_n \}]と書かずに単に[tex:\code{x}_1:\code{T}_1, \ldots, \code{x}_n:\code{T}_n]と書き, [tex:\Gamma \cup \{ \code{x}:\code{T} \}]と書かずに単に[tex:\Gamma, \code{x}:\code{T}]と書きます.  また, 同じ変数を異なる型と組にして重複して含んではいけない([tex:\code{a}:\code{Int}, \code{a}:\code{String}]のようにしてはいけない)こととします(つまり[tex:\Gamma]は単なる組の集合ではなく, 変数から型への写像です).

[tex:\Gamma \vdash \code{m}: \code{T}]は判断(judgment)と言い, [tex:\Gamma]という仮定の下で[tex:\code{m}: \code{T}]が成り立つという主張です(主張なのでまだこれだけでは本当に成り立つかどうかはわかりません).  いまは型付け規則の話をしているので, これは型判断であり, [tex:\Gamma]という環境([tex:\Gamma]で示した方法により各変数の型を割り当てたという仮定)の下で[tex:\code{m}]に[tex:\code{T}]という型がつくと主張しています.

それぞれの型付け規則は, 横線の上が前提で下が結論です.  前提が成り立つ場合に限って, 結論に書かれた判断が成り立つと思ってよいことを意味します.  前提が複数ある場合([tex:\textsc{APP}]の規則)は, 両方の前提が成り立つ場合に限り結論を導いてよいとします.

型検査は, このような型付け規則のみを用いて[tex:\Gamma \vdash \code{m}: \code{T}]の形の判断が成り立つと言えるか調べることです.  もし結論を導くのに[tex:\textsc{APP}]や[tex:\textsc{ABS}]を用いるとすると, これらの規則の前提にはまた[tex:\vdash]が登場するので, 前提が成り立つと言うために再び3つの規則のうちのいずれかを用いなければなりません.  そうやって前提を成り立たせるための規則をどんどんつなげていくと([tex:\textsc{APP}]で枝分かれが生じるため)木構造ができ上がります.  木構造の葉の部分がすべて[tex:\textsc{VAR}]の規則で終わっていれば, もうそれ以上は規則を当てはめる必要がなくなり, 最終的な結論(木構造の根のところにある判断)が成り立つと言えます.  このように規則をつないで結論を導くことを導出(derivation)と言います.

*** 型検査: 型がつく例

例を見てみましょう.  <code>(f: (a: A) => A) => (n: A) => f(f(n))</code>に[tex:(A \to A) \to A \to A]の型がつくことを確認します.

[tex:
    {\scriptsize
    \infer(APP)
    {
        \infer(ABS)
        {\vdash (\code{(f: (a:A) => A) => (n:A) => f(f(n))}): (\code{A} \to \code{A}) \to \code{A} \to \code{A}}
        {
            \infer(ABS)
            {\code{f}: \code{A} \to \code{A} \vdash (\code{(n:A) => f(f(n))}): \code{A} \to \code{A}}
            {\code{f}: \code{A} \to \code{A}, \code{n}: \code{A} \vdash \code{f(f(n))}: \code{A}}
        }
    }
    {
        \infer(VAR)
        {\code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \vdash \code{f}: \code{A} \to \code{A}}
        {\code{f}: \code{A} \to \code{A} \in \{ \code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \}}
        \quad
        {
            \infer(APP)
            {\code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \vdash \code{f(n)}:\code{A}}
            {
                \infer(VAR)
                {\code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \vdash \code{f}: \code{A} \to \code{A}}
                {\code{f}: \code{A} \to \code{A} \in \{ \code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \}}
                \quad
                \infer(VAR)
                {\code{f}: \code{A} \to \code{A}, \code{n}:\code{A} \vdash \code{n}: \code{A}}
                {\code{n}:\code{A} \in \{ \code{f}: \code{A} \to \code{A}, \code{n}: \code{A} \}}
            }
        }
    }
    }
]

いきなり完成した導出を見せてしまいましたが, どうやってこれが導かれるのか確認していきましょう.

まず, [tex:\code{(f: (a: A) => A) => (n: A) => f(f(n))}]の式は全体が関数になっています.  3つの型付け規則はすべて結論の部分の形が違っていて, 結論が関数の形にマッチするのは[tex:\textsc{ABS}]の規則のみなのでこれを使います.  式を[tex:\textsc{ABS}]の結論にパターンマッチすると, [tex:\code{x} = \code{f}], [tex:\code{S} = \code{(a:A) => A}], [tex:\code{m} = \code{(n: A) => f(f(n))}]になります.  型の部分も同様に[tex:(A \to A) \to A \to A]を[tex:\ty|\code{S}| \to \code{T}]にパターンマッチすると, [tex:\code{T} = \code{A} \to \code{A}]とわかります.  これらの情報を用いると, [tex:\textsc{ABS}]の前提部分は以下のようになるはずです.

[tex:\code{f}:\code{A} \to \code{A} \vdash (\code{(n: A) => f(f(n))}): \code{A} \to \code{A}]

前提が判断の形なので, 成り立つことを示す必要があります.  この判断も, 式の形が関数なので[tex:\textsc{ABS}]を使います.  すると前提は以下の形になります(いま下から3段目まできました).

[tex:\code{f}: \code{A} \to \code{A}, \code{n}: \code{A} \vdash \code{f(f(n))}: \code{A}]

今度は関数適用(関数呼び出し)の形になっていて, マッチするのは[tex:\textsc{APP}]の規則です.  [tex:\textsc{APP}]は前提が2つあるので, 枝が2つに分かれます.  左右の枝は次のようになります(いま下から4段目です).

(左) [tex:\code{f}: \code{A} \to \code{A}, \code{n}: \code{A} \vdash \code{f}: \code{A} \to \code{A}]
(右) [tex:\code{f}: \code{A} \to \code{A}, \code{n}: \code{A} \vdash \code{f(n)}: \code{A}]

左の枝は関数抽象でも関数適用でもなく, マッチするとしたら[tex:\textsc{VAR}]の規則だけです.  実際にマッチしていて, 型環境[tex:\code{f}: \code{A} \to \code{A}, \code{n}: \code{A}]には[tex:\code{f}: \code{A} \to \code{A}]を含んでいるので前提も成り立ちます.  [tex:\textsc{VAR}]の規則を使ったので, 左の枝はこれで終わりで, 成り立つことが確認できました.

右の枝も同様に繰り返し規則を使っていくと[tex:\textsc{VAR}]で終わらせることができます.

型検査とはこのように, 型付け規則をパターンマッチして結論から前提へと木構造を作っていき, すべての枝が判断を含まない前提で終わる形にすることで, たしかに結論の型がつくと確かめることです.

*** 型検査: 型がつかない例

いまのは型がつく場合の例でした.  型がつかない場合にどうなるかも見ておきましょう.  <code>(x: (a:A) => B) => x(x)</code>の式には型がつかないことを確認します.

まず, 式は関数の形になっているので最初に使う規則は[tex:\textsc{ABS}]しかありえません.  結論の部分にパターンマッチすると, [tex:\code{x} = \code{x}], [tex:\code{S} = \code{(a:A) => B}], [tex:\code{m} = \code{x(x)}]となります.  [tex:\code{T}]はまだなにかわからないのでいったん[tex:\code{T}]のままにしておきます.  これらの情報を使うと前提は以下の判断になります.

[tex:\code{x}: \code{A} \to \code{B} \vdash \code{x(x)}: \code{T}]

この判断の式の形は関数適用なので[tex:\textsc{APP}]の規則を使います.  前提の左右の枝は以下のようになります([tex:\code{S}]はまだなにかわからないのでそのままにしておきます).

(左) [tex:\code{x}: \code{A} \to \code{B} \vdash \code{x}: \code{S} \to \code{T}]
(右) [tex:\code{x}: \code{A} \to \code{B} \vdash \code{x}: \code{S}]

左の枝にマッチするのは[tex:\textsc{VAR}]の規則だけです.  パターンマッチしてみると前提が[tex:\code{x}: \code{S} \to \code{T} \in \{ \code{x}: \code{A} \to \code{B} \}]でないといけないとわかるので, けっきょく[tex:\code{S} = \code{A}], [tex:\code{T} = \code{B}]だったとわかります.  ここまでは辻褄が合っていてとくに問題ありません.

一方, [tex:\code{S} = \code{A}]とわかったので右の枝は以下のようになっています.

(右) [tex:\code{x}: \code{A} \to \code{B} \vdash \code{x}: \code{A}]

やはり使える規則は[tex:\textsc{VAR}]だけですが, パターンマッチしてみると前提として[tex:\code{x}: \code{A} \in \{ \code{x}: \code{A} \to \code{B} \}]になる必要があり, これは成り立ちません.  [tex:\code{A} \ne \code{A} \to \code{B}]だからです.  [tex:\textsc{VAR}]の前提を満たす以外に導出木を下から上へのばしていく方法はないので, 導出木は完成できません.  したがって, 右の枝がどうやっても完成しないので, <code>(x: (a:A) => B) => x(x)</code>の式には型がつかないとわかります.  全体像としては以下のようになっていました.

[tex:
    \infer(APP)
    {
        \infer(ABS)
        {\vdash (\code{(x: (a:A) => B) => x(x)}): (\code{A} \to \code{B}) \to \code{B}}
        {\code{x}: \code{A} \to \code{B} \vdash \code{x(x)}: \code{B}}
    }
    {
        \infer(VAR)
        {\code{x}: \code{A} \to \code{B} \vdash \code{x}: \code{A} \to \code{B}}
        {\code{x}: \code{A} \to \code{B} \in \{ \code{x}: \code{A} \to \code{B} \}}
        \quad
        \uninfer(VAR)
        {\code{x}: \code{A} \to \code{B} \vdash \code{x}: \code{A}}
        {\code{x}: \hl{\code{A}} \not\in \{ \code{x}: \hl{\code{A} \to \code{B}} \}}
    }
]

型付け規則やその導出に慣れるには練習が必要そうですね.  こういったやり方を身につけたい人は以下の本を読むとよさそうです.  オンライン課題提出(自動採点)システムで練習もできます.

[asin:4781912850:detail]

実際の型付言語の処理系は, 型がつく/つかないの検査を自動的にやるプログラム(型検査器(type checker))を内蔵していて, 人間にかわってこのような検査をやってくれているわけです.

*** 型安全性

型付け規則を定めておくと, それに基づいて型検査器を実装できるだけでなく, 型検査を通ったプログラムが満たす性質を数学的な議論により証明できます.  巷で言われる「型安全性」とはこのことです.

たとえばSTLCの場合は以下のような性質が成り立ちます.

><dl><dt>型のついた式は</dt>
<dd>
- 実行した結果の値も同じ型になる.
-- 最終結果だけでなく実行途中も同様
-- 主部簡約定理(subject reduction)と言う
- 実行したら必ず停止する(必ず結果の値が得られる).
-- 評価戦略によらない(遅延評価しても結果が変わらない)
-- 強正規化性(strong normalization)と言う
</dd></dl><

これはSTLCの場合で, ふつうのプログラミング言語は無限ループが書けるので2つ目の性質は少し違って「停止したときは必ず値になっている(エラーにはならない)」という形になります.  いずれにせよ, 型をつけることである種の「安全な操作」しかできないようになっているのは, このような性質によります.  型付け規則は闇雲に決めているわけではなく, こういった性質が成り立つようにうまく定めているのですね.

型付け規則のバリエーションや, 成り立つ性質の議論などについてもっと詳しく知りたい人は以下の本を読みましょう.

[asin:4274069117:detail]

*** 型検査の限界

STLCの型付け規則は非常に単純で, それゆえ限界もあります.

- チューリング完全ではない(再帰関数が書けない)
- 一見すると問題ないのに書けない式がある

1つ目は, 実際のプログラミング言語では再帰関数のための型付け規則が入っているのでまず問題になりません.  しかし2つ目はしばしば問題になります.

たとえば, 型がなければとくに問題のない以下のようなプログラムを考えます.

>|typescript|
let apply2 = (f, n) => f(f(n))
apply2((n) => [n], 3)
||<

>||
=> [[3]]
||<

一方, これに素朴に型をつけるとうまくいきません.

>|typescript|
let apply2 = <A>(f: (a: A) => A, n: A) => f(f(n))
apply2((n) => [n], 3)
||<

><div class="error">
>||
Type 'number[]' is not assignable to type 'number'
||<
</div><

型システム(書ける型の種類や型付け規則を定めたもの)を工夫すればこういった不便をなくせる可能性はありますが, 一般に型システムの自由度を上げると型検査が決定不能((停止するアルゴリズムが存在しないこと))になったり, 型安全性が成り立たなくなったりします.  さまざまな型システムを備えたプログラミング言語が提案され続けるのは, 型検査が万能にはなりえない中で自由と安全のバランスを模索しているからです.

><aside class="column">
<h5><span class="only-in-toc">(コラム)</span> そもそも型とは</h5>

<p>型とは, 区別したい概念によってものの集まりを分けたものです.  「型」という名前こそ出てこないものの, ユークリッドが『原論』を著したときに「点」と「線」とを区別していたように, 考え方そのものは大昔から存在します.</p>

<p>こういった概念に「型」という名前を与えたのは[https://www.jstor.org/stable/pdf/2369948.pdf:title=Russellの"Mathematical Logic as Based on the Theory of Types"(1908)]で, これは[https://ja.wikipedia.org/wiki/%E3%83%A9%E3%83%83%E3%82%BB%E3%83%AB%E3%81%AE%E3%83%91%E3%83%A9%E3%83%89%E3%83%83%E3%82%AF%E3%82%B9:title=ラッセルのパラドックス]を避けるために必要なアイディアでした.  その後, [https://pdfs.semanticscholar.org/28bf/123690205ae5bbd9f8c84b1330025e8476e4.pdf:title=1940年に発表されたChurchの単純型付ラムダ計算]に至るまで, 「型」はもっぱら論理学上の関心事((当時はコンピュータもプログラムも存在しなかったので, 当然プログラミング上の関心事にはなりえません))で, 命題関数(論理式をとる関数)が自己言及(つまり再帰呼び出し)を含んで矛盾してしまうのを防ぐためにありました.  単純型付ラムダ計算で再帰関数が書けないのはこのためです.</p>

<p>これら1940年までの「型」にまつわる歴史は以下の文献にまとめられています.</p>

- [https://www.researchgate.net/publication/2832399_Types_in_Logic_and_Mathematics_Before_1940:title=Types in Logic and Mathematics Before 1940].<br>Fairouz Kamareddine, Twan Laan, and Rob Nederpelt.<br><span class="journal">Bulletin of Symbolic Logic</span>, 8(2):185-245, 2002.

<p>最近はあまり見かけなくなったようにも思いますが, 「型とはメモリ上のデータの扱い方を区別するためのもの」のような説明を目にするかもしれません.  しかしこれは「データ型」の側面しかとらえておらず, ALGOLの系譜に偏った見方です.  現代の型理論はALGOLが登場するよりずっと前から議論されてきた「型」の概念に基づいている点に留意しておきましょう.</p>
</aside><

*** ここまでのまとめ

計算ありきで考えたときの型は, 自由を制限する代わりに安全を得るためのしくみでした.

- 型がなければ自由だがめちゃくちゃなこともできてしまう
- 型によって安全がもたらされるが自由は制限される
- 型システムは自由と安全のバランスの下に発展してきた

** 型ファーストで考える計算

ここまでは計算ファーストで, まずなにかやりたいことをコードで書いて, そこに型を書き加えることで安全になる話をしてきました.  しかし実際に型付言語をばりばり書いている人たちはあまりそういうつもりで書いておらず, どちらかと言うとまず型を考えていそうです((計算ファーストだけど型付言語をばりばり書いているつもりだった人は......なんか, がんばりましょう)).  まずやりたいことを型で表して, その型に沿う実装を書いていくのです.

この感覚は, 型をある種の仕様だと思うとしっくりきます.  実は感覚だけの問題ではなく, 理論上の必然としてそうなります.

*** 型はある種の仕様

まずは例を見てみましょう.  <s>関数型が<code>(x: A) => B</code>みたいな記法になるのがだるいので</s>とくに言語は関係ないので型の表記がわかりやすいScalaのメソッドのシグネチャをいくつか見てみます.  Scalaが読める必要はありません.

1つ目は, はてなブックマークのソースコードに実際に登場するメソッドのシグネチャです.

>|scala|
BookmarkRepository.find: BookmarkId => Option[BookmarkEntity]
// - ブックマークIDがあるなら,
// - ブックマークのエンティティがあれば得られる
||<

コメントとして書いたように, 型をそのまま読んで仕様がわかりますね.  <code>Option[A]</code>型は<code>Some[A]</code>(値があるとき)もしくは<code>None</code>(値がないとき)を表す型なので, 「あれば」とつけるとスムーズに読めますね.

次の例はScalaの標準ライブラリから引っぱってきました.  列をソートするメソッドです.

>|scala|
Seq.sortBy: Seq[A] => (A => B) => Ordering[B] => Seq[A]
// - Aの列があり,
// - AをBに変換でき,
// - Bの順序が規定されていれば,
// - (Bでソート済みの)Aの列が得られる
||<

<code>Seq[A]</code>, <code>A => B</code>, <code>Ordering[B]</code>をそれぞれ, <code>A</code>の列がある, <code>A</code>を<code>B</code>にできる, <code>B</code>の順序が与えられている, と読めば, そこから新たな<code>A</code>の列ができる, と言っているとわかります.  最後に得られる列が<code>B</code>でソート済みなのは残念ながら型を見ただけではわかりません.

最後の例もScalaの標準ライブラリからです.

>|scala|
Either.fold: Either[A, B] => (A => C) => (B => C) => C
// - AまたはBどちらかのインスタンスがあり,
// - AからCへの変換と, BからCへの変換があれば,
// - 常にCのインスタンスが得られる
||<

<code>Either[A, B]</code>は, <code>A</code>もしくは<code>B</code>どちらのインスタンスでもかまわないときに使う型です.  たとえば<code>A</code>にエラーを表す型を入れて, 「<code>B</code>が得られるか, もしくはエラーが返る」ときによく使います.

<code>Either</code>には<code>fold</code>メソッドがあり, <code>A</code>, <code>B</code>それぞれを共通の<code>C</code>に変換するメソッドを与えると, 実際の値が<code>A</code>, <code>B</code>どちらであってもとにかく<code>C</code>型の値が得られます.  <code>fold</code>の型を読んだそのままですね.

このように, 型<code>A</code>が出てきたら「<code>A</code>のインスタンスがある」に, <code>=&gt;</code>が出てきたら「ならば」に置き換えながら読んでいくと, なんとなくどういう仕様のメソッドかを表した文になりそうです.

*** Curry-Howard同型対応

型を読むとまるで仕様のようになるのは偶然ではありません.  ここまではふんわりと「仕様」と言っていましたが, 実のところ型は論理式で表した命題そのものであり, (型のついた)式による実装はその命題が成り立つことの証明です.

|*計算体系   |*論理体系|*(気持ち)           |
|型          |論理式   |(仕様)              |
|型のついた式|証明     |(仕様が満たせる証拠)|

つまり, 式に型がつくことと, (その型に対応する論理式で表される)仕様を満たす実装が存在することは一致します.  この対応関係を<strong>Curry-Howard同型対応(Curry-Howard isomorphism)</strong>と言います((ちなみに, この対応関係の発見者のCurryさんは, プログラミング言語Haskellの名前の由来となったHaskell Curryさんであり, カリー化の概念でおなじみのカリーさんです))((ちなみに, 計算体系と論理体系の一致に加えて, デカルト閉圏という圏論上の概念とも一致することが知られていて, Curry-Howard-Lambek対応とも言います)).

いきなり「型は論理式で式は証明だったんだよ!!!」と言われてもわけがわからないと思うので, 実際に一致していそうなことを視覚的に確認してみましょう.

まずは, STLCに加えて直積型(タプル)や直和型((TypeScriptには直接的な直和型はありませんが<a href="https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions">やりようはある</a>のでそれを反映した形になっています))を入れた言語の型付け規則を見てみます(式の部分はTypeScriptですが都合により型の表記は別になっており, また[tex:\bot]に関するルールも加えられています).

><div style="display:none">
[tex:
    \def\inhabitant#1{#1}

    \def\Ax{
    \Gamma, \inhabitant{\code{x}:} \varphi \vdash \inhabitant{\code{x}:} \varphi\,(\textsc{AX})}

    \def\ExFalso{
    \infrule(\bot E)
    {\Gamma \vdash \inhabitant{\code{m}:} \bot}
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi}}

    \def\ImpI{
    \infrule(\to I)
    {\Gamma, \inhabitant{\code{x}:} \ty|\varphi| \vdash \inhabitant{\code{m}:} \psi}
    {\Gamma \vdash \inhabitant{(\code{(x:$\varphi$) => m}):} \ty|\varphi| \to \psi}}

    \def\ImpE{
    \infrule(\to E)
    {
        \Gamma \vdash \inhabitant{\code{m}:} \varphi \to \psi
        \andalso
        \Gamma \vdash \inhabitant{\code{n}:} \varphi
    }
    {\Gamma \vdash \inhabitant{\code{m(n)}:} \psi}}

    \def\ConjI{
    \infrule(\wedge I)
    {
        \Gamma \vdash \inhabitant{\code{m}:} \varphi
        \andalso
        \Gamma \vdash \inhabitant{\code{n}:} \psi
    }
    {\Gamma \vdash \inhabitant{\code{\[m, n\]}:} \varphi \wedge \psi}}

    \def\ConjE{
    \infrulew
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi \wedge \psi}
    {\Gamma \vdash \inhabitant{\code{m\[0\]}:} \varphi}
    (\wedge E)
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi \wedge \psi}
    {\Gamma \vdash \inhabitant{\code{m\[1\]}:} \psi}}

    \def\DisI{
    \infrulew
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi \andalso \inhabitant{\code{kind}\ \mathrm{distinct\ in}\ \varphi, \psi}}
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi \vee \psi}
    (\vee I)
    {\Gamma \vdash \inhabitant{\code{m}:} \psi \andalso  \inhabitant{\code{kind}\ \mathrm{distinct\ in}\ \varphi, \psi}}
    {\Gamma \vdash \inhabitant{\code{m}:} \varphi \vee \psi}}

    \def\DisE{
    \infrule(\vee E)
    {
        \Gamma \vdash \inhabitant{\code{m}:} \varphi \vee \psi
        \andalso
        \Gamma \vdash \inhabitant{\code{n}_1:} \varphi \to \vartheta
        \andalso
        \Gamma \vdash \inhabitant{\code{n}_2:} \psi \to \vartheta
        \andalso
        \inhabitant{\code{kind}: \code{K}{}_i\ \mathrm{in}\ \varphi, \psi}
    }
    {\Gamma \vdash \inhabitant{\code{switch (m.kind) \{ case K}{}_i\code{: return n}{}_i\code{(m); }\ldots \code{\}}:} \vartheta}}
]
</div><

[tex:\Ax \qquad][tex:\ExFalso]

[tex:\ImpI \qquad][tex:\ImpE]

[tex:\ConjI \qquad][tex:\ConjE]

[tex:\DisI]

[tex:\DisE]

[tex: {\scriptsize
    \left\{ \begin{array}{rcl}
        \ty|\code{(x:S) => T}| & = & \ty|\code{S}| \to \ty|\code{T}| \\
        \ty|\code{\[S, T\]}|   & = & \ty|\code{S}| \wedge \ty|\code{T}| \\
        \ty|\code{S | T}|     & = & \ty|\code{S}| \vee \ty|\code{T}| \\
        \ty|\code{never}|      & = & \bot \\
        \text{(otherwise)} \quad
        \ty|\code{T}|          & = & \code{T} \\
    \end{array} \right.
}]

この規則から式の部分を隠すと以下のようになります.

><div style="display:none">
[tex:
    \require{color}
    \definecolor{cover}{rgb}{0.9,0.9,0.9}
    \def\inhabitant#1{{\color{cover}#1}\;}
    \def\ty|#1|{{\color{cover}\lfloor} #1 {\color{cover}\rfloor}}
]
</div><

[tex:\Ax \qquad][tex:\ExFalso]

[tex:\ImpI \qquad][tex:\ImpE]

[tex:\ConjI \qquad][tex:\ConjE]

[tex:\DisI]

[tex:\DisE]

これは実は直観主義命題論理((古典命題論理(ふつうの論理)から二重否定除去のルールを除いた論理体系))の推論規則まったくそのままで, GentzenのNJと呼ばれる自然演繹体系です((もともとのGentzenの表記では⊦(ターンスタイル記号)を使わずに木構造の上の方に出てくる仮定を下の方で参照する形で, 論理学の教科書でもそちらの定義のしかたをよく見ると思いますが, 本質的には同じものです)).  自然演繹の体系では, 型付け規則でやったように, 推論規則を木構造につなげていって, すべての葉を[tex:\textsc{AX}]にできれば根の判断が成り立ちます.  (直観主義)命題論理の自然演繹なので, これらの規則を用いると「[tex:A]ならば[tex:A]」や「[tex:A]ならば[tex:B]で, かつ, [tex:B]ならば[tex:C]が成り立つなら, [tex:A]ならば[tex:C]も成り立つ」といった, 一般に成り立つ命題(トートロジーと言います)を導出できます.  逆に, この規則で導出できない命題はトートロジーではありません.  たとえば, いま挙げた2つの例は以下のように証明できます.

[tex:
    \infer(\to I)
    {\vdash A \to A}
    {\infer(AX){A \vdash A}{}}
]

[tex: {\tiny
    \infer(\to E)
    {
        \infer(\to I)
        {\vdash ({(A \to B)} \wedge (B \to C)) \to (A \to C)}
        {
            \infer(\to I)
            {(A \to B) \wedge (B \to C) \vdash A \to C}
            {(A \to B) \wedge (B \to C), A \vdash C}
        }
    }
    {
        \infer(\wedge E)
        {(A \to B) \wedge (B \to C), A \vdash B \to C}
        {
            \infer(AX){(A \to B) \wedge (B \to C), A \vdash (A \to B) \wedge (B \to C)}{}
        }
        \quad
        \infer(\to E)
        {(A \to B) \wedge (B \to C), A \vdash B}
        {
            \infer(\wedge E)
            {(A \to B) \wedge (B \to C), A \vdash A \to B}
            {
                \infer(AX)
                {(A \to B) \wedge (B \to C), A \vdash (A \to B) \wedge (B \to C)}
                {}
            }
            \quad
            \infer(AX){(A \to B) \wedge (B \to C), A \vdash A}{}
        }
    }
} ]

ところで, 「型は論理式」の話を思い出すと, 2つ目の例の[tex:({(A \to B)} \wedge (B \to C)) \to (A \to C)]は型でもあります.  では, この型のつく式はどういったものがあるでしょうか?  たとえば以下の式がそうです((他にも同じ型になる式はあります)).

>|typescript|
(x: [(a:A) => B, (b:B) => C]) => (a: A) => x[1](x[0](a))
||<

つまり,

[tex:{\scriptsize
\vdash \code{(x: \[(a:A) => B, (b:B) => C\]) => (a: A) => x\[1\](x\[0\](a))}: ({(A \to B)} \wedge (B \to C)) \to (A \to C)
}]

と主張しているわけです.  この主張が正しいかどうか確かめる(型検査する)には, 式を型付け規則にパターンマッチして, 根から葉へと導出木を作っていけばよいのでした.  まず式全体は関数になっているので[tex:\textsc{\to I}]の規則を使います.  次もまた関数なので[tex:\textsc{\to I}]を再び使います.  すると今度は関数適用なので[tex:\textsc{\to E}]を使います.  左の枝はタプルの要素へのアクセスなので[tex:\textsc{\wedge E}]を使います.

......といった具合にやっていくと, いま書いた命題論理での証明がそっくりそのまま復元されます.  型付け規則の結論の部分は([tex:\textsc{\bot E}]の規則をいったん忘れると)規則ごとにすべて異なるので, マッチする規則は毎回一つに決まります.  すると復元される内容全体も, 式が決まれば一つに決まります.  ということは, この式は命題論理での<strong>証明をTypeScriptの式の形にエンコードしたもの</strong>になっているのです.  これが「型のついた式は証明」の真相です.

Curry-Howard同型対応についてもっと詳しく知りたい人は以下の本を読みましょう.

[asin:B00OFROOBA:detail]

*** なぜ型ファースト?

ここまで分かればなぜ型ファーストなのかは明らかです.

>>
<span style="font-size: 200%">何を証明したいか決めずに<br>証明を書くヤツはいない</span>
<<

というだけです.

たとえば, 「素数が無限に存在することを証明せよ」と言われたら「よーし, 素数が無限に存在することを証明するゾ」と思って証明しますよね.  「ふと, ある数を階乗してみた.  ひとまず1を足してみた.  なんとなく素因数を求めてみた.  あれれぇ, 元の数が素数だとすると, この素因数はそれより大きいよね? これって素数が無限にあるってことじゃない?」なんて言ってなんの脈略もないところから突然なにかの証明を導き出してくる人はいないと思います.  もしかしたらいるかもしれませんが天才っぽいですね.  あまり真似できそうにありません.

プログラミングの話としては, テスト駆動開発との類似性を考えるとわかりやすいかもしれません.  テスト駆動開発では, まずは機能の要件(つまり仕様)を満たすことをチェックするためのテストを書き, 最初はテストが失敗する状態にしておき, テストを通す最低限のコードを書いて, 徐々に洗練させていきます.

型ファーストも同じ考えで, 仕様が何なのかをまず書いて, それに沿うコードを書いていくのです.  もし仕様そのものになんらかの不備があればそれを型として表現しようとした時点で不備に気づきやすくもなります.  闇雲に実装を書き始めるよりは, まずは仕様が何なのかはっきりさせましょう.

><aside class="column">
<!--
<h5><span class="only-in-toc">(コラム)</span> 本当に命題が先?</h5>

<p>数学の証明を書くとき, 本当に常に証明したい命題がなんなのかを先に書くのかと言うと, 厳密にはそうとは限りません.  命題を書いて証明を書き始めたものの, いざ証明してみるとその命題は満たせないとわかり, 何か前提を加えて主張を弱くすることもあります.  何か新しい概念を構築していて「こういうよい定理が成り立つ」と証明しようとしたものの, やってみると少し妥協した定理しか成り立たなかった場合です.  プログラミングで言えば, 当初掲げた強気な仕様は実装がたいへんとわかったので, 少し妥協したものに後から変える感じでしょうか.</p>

<p>大きな定理の証明のために補題を考えているときも「おそらくこういう補題があればよいはず」と思って書き始めたものの, その補題は成り立たない(もしくは証明が困難)とわかり, 定理を成り立たせるのに役立つ別の補題を考えないといけなくなることはありそうです.  そのときは途中まで書いた証明を見て「こういうことだったら証明できる」という発想で何を補題とするかを決めるかもしれません.  プログラミングで言えば, 大きなメソッドを実装するための補助的なプライベートメソッドを書いてみたらうまく実装できないとわかったので, プライベートメソッドの型は実装ありきでやりやすいものにしてしまうかもしれません.</p>

<p>とはいえ, いずれの場合も, いきなり証明を書いていくわけではなく「何を証明したいのか」を意識することには変わりありません.</p>
-->

<h5><span class="only-in-toc">(コラム)</span> コードはトップダウンに書く?</h5>

<p>型をまず書いてから実装をするなら, 事実上コードをトップダウンに書いていくことになります.  実装したいメソッドの型をまず書くと, 実装をまるごとぜんぶ書ききるまでコンパイルが通らないとなれば非常に不便です.  途中まで書いた段階でそこまでは型検査に通るかどうか確認しながら書いていきたいものです.</p>

<p>この不便を解消するために, [tex:\bot]型を積極的に使える言語があります.  たとえばScalaでは<code>Nothing</code>型が[tex:\bot]型に相当します.  そして<code>Nothing</code>型の式として<code>???</code>(実体は<code>NotImplementedError</code>の例外を投げるだけのメソッド)が用意されていて, どんな型が要求される場所にも書けます.  これはSTLCで言うと, [tex:\textsc{\bot E}]の規則で, 前提として[tex:\bot]型がついた式があったなら, 同じ式に任意の型[tex:\varphi]をつけてもよいとなっているためです.  型は決まっているがまだ実装を書いていない部分はひとまず<code>???</code>と書いておけばコンパイルでき, 既に実装を書いた部分の型が正しいかどうか確かめながら実装していけます.</p>
</aside><

*** 型は「ある種」の仕様

型によってある種の仕様を表現できましたが, どういう仕様を表現できるかは型の表現力次第です.  たとえば<code>sortBy</code>の例では, 結果の型がソート済みであることは型からはわかりませんでした.

>|scala|
Seq.sortBy: Seq[A] => (A => B) => Ordering[B] => Seq[A]
// - Aの列があり,
// - AをBに変換でき,
// - Bの順序が規定されていれば,
// - Aの列が得られる
||<

型の表現力は, 対応する論理体系の論理式の表現力です.  命題論理なら論理式に書けるの個々の要素は命題変数なので, 主語と述語の関係を表したりはできません.

ではもっとリッチな論理体系に対応する型システムにすればよいのかというと, 一概にそうとは言えません.  あまり表現力の高い型システムにしてしまうと, 型検査が決定不能になったり, 型推論ができなくなったりします.

よりリッチな論理体系とそれに対応する計算体系の研究はさまざま進んでおり, 一部は実際のプログラミング言語にもとり入れられています.  例をいくつか挙げておきます.

|*論理体系             |*計算体系や言語|
|古典論理              |STLC + 継続    |
|二階直観主義命題論理  |System F       |
|様相論理              |MetaOCaml      |
|線形論理(アフィン論理)|Rust           |

たとえば, 計算体系System Fは二階論理に基づいているので, [tex:\forall{\alpha}.(\alpha \to \alpha) \to \alpha \to \alpha]のような型が使えます(このような型を多相型と言います).  しかし, System Fで型検査を決定可能にするためには型を明示的に書かなければならない場合があり, 型推論の恩恵が減って不便になります.  なのでHaskellやMLといった実際のプログラミング言語では, System Fよりも少し制限された形の型システム(Hindley-Milner型システムと呼ばれるもの)を用いて, 多相型が使えて型推論も可能にしています.

仕様を細かく記述できる表現力の高い型システムにすることと, 型検査や型推論がうまくいくことの間にはトレードオフがあり, ここでもやはりよいバランスを追求していく必要があります.

*** ここまでのまとめ

- 型は論理式に, 型のついた式は証明に対応する
- 証明を書くときはまず命題から書く, だから実装を書くときはまず型から書く
- 型から書くのは仕様から書くのと同じ
- 仕様を型として細かく書けるとありがたいけど限界はある

** 型の表現を工夫する

型システムの表現力を高めるのには限界がありますが, さまざまな工夫によって仕様をうまく表現した型を考えることもできます.  (筆者の知る限りでは)この部分にとくに体系だったなにかがあるわけでもないので, 思いつくままにいくつか紹介します.  みなさんも「こんな工夫があるよ」というのがあれば是非それをご自身で紹介していってください.  「仕様としての型」をどんどん便利にしていきましょう.

思いつくままに書いたので例はすべてScalaです.

*** Scalaの<code>implicit</code>

Scalaには<code>implicit</code>という言語機能があります.  使い道はいろいろありますが, 「仕様としての型」の観点では, 使う側に不便を強いることなく必要とされる前提を表現できてたいへん便利です.

たとえば, <code>List[(A, B)]</code>を<code>Map[A, B]</code>に変換するところで<code>implicit</code>は使われています.

>|scala|
val l1: List[(Int, String)] = List(1 -> "foo", 2 -> "bar")
val m1 = l1.toMap // m1: Map[Int, String]
||<

ふつうですね.  これがよくできているのは, タプル以外を要素とする<code>List</code>は<code>Map</code>には変換できないところです.  「できない」のは, やるとコケるのではなく, そもそもコンパイルが通りません.

>|scala|
val l2: List[Int] = List(1, 2, 3)
val m2 = l2.toMap // コンパイルエラー
||<

これは一体どう実装されているのでしょうか?  素朴に<code>List[A]</code>を定義しようとすると, 行き詰まってしまいます.  <code>Map</code>の型引数になんと書いていいかわからなくなるからです.

>|scala|
class List[A] {
  def toMap: Map[???]
  ...
}
||<

こう書けばいいでしょうか??

>|scala|
class List[A] {
  def toMap[K, V]: Map[K, V]
  ...
}
||<

これだと, <code>A</code>の<code>List</code>をどこの馬の骨とも知れない<code>K</code>から<code>V</code>への<code>Map</code>に変換できてしまいます.  たとえば<code>List[Int]</code>を<code>Map[Int, String]</code>にしてしまうといった具合です.

そこで, <code>implicit</code>を使えば「<code>A</code>が<code>(K, V)</code>と互換性のある型である」前提を要求できます(<code>&lt;:&lt;</code>はScalaにあらかじめ定義されている型です)((Scalaの標準ライブラリのソースコードを見ていると, このような引数の名前は多くの場合"ev"になっていて, おそらく"<strong>ev</strong>idence"の略で, 「証拠」を要求しているわけですね)).

>|scala|
class List[A] {
  def toMap[K, V](implicit ev: A <:< (K, V)): Map[K, V]
  ...
}
||<

<code>implicit</code>な引数は, 呼び出されたコンテキストで静的に解決できる<code>implicit</code>な値があれば自動的に解決される(今回の場合は<code>&lt;:&lt;</code>の<code>implicit</code>値は標準で定義されていてどのコンテキストからでも解決されます)ので, <code>A</code>がなんらかのタプル型である限りは呼び出し側はとくに引数を渡す必要はありません.

*** スマートコンストラクタ

Scalaのリストは空の場合に<code>head</code>で先頭要素を取り出すと実行時エラー(例外)が発生します.

>|scala|
val l1 = List(1, 2, 3)
l1.head // => 1
val l2 = List()
l2.head // 実行時エラー
||<

こういうことがないように, 「空でないリスト」をなんとか型で表現したくなりますね.  しかし「<code>List[A]</code>のインスタンスであって, そして空でない」ことを論理として扱うのはけっこう大変です.

そこで, 「空でない」かどうかチェックするところは実行時に確かめるコードとして書いてしまって, チェック済みを表すタグとして型を使えば, 条件がいくら複雑になっても簡単に表現できます.  もちろん, そのタグの型はチェックに通ったときしかインスタンス化できないようにします.

>|scala|
class Nel[A] private (val v: List[A])
object Nel {
  def apply[A](v: A*): Option[Nel[A]] = fromList(v.toList)

  def fromList[A](l: List[A]): Option[Nel[A]] =
    if (l.nonEmpty) Some(new Nel(l))
    else None
}
||<

<code>Nel</code>クラスのコンストラクタは<code>private</code>なのでこのクラスの外で<code>new Nel</code>はできず, <code>Nel</code>のインスタンスを作るには<code>Nel.apply</code>か<code>Nel.fromList</code>を使うしかありませんが, これらは<code>Nel()</code>のように空で呼び出すと<code>None</code>が返ります.  <code>Some[Nel[A]]</code>のインスタンスが得られたときは必ず空ではないと保証されます((細かいことを言うとmutableなリストを与えられたらダメなので本来はimmutableに限定したり, 渡された時点でimmutableなリストに変換するなりすべきです)).

>|scala|
val nel1 = Nel(1, 2, 3)
nel1.map(_.v.head) // => Some(1)
val nel2 = Nel()
nel2.map(_.v.head) // => None
||<

*** 篩型 (refinement type)

スマートコンストラクタの例では, 実際にリストの操作をするときには<code>.v</code>を経由する必要があって面倒ですね.  できれば元の<code>List[A]</code>のインタフェースはそのままに, 空でない裏付けがタグとして付加された形にしたいものです.  まさにこれを実現する考え方が<ruby>篩<rt>ふるい</rt></ruby>型(refinement type)です.

篩型は, 交差型(intersection type)を用いて追加の制約を表現するアイディアです.  Scalaには交差型がある(<code>A with B</code>のような型)ので実現できます.  以下のように使うイメージです(定義はちょっとむずかしいので省いています).

>|scala|
val Some(nel) = RefinedNel(1, 2, 3) // nel: List[Int] with NonEmpty
nel.head                 // => 1
nel.map(n => n * n)      // => List(1, 4, 9)
nel.flatMap(n => List()) // => List()
||<

篩型版の<code>Nel</code>(<code>RefinedNel</code>)はスマートコンストラクタで空でない場合だけインスタンスを返すのは同じですが, <code>RefinedNel</code>のインスタンスを返すのではなく<code>List[A] with NonEmpty</code>のインスタンスを返します.  これはれっきとした<code>List[A]</code>型なので, <code>List[A]</code>のメソッドはそのまま呼べます(たとえば<code>head</code>や<code>map</code>, <code>flatMap</code>).

ただ, 一つ面倒な点があります.  空でない<code>List[A]</code>を<code>map</code>しても「空でない」ことは保たれるはずですが, 結果はただの<code>List[A]</code>になってしまって<code>NonEmpty</code>タグが外れてしまうので, 「空でない」情報が失われてしまいます.  裏側では篩型の概念を使いつつも非空性を保つかどうかを注意深く扱った拙作のライブラリがあるのでご利用ください.

[https://github.com/tarao/nonempty-scala:embed]

篩型は以下の論文で提案されました.

- [https://www.cs.cmu.edu/~fp/papers/pldi91.pdf:title=Refinement Types for ML].<br>Tim Freeman and Frank Pfenning.<br>In <span class="journal">Proceedings of the ACM SIGPLAN 1991 Conference on Programming Language Design and Implementation (PLDI '91)</span>, pages 268-277, New York, NY, 1991.

ScalaやHaskellでは"refined"という名前のライブラリとして提供されています.

- [https://github.com/fthomas/refined:title=Scala]
- [https://github.com/nikita-volkov/refined:title=Haskell]

*** 高カインド型

カインドは, 型コンストラクタの型のことです.  型コンストラクタとは「型に適用して型を得るもの」のことで, たとえば<code>List[_]</code>は, <code>Int</code>に適用して<code>List[Int]</code>型を得る型コンストラクタです.  「高カインド型」(がある言語でサポートされている)とは, 型(パラメータ)として型コンストラクタを渡せるという意味です.

例としては以下のように, 型コンストラクタをふつうの型と同様に型パラメータにできます.

>|scala|
trait ApplyToString[CC[_]] {
  type Result = CC[String]
}
val stringList: ApplyToString[List]#Result = List("a", "b", "c")
val stringArray: ApplyToString[Array]#Result = Array("a", "b", "c")
||<

これ自体は型システムの表現力の問題になりますが, サポートしている言語は(多くはないものの)そこそこあるのでうまくすれば大きなデメリットなしに実現できるはずです.  Haskell, Scala, C++, Rustあたりでは使えると思います.

高カインド型にはさまざまな応用がありますが, たとえば圏論的な概念を素直に表現できます((たとえば, 関手(functor)はすべて型コンストラクタで(も)あると思え, 自然変換(natural transformation)を表現するには「関手を受け取って何かする」必要があるため)).  例として, Scala標準のタプル型(<code>Tuple2</code>)と, 独自に定義した直積型(<code>|*|</code>)のどちらであっても同じように要素を取り出せるメソッド<code>π1</code>と<code>π2</code>を定義して, <code>Tuple2</code>と<code>|*|</code>を相互変換する場合を見てみましょう(完全なコードは[https://gist.github.com/tarao/80d52dff89cbbc6bdf0e07bc26b0af54:title=こちら]).

>|scala|
val p1 = |*|(1, "foo") // 独自定義の直積型
// => p1: Int |*| String = <1, foo>

π1(p1)
// => res0: Int = 1

π2(p1)
// => res1: String = foo

val p2 = ("bar", 2)    // Scala標準のタプル
// => p2: (String, Int) = (bar,2)

π1(p2)
// => res2: String = bar

π2(p2)
// => res3: Int = 2

p1.toProduct[Tuple2]
// => res4: (Int, String) = (1,foo)

p2.toProduct[|*|]
// => res5: String |*| Int = <bar, 2>
||<

<code>π1</code>と<code>π2</code>の定義は以下のようになっていて, なんらかの型コンストラクタ<code>P[_, _]</code>に対して, それを2項の直積として扱える証拠<code>Product2[P]</code>があれば, 要素を取り出せる定義になっています.

>|scala|
def π1[A, B, P[_, _]](p: P[A, B])(implicit product: Product2[P]): A =
  product.p1(p)

def π2[A, B, P[_, _]](p: P[A, B])(implicit product: Product2[P]): B =
  product.p2(p)
||<

<code>toProduct</code>の方も, <code>Tuple2</code>や<code>|*|</code>といった具体的な型コンストラクタに言及しない定義になっています.

>|scala|
implicit class WedgeOps[W, A, B](val w: W)(implicit
  wedge: Wedge[W, A, B]
) {
  def toProduct[P[_, _]](implicit
    product: Product2[P],
  ): P[A, B] = product.mediate[A, B, W](wedge)(w)
}
||<

これらは以下の本を社内で輪読したときにささっと書いたものですが, 高カインド型がなかったらコードで表現できるとは思わなかったでしょう.

[asin:0262660717:detail]

逆に, 圏論的なものをコードに落とし込めれば強力な抽象概念を記述でき, 「仕様としての型」の幅がだいぶ広がります.

** おわりに

計算ファーストは自由を制限して安全を得る考え方, 型ファーストは仕様から先に考えるやり方でした.  まず仕様を考えるやり方にすればあまり「自由を制限されている」気持ちにならずに済みます.  とはいえ仕様を型として表現するには限界もあるので, うまく工夫していきましょうという話でした.

<hr>

そういえば, ちょうど10年前にもラムダ計算の話を書いていました.  またラムダ計算の話してる......

[https://tarao.hatenablog.com/entry/20100208/1265605429:embed]

このときは「最速マスター」の流行に乗る都合もあって, 記述はなるべく少なく, 意義はともかく最速で説明する内容でした.  今回は対照的に, 言葉を尽くして形式的な概念をプログラミングするときの心構えにつなげる内容になりました.  10年でいろいろ成長して(歳をとって)見えてきたからこういう話も書けるようになったのでしょうか.  でも, ここに書いたことは10年前には知っていたことばかりなので何も変わってないとも言えますね.  「最速マスター」では型なしラムダ計算の話しかしなかったので, 10年越しに続きを書いた感じもします.
