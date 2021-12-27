---
Title: 再帰的な構造のデータの同値性判定はどうしたらいいか
Category:
- lang
- javascript
- scala
- ruby
Date: 2021-08-23T12:44:56+09:00
URL: https://tarao.hatenablog.com/entry/deep-equality
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/26006613798684934
---

数日前にTwitterで, JavaScriptのオブジェクトに対する<code>===</code>の挙動が初心者には難しいみたいな話を見かけた.  発端や周辺の議論をちゃんと追いかけてないからとくに出典は貼らない.  たぶん元々の話は「へぇ, こういう挙動なんだ, 簡単ではないね」くらいの話だったのかもしれない.  自分のタイムラインの観測範囲では「そうだそうだ, (参照の同一性ではなく)同値性にしとけばいいのに」と思っている人もそれなりにいそうに見えた.

個人的にも同値性が簡単に確認できるとよい気はするものの, 「なんでそうしないんだ, オブジェクトの中身を確認していくだけだろ!」みたいな簡単な話ではないことも知っているため, 以下のようなツイートをしたのだった.

[https://twitter.com/oarat/status/1427584381825081346:embed]
[https://twitter.com/oarat/status/1427585597967724549:embed]

オブジェクトの同値性を判定するメソッドや演算子を実装しようとすると, 再帰的な構造があると困ってしまう((それ以外にも, 同値性判定に計算コストがだいぶかかってしまう(効率よく計算できるようにするのがたいへん)といった問題もある)). なぜなら循環している可能性があるから.  同値性を判定できないと主張したいわけではなくて「同値性をどう定義したらいいか自明ではない」という話.

====
この記事では, オブジェクトの参照が一致するかを「(参照の)同一性」と呼び, 値が一致するかを「同値性」と呼んで区別する.

** 同値性の定義のパターン

どういうふうに同値性を定義したらよいか自明ではない根拠として, いくつかパターンを書いておく.  他にも変な定義はいくらでも考えられるとは思うものの, ある程度納得のいくものはこのくらいのバリエーションではないだろうか.

*** パターン1: 循環のない範囲でのみ同値性を定義する

たとえば, JSONとして有効な範囲内では同値性の判定が可能で, それを逸脱するものに対する判定結果は定義しない, というやり方.  定義として成立はしているものの, なに気なく書いたコードが未定義動作の可能性があって, 字面からはまったく予測できない場合もあるため, 利用者側としては嬉しくない. 個人的には絶対やめてほしい.

*** パターン2: 循環していたら止まらないのを許す

未定義ではないが無限ループすることがある, とするパターン.

実は同値性判定がこの仕様になっている言語もある.  たとえばScala(の<code>case class</code>や標準ライブラリのコレクション)がそう.  Scalaはもともと<code>==</code>で(同一性ではなく)同値性を判定できるが, これを循環構造を持つ値に対してやると<code>==</code>の計算が止まらない.

循環を作るにはいくつかやり方があるが, ふつうにやっていてできるものではなくて<code>var</code>とか<code>lazy val</code>といった黒魔術的なものを使う必要がある.  <code>var</code>の場合はたとえばこういう感じ.

>|scala|
sealed trait List[+A]
case class Cons[A](car: A, var cdr: List[A]) extends List[A]
case object Nil extends List[Nothing]

val l1 = Cons(1, Nil)
l1.cdr = l1

val l2 = Cons(1, Cons(1, Nil))
val l3 @ Cons(_, _) = l2.cdr
l3.cdr = l2
||<

<code>lazy val</code>はより魔術性が高く, 見た感じ簡単に書ける.

>|scala|
sealed trait List[+A]
case class Cons[A](car: A, cdr: List[A]) extends List[A]
case object Nil extends List[Nothing]

lazy val l1: Cons[Int] = Cons(1, l1)
lazy val l2: Cons[Int] = Cons(1, Cons(1, l2))
||<

どちらの場合も<code>l1 == l2</code>は止まらず, <code>var</code>でやった場合はスタックオーバーフローになり, <code>lazy val</code>でやった場合は結果が返ってこない.

Scalaの場合, もう一つ別な方法もあって, <code>LazyList</code>を使うと循環構造に限らず任意の無限リストを実現できる.  循環構造の場合はたとえばこう.

>|scala|
val l1: LazyList[Int] = 1 #:: l1
val l2: LazyList[Int] = 1 #:: 1 #:: l2
||<

この場合もやはり<code>l1 == l2</code>は結果が返ってこない.

<code>LazyList</code>は循環なしの無限リストも表現できて, 自然数全体とかすべての素数を並べたリストみたいなものを作ることもできる.

>|scala|
// 自然数全体
val n = LazyList.from(0)

// すべての素数
val p = {
  def sieve(l: LazyList[Int]): LazyList[Int] = l match {
    case head #:: tail => head #:: sieve(tail.filter(_ % head != 0))
  }
  sieve(LazyList.from(2))
}
||<

これらは値の種類がそもそも無限にあるから有限の計算で同値性を判定することが(ふつうは)できない.  ここまでくるとそもそも「データ」とは言えない(値を取り出すときに計算が起こる)から, 止まらないのもやむなし.  おそらく, 一般にはこういう場合もあるからあえて<code>==</code>が必ず止まるような仕様にはしていないのだと思う.  逆に, <code>var</code>も<code>lazy val</code>も<code>LazyList</code>も使っていない, 副作用のない純粋なデータであれば<code>==</code>は止まるはずで, 純粋なデータかどうかは静的にわかるから構わないだろう, ということではないか.  そして<code>var</code>や<code>lazy val</code>や<code>LazyList</code>はそう滅多に使うものでもない.  僕だったら, 意味なくこれらを使っているコードがあったらレビューでリジェクトする.

一方で, JavaScriptではオブジェクトが副作用の影響を受けうるかどうかは(ソースコード全体をスキャンしない限りは)静的にはわからないだろうし, 破壊的代入は日常的に発生している.  普段から同値性判定が止まらないかもしれないと思いながら暮らしたくはない.

あと, 同値性を数学的な「同値関係」だと思うと, 計算が止まらない値に関しては, 関係が成り立つかどうか未定義であるとするしかなさそう.  であれば, パターン1と同様に(静的に区別できるのでなければ)やめてほしい((たとえば, PythonのdictはScalaと同様に, 循環する場合は==が無限ループするようだけど, 動的型付けの言語でこれは正直やめてほしい)).

*** パターン3: 同じ循環の仕方の場合のみ同値と見なす

アイディアとして, 循環する場合もなんとかして循環のない場合の同値性に帰着させられないか考えてみる.

たとえば

>|json|
x = { a: { a: x } }
y = { a: { a: y } }
||<

の<code>x</code>と<code>y</code>が同値かどうかで考えてみると, 自己参照しているところを<code>●</code>で潰すとどちらも

>|json|
{ a: { a: ● } }
||<

となり一致する(循環を考慮しない範囲の同値性で一致を確認できる).  一方,

>|json|
x = { a: x }
y = { a: { a: y } }
||<

の場合は,

>|json|
x = { a: ● }
y = { a: { a: ● } }
||<

となり一致しない(という定義にしようとしている).

相互再帰になっている場合, たとえば

>|json|
x = { a: { a: y } }
y = { a: { a: x } }
||<

のような場合も, 自己参照がくるまで展開したらよさそうに一見おもえる.  この場合はどちらも

>|json|
{ a: { a: { a: { a: ● } } } }
||<

になる.  しかし実はそれではダメで,

>|json|
x = { a: { a: y } }
y = { a: { a: y } }
||<

の場合に<code>x</code>の方は自己参照なしで延々と<code>y</code>が展開され続けることになる.  実際には「自己参照のところを潰す」のではなく「既に通ったオブジェクトへの参照がきたら潰す」としなければならない.

さらに, ここまでの議論だと「どのオブジェクトが潰されたか」を区別した方がよい感じがする.  たとえば

>|json|
x = { a: { a: y }, b: x }
y = { a: { a: x }, b: y }
||<

のときは

>|json|
(1){ a: { a: (2){ a: { a: ●(1) }, b: ●(2) } }, b: ●(1) }
||<

みたいにしないといけないはず((再帰的な参照をしている部分をなんらかの束縛関係のようなものだと思ってDe Bruijn indexのようなものを計算している, あるいはα同値性を見ている, みたいにも見えるがそう言って伝わる人には言わなくてもわかると思うのでそちらの方向の説明はしない)).

結局これは何をしているかというと, 通ったオブジェクトを状態, オブジェクトのキーを遷移と思ったときの状態遷移グラフを描いている.  グラフの(始点を揃えたときの)形が一致するかどうかが同値かどうかを決める.  たとえば最後の例の状態遷移グラフは次のようになる.

<!--
digraph state_transition {
  graph[rankdir=LR];
  node[shape=circle];

  node0[label="x"];
  node2[label="y"];

  { rank=same;
    node1[label=""];
    node3[label=""];
  }

  node0->node1[label="a"];
  node0->node0[label="b"];
  node1->node2[label="a"];
  node2->node3[label="a"];
  node2->node2[label="b"];
  node3->node0[label="a"];
}
-->
<img src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDIuNDAuMSAoMjAxNjEyMjUuMDMwNCkKIC0tPgo8IS0tIFRpdGxlOiBzdGF0ZV90cmFuc2l0aW9uIFBhZ2VzOiAxIC0tPgo8c3ZnIHdpZHRoPSIyMDJwdCIgaGVpZ2h0PSIxMDRwdCIKIHZpZXdCb3g9IjAuMDAgMC4wMCAyMDIuMDAgMTA0LjAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIj4KPGcgaWQ9ImdyYXBoMCIgY2xhc3M9ImdyYXBoIiB0cmFuc2Zvcm09InNjYWxlKDEgMSkgcm90YXRlKDApIHRyYW5zbGF0ZSg0IDEwMCkiPgo8dGl0bGU+c3RhdGVfdHJhbnNpdGlvbjwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0idHJhbnNwYXJlbnQiIHBvaW50cz0iLTQsNCAtNCwtMTAwIDE5OCwtMTAwIDE5OCw0IC00LDQiLz4KPCEtLSBub2RlMCAtLT4KPGcgaWQ9Im5vZGUxIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMDwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9IjE4IiBjeT0iLTQ1IiByeD0iMTgiIHJ5PSIxOCIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxOCIgeT0iLTQxLjMiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+eDwvdGV4dD4KPC9nPgo8IS0tIG5vZGUwJiM0NTsmZ3Q7bm9kZTAgLS0+CjxnIGlkPSJlZGdlMiIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+bm9kZTAmIzQ1OyZndDtub2RlMDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTExLjI2NjQsLTYyLjAzNzNDOS44OTIyLC03MS44NTc5IDEyLjEzNjcsLTgxIDE4LC04MSAyMS42NjQ2LC04MSAyMy45MTU1LC03Ny40Mjg5IDI0Ljc1MjksLTcyLjM1MjkiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjI4LjI1MjQsLTcyLjAzMDcgMjQuNzMzNiwtNjIuMDM3MyAyMS4yNTI0LC03Mi4wNDM5IDI4LjI1MjQsLTcyLjAzMDciLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTgiIHk9Ii04NC44IiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiIGZpbGw9IiMwMDAwMDAiPmI8L3RleHQ+CjwvZz4KPCEtLSBub2RlMSAtLT4KPGcgaWQ9Im5vZGUzIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMTwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9Ijk3IiBjeT0iLTcyIiByeD0iMTgiIHJ5PSIxOCIvPgo8L2c+CjwhLS0gbm9kZTAmIzQ1OyZndDtub2RlMSAtLT4KPGcgaWQ9ImVkZ2UxIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMCYjNDU7Jmd0O25vZGUxPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMzUuMTQxOCwtNTAuODU4NkM0NS4zOTA2LC01NC4zNjEzIDU4LjYzMzYsLTU4Ljg4NzQgNzAuMjM1MywtNjIuODUyNiIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iNjkuMjk3MSwtNjYuMjMwNiA3OS44OTE2LC02Ni4xNTI4IDcxLjU2MSwtNTkuNjA2OCA2OS4yOTcxLC02Ni4yMzA2Ii8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjU3LjUiIHk9Ii02Mi44IiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiIGZpbGw9IiMwMDAwMDAiPmE8L3RleHQ+CjwvZz4KPCEtLSBub2RlMiAtLT4KPGcgaWQ9Im5vZGUyIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMjwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9IjE3NiIgY3k9Ii00NSIgcng9IjE4IiByeT0iMTgiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTc2IiB5PSItNDEuMyIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIiBmaWxsPSIjMDAwMDAwIj55PC90ZXh0Pgo8L2c+CjwhLS0gbm9kZTImIzQ1OyZndDtub2RlMiAtLT4KPGcgaWQ9ImVkZ2U1IiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMiYjNDU7Jmd0O25vZGUyPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMTY5LjI2NjQsLTYyLjAzNzNDMTY3Ljg5MjIsLTcxLjg1NzkgMTcwLjEzNjcsLTgxIDE3NiwtODEgMTc5LjY2NDYsLTgxIDE4MS45MTU1LC03Ny40Mjg5IDE4Mi43NTI5LC03Mi4zNTI5Ii8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIxODYuMjUyNCwtNzIuMDMwNyAxODIuNzMzNiwtNjIuMDM3MyAxNzkuMjUyNCwtNzIuMDQzOSAxODYuMjUyNCwtNzIuMDMwNyIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxNzYiIHk9Ii04NC44IiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiIGZpbGw9IiMwMDAwMDAiPmI8L3RleHQ+CjwvZz4KPCEtLSBub2RlMyAtLT4KPGcgaWQ9Im5vZGU0IiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMzwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9Ijk3IiBjeT0iLTE4IiByeD0iMTgiIHJ5PSIxOCIvPgo8L2c+CjwhLS0gbm9kZTImIzQ1OyZndDtub2RlMyAtLT4KPGcgaWQ9ImVkZ2U0IiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMiYjNDU7Jmd0O25vZGUzPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMTU4Ljg5MTYsLTM5LjE1MjhDMTQ4LjY0OTQsLTM1LjY1MjMgMTM1LjQwNzgsLTMxLjEyNjcgMTIzLjgwMjYsLTI3LjE2MDQiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjEyNC43MzY0LC0yMy43ODA4IDExNC4xNDE4LC0yMy44NTg2IDEyMi40NzI1LC0zMC40MDQ2IDEyNC43MzY0LC0yMy43ODA4Ii8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEzNi41IiB5PSItMzUuOCIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIiBmaWxsPSIjMDAwMDAwIj5hPC90ZXh0Pgo8L2c+CjwhLS0gbm9kZTEmIzQ1OyZndDtub2RlMiAtLT4KPGcgaWQ9ImVkZ2UzIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMSYjNDU7Jmd0O25vZGUyPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMTE0LjE0MTgsLTY2LjE0MTRDMTI0LjM5MDYsLTYyLjYzODcgMTM3LjYzMzYsLTU4LjExMjYgMTQ5LjIzNTMsLTU0LjE0NzQiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjE1MC41NjEsLTU3LjM5MzIgMTU4Ljg5MTYsLTUwLjg0NzIgMTQ4LjI5NzEsLTUwLjc2OTQgMTUwLjU2MSwtNTcuMzkzMiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzYuNSIgeT0iLTYyLjgiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+YTwvdGV4dD4KPC9nPgo8IS0tIG5vZGUzJiM0NTsmZ3Q7bm9kZTAgLS0+CjxnIGlkPSJlZGdlNiIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+bm9kZTMmIzQ1OyZndDtub2RlMDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTc5Ljg5MTYsLTIzLjg0NzJDNjkuNjQ5NCwtMjcuMzQ3NyA1Ni40MDc4LC0zMS44NzMzIDQ0LjgwMjYsLTM1LjgzOTYiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjQzLjQ3MjUsLTMyLjU5NTQgMzUuMTQxOCwtMzkuMTQxNCA0NS43MzY0LC0zOS4yMTkyIDQzLjQ3MjUsLTMyLjU5NTQiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNTcuNSIgeT0iLTM1LjgiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+YTwvdGV4dD4KPC9nPgo8L2c+Cjwvc3ZnPgo=" />

これを見れば, グラフをコピーしてひっくり返して重ねると<code>x</code>と<code>y</code>が一致することがわかる(グラフの形が<code>x</code>と<code>y</code>でちょうど対称になっていることが一目でわかる).

きちんと定義できているし, 一見これで問題ない.  ただ個人的には少し気に入らない.  この定義だとオブジェクト(の参照)が1対1に対応している場合しか同値と見なさないため「ある2つのオブジェクト(の参照)が同一か否か」の区別が定義に表れてしまっている.  値が一致しているかどうかで判定したいはずが, 同一性を気にしてしまっている.  具体例で言うと, 途中で挙げたこの例

>|json|
x = { a: { a: y } }
y = { a: { a: y } }
||<

は, 状態遷移グラフを描くと以下のようになり

<!--
digraph state_transition {
  graph[rankdir=LR];
  node[shape=circle];

  node0[label="x"];
  node2[label="y"];

  { rank=same;
    node1[label=""];
    node3[label=""];
  }

  node0->node1[label="a"];
  node1->node2[label="a"];
  node2->node3[label="a"];
  node3->node2[label="a"];
}
-->
<img src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDIuNDAuMSAoMjAxNjEyMjUuMDMwNCkKIC0tPgo8IS0tIFRpdGxlOiBzdGF0ZV90cmFuc2l0aW9uIFBhZ2VzOiAxIC0tPgo8c3ZnIHdpZHRoPSIyMDJwdCIgaGVpZ2h0PSIxMDlwdCIKIHZpZXdCb3g9IjAuMDAgMC4wMCAyMDIuMDAgMTA5LjAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIj4KPGcgaWQ9ImdyYXBoMCIgY2xhc3M9ImdyYXBoIiB0cmFuc2Zvcm09InNjYWxlKDEgMSkgcm90YXRlKDApIHRyYW5zbGF0ZSg0IDEwNSkiPgo8dGl0bGU+c3RhdGVfdHJhbnNpdGlvbjwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0idHJhbnNwYXJlbnQiIHBvaW50cz0iLTQsNCAtNCwtMTA1IDE5OCwtMTA1IDE5OCw0IC00LDQiLz4KPCEtLSBub2RlMCAtLT4KPGcgaWQ9Im5vZGUxIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMDwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9IjE4IiBjeT0iLTgzIiByeD0iMTgiIHJ5PSIxOCIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxOCIgeT0iLTc5LjMiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+eDwvdGV4dD4KPC9nPgo8IS0tIG5vZGUxIC0tPgo8ZyBpZD0ibm9kZTMiIGNsYXNzPSJub2RlIj4KPHRpdGxlPm5vZGUxPC90aXRsZT4KPGVsbGlwc2UgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBjeD0iOTciIGN5PSItODMiIHJ4PSIxOCIgcnk9IjE4Ii8+CjwvZz4KPCEtLSBub2RlMCYjNDU7Jmd0O25vZGUxIC0tPgo8ZyBpZD0iZWRnZTEiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPm5vZGUwJiM0NTsmZ3Q7bm9kZTE8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik0zNi4zMjI4LC04M0M0NS44OTQ3LC04MyA1Ny44MjIzLC04MyA2OC41NjI5LC04MyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iNjguNzc2NiwtODYuNTAwMSA3OC43NzY2LC04MyA2OC43NzY2LC03OS41MDAxIDY4Ljc3NjYsLTg2LjUwMDEiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNTcuNSIgeT0iLTg2LjgiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+YTwvdGV4dD4KPC9nPgo8IS0tIG5vZGUyIC0tPgo8ZyBpZD0ibm9kZTIiIGNsYXNzPSJub2RlIj4KPHRpdGxlPm5vZGUyPC90aXRsZT4KPGVsbGlwc2UgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBjeD0iMTc2IiBjeT0iLTQ1IiByeD0iMTgiIHJ5PSIxOCIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxNzYiIHk9Ii00MS4zIiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiIGZpbGw9IiMwMDAwMDAiPnk8L3RleHQ+CjwvZz4KPCEtLSBub2RlMyAtLT4KPGcgaWQ9Im5vZGU0IiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub2RlMzwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgY3g9Ijk3IiBjeT0iLTE4IiByeD0iMTgiIHJ5PSIxOCIvPgo8L2c+CjwhLS0gbm9kZTImIzQ1OyZndDtub2RlMyAtLT4KPGcgaWQ9ImVkZ2UzIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMiYjNDU7Jmd0O25vZGUzPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMTY1LjkyNjMsLTI5Ljg0NjlDMTU5LjU2ODQsLTIxLjcxMjYgMTUwLjU2ODQsLTEyLjQ2MjYgMTQwLC04IDEzNC43MDQ0LC01Ljc2MzkgMTI4LjgyMSwtNS42OTYzIDEyMy4xNDg3LC02LjcxODUiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjEyMi4wMDM1LC0zLjQwNTIgMTEzLjMzOTEsLTkuNTAyNiAxMjMuOTE0OCwtMTAuMTM5MyAxMjIuMDAzNSwtMy40MDUyIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEzNi41IiB5PSItMTEuOCIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIiBmaWxsPSIjMDAwMDAwIj5hPC90ZXh0Pgo8L2c+CjwhLS0gbm9kZTEmIzQ1OyZndDtub2RlMiAtLT4KPGcgaWQ9ImVkZ2UyIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub2RlMSYjNDU7Jmd0O25vZGUyPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMTEzLjM2ODksLTc1LjEyNjRDMTI0LjA2NzIsLTY5Ljk4MDMgMTM4LjI2MSwtNjMuMTUyOSAxNTAuNDMxMiwtNTcuMjk4OSIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMTUyLjE1NzYsLTYwLjM1MjQgMTU5LjY1MjEsLTUyLjg2MzUgMTQ5LjEyMzMsLTU0LjA0NDIgMTUyLjE1NzYsLTYwLjM1MjQiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTM2LjUiIHk9Ii02OC44IiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiIGZpbGw9IiMwMDAwMDAiPmE8L3RleHQ+CjwvZz4KPCEtLSBub2RlMyYjNDU7Jmd0O25vZGUyIC0tPgo8ZyBpZD0iZWRnZTQiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPm5vZGUzJiM0NTsmZ3Q7bm9kZTI8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik0xMTQuMTQxOCwtMjMuODU4NkMxMjQuMzkwNiwtMjcuMzYxMyAxMzcuNjMzNiwtMzEuODg3NCAxNDkuMjM1MywtMzUuODUyNiIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMTQ4LjI5NzEsLTM5LjIzMDYgMTU4Ljg5MTYsLTM5LjE1MjggMTUwLjU2MSwtMzIuNjA2OCAxNDguMjk3MSwtMzkuMjMwNiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzYuNSIgeT0iLTM2LjgiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCIgZmlsbD0iIzAwMDAwMCI+YTwvdGV4dD4KPC9nPgo8L2c+Cjwvc3ZnPgo=" />

<code>x</code>と<code>y</code>は同値ではないということになる.  なぜなら, <code>x</code>の方は<code>x.a.a</code>と辿ると<code>x</code>とは別のオブジェクトが現れるのに対して, <code>y</code>の方は<code>y.a.a</code>すると<code>y</code>自身が返るから, ということになる.  これはつまり<code>x</code>と<code>y</code>の参照は異なるという話をしてしまっている.

別な言い方をしてみる.  循環のない

>|json|
x = { a: { a: 1 } }
y = { a: { a: 1 } }
||<

だったら<code>x</code>と<code>y</code>は同値ということで誰も異論はないだろう.  しかし<code>1</code>の部分が他のオブジェクトへの参照になった途端, 指している先は同じ<code>y</code>なのに, それが自分自身を指しているか否かで同値かどうかが変わってしまう.  これは奇妙.

定義として間違っているわけではないが, 奇妙な定義だと言ってよさそう.

*** パターン4: なるべく多くのものを同値と見なす

パターン3の最後の方で見たこの例

>|json|
x = { a: { a: y } }
y = { a: { a: y } }
||<

は参照の同一性を考えないなら, 同値と見なしてもまったく問題ないように思える.  <code>x</code>も<code>y</code>も, どちらも<code>a</code>のキーを辿るとずっと<code>a</code> → <code>a</code> → <code>a</code> → …と無限に続く.  こういった場合も含めて「同値と見なして問題ないものはすべて同値と見なす」ようにできないものだろうか?  実はできる.

いったん, 循環のない場合の同値性をあらためて考え直してみる.  たとえば,

>|json|
x = { a: { a: 1 }, b: 2 }
y = { a: { a: 1 }, b: 2 }
||<

が同値かどうか判定するには<code>x.a</code>と<code>y.a</code>, および<code>x.b</code>と<code>y.b</code>が同値かどうか見ることになる.  <code>x.b</code>と<code>y.b</code>はプリミティブ値なのでそのまま値が等しいか比較し, <code>x.a</code>と<code>y.a</code>はオブジェクトなのでまたその中のキーを辿った先が同値かどうか見る.  数学的にはこう定義できる.

><div style="display:none">
[tex:\def\objeq{\equiv_{\textrm{obj}}}]
</div><

><div class="definition">
<h6>定義 : 循環のないオブジェクトの同値性</h6>

循環のないオブジェクト[tex:x], [tex:y]が同値である([tex:x \objeq y]と書く)とは, 以下を満たすことを言う.

- [tex:x]中の任意のキー[tex:k]について, [tex:y.k]が存在し, [tex:x.k \objeq y.k]が成り立つ
- [tex:y]中の任意のキー[tex:k]について, [tex:x.k]が存在し, [tex:y.k \objeq x.k]が成り立つ

ただし, プリミティブ値[tex:p_1], [tex: p_2]は[tex:p_1 = p_2]のとき[tex:p_1 \objeq p_2]であるものとし, 配列などオブジェクト以外の非プリミティブ値はないものとする.
</div><

どうにかして, この定義を元に循環するオブジェクトに対しても使えるものを作れないだろうか?  実は, この定義はそっくりそのまま循環するオブジェクトに対して適用してもなんの齟齬もない.  ただ循環する場合は選択肢があって, たとえば,

>|json|
x = { a: x }
y = { a: y }
||<

の場合, これを同値としてもしなくても, 上に書いた定義の条件に当てはまっている.  なぜなら, 同値とした場合は<code>x.a</code>に対して<code>y.a</code>は存在するし, それぞれの指す先は<code>x</code>と<code>y</code>そのもので, たったいま同値ということにしたばかりのものだから条件を満たす.  同値としない場合, <code>x.a</code>の指す先の<code>x</code>は<code>y.a</code>の指す先の<code>y</code>と同値ではないから, <code>a</code>については条件を満たさず, <code>x</code>と<code>y</code>は同値ではないという結論になり辻褄が合う.

どちらでもよいのなら同値ということにしよう.  その方が, 循環している場合も, 辿ったキーの並びが同じものは同値ということにできる.  こうやって「同値ということにしても定義上の齟齬がないものはすべて同値とする」という定義にしてしまおう((数学的にきちんと言うと「このような条件を満たす関係のうち最大のものをとる」ということになり, 本来はそのような最大のものが存在することを証明しなければならない)).

こう定義すれば, 循環のないオブジェクトに関しては従来通りの同値性となる.  循環する場合は, 同値関係の両辺とも循環し, かつキーを辿ったパスが同じ場合に同値と判定されることになる.  どちらか片方が循環しない場合は循環のないオブジェクトの同値性の条件に反する(循環しない方はプリミティブ値になるのにもう片方はオブジェクトになる).  どちらも循環する場合は同じキーのパスを辿れさえすれば, キーの存在の条件を満たせるから齟齬はない.

いくつか例を見よう.

>|json|
x = { a: x }
y = { a: { a: y } }
||<

まず<code>x</code>も<code>y</code>も<code>a</code>のキーを持つので<code>x.a</code>と<code>y.a</code>を見る.  <code>x.a</code>は循環しているが<code>y.a</code>はまだ循環していないからまた<code>a</code>を辿る.  <code>x.a.a</code>は(再び)循環して<code>y.a.a</code>も循環する.  ここまで辿ってきたパスは同じ.  どちらも循環しているからこの先どれだけ辿っても同じパスが現れる.  だから同値でよい.

>|json|
x = { a: y, b: 1 }
y = { a: { a: y, b: 1 }, b: 1 }
||<

まず<code>x.b</code>と<code>y.b</code>は値が一致する.  <code>x.a</code>と<code>y.a</code>はどちらもオブジェクトだからその先を辿る.  <code>x.a.b</code>と<code>y.a.b</code>は値が一致する. <code>y.a.a</code>は循環しているが<code>x.a.a</code>はまだ循環していないから先を辿る.  <code>x.a.a.b</code>と<code>y.a.a.b</code>は値が一致する.  <code>x.a.a.a</code>も<code>y.a.a.a</code>も循環していて, パスも一致している.  だから同値でよい.

この定義なら変に参照の同一性を気にすることもなく, 値がどうなっているかだけを見て同値性を判定できているように思える.

ちなみに, Rubyはどうやらこの定義に相当する同値性判定を実装している.  偉い.

>|ruby|
x = { a: nil, b: 1 }
y = { a: { a: nil, b: 1 }, b: 1 }
x[:a] = y
y[:a][:a] = y
x == y
# => true
||<

** 理論的背景: 双模倣による定義

実は, パターン4で見た循環のないオブジェクトの同値性の定義は, オブジェクトを状態, キーを状態遷移と思ったとき((すべてのプリミティブ値もある唯一の状態への遷移と見なす))の状態遷移システムの双模倣性([https://en.wikipedia.org/wiki/Bisimulation:title=bisimulation])の定義になっている.  そして「同値にしても齟齬がないものは同値とする」として広げていったものは双模倣関係((日本語がこれでよいか少し自信がなくて, bisimilarなxとyがあったとき日本語では「xとyは双模倣である」と言うところまではよさそうだけど, このような関係のことをなんと言うのか(英語だとbisimilarity)がちょっとよくわかっていないし, 「双模倣関係」は「双模倣性」(これも関係)とたいへんまぎらわしい(英語のbisimilarityとbisimulationもたいがいまぎらわしいが) ))(bisimilarity relation)になっていて, これは自動的に同値関係(反射律・対称律・推移律を満たす関係)になる.

こうやって作られた関係は, オブジェクトの同一性を最も粗く区別し, 同時にプリミティブ値はきちんとすべて区別するものになっている.

>>
Proposition 1 yields insight in the concept of deep equality: deep equality is the
equivalence relation which makes the fewest possible distinctions among oids while at the
same time distinguishing among all different basic values such that objects and their
values are identified. Moreover the reader familiar with the theory of communication and
concurrency will have noticed the analogy with the observational equivalence concept of
<em>strong bisimilarity</em> [Mil89].
<<

[https://link.springer.com/chapter/10.1007%2F3-540-60608-4_42:title=Deep equality revisited].<br>Serge Abiteboul and Jan Van den Bussche.<br>In <span class="journal">Deductive and Object-Oriented Databases (DOOD 1995)</span>, Singapore, 1995.

先述の同値性のバリエーションはすべて双模倣性(bisimulation)にもなっている(はず).  パターン4(双模倣(bisimilarity))はその中でも最大のもので, パターン3は最大でも最小でもない半端なものだということもわかる.  「奇妙」に思えたのはこの半端さに由来する.

双模倣はもともと, 状態遷移システムや(並行)プロセス計算体系で, 観測上の振る舞いが一致することを数学的に扱うために考え出されたもの.  だから同じ概念でデータの「値としての振る舞い」の同一性を判別できることも頷ける.  もしこの辺りのことをもっと詳しく知りたいなら以下の本を読むといいだろうか.

[asin:0521658691:detail]

** 双模倣による同値性判定の実装例

実は10年くらい前に趣味で作っていたJavaScriptのライブラリ((GitHubに載せていなかったのでこの機会にサルベージしてきたもので, このライブラリはけっきょく(風呂敷を広げすぎて)作りかけのまま放置していて日の目を見ることはなかった))で, [https://github.com/tarao/gnn.js/blob/5deb0f7f81020c26f13570cf135afea560524f41/gnn/base.js#L343-L364:title=双模倣に基づいて同値性を判定するメソッドを実装していた]ので紹介しておく.

>https://github.com/tarao/gnn.js/blob/5deb0f7f81020c26f13570cf135afea560524f41/gnn/base.js#L279-L291>
>|javascript|
        sim: function(lhs, rhs, seen, axiom, transition) {
            if (!axiom.call(this, lhs, rhs)) return false;

            // detect cyclic reference
            if ((seen=seen.clone()).add(lhs, rhs)) return true;

            var states = transition.call(this, lhs, rhs);
            for (var i=0; i < states.length; i++) {
                var l = states[i][0]; var r = states[i][1];
                if (!this.sim(l, r, seen, axiom, transition)) return false;
            }
            return true;
        }
||<
<<

基本的な流れは, <code>transition</code>が遷移先を集めてきて, その各遷移先について<code>axiom</code>(今回の場合はプリミティブ値の場合に一致するか, 片方だけ遷移先が欠けていたりしないか)が成り立つかどうかを確認する.  どちらもオブジェクトに辿りついて, 循環していたら(既に見たオブジェクトだったら)そのパスは一致したことにして探索を打ち切る.  循環していなければ再帰的にさらなる遷移先を調べていく.

循環しているかどうかの判定は, 見たオブジェクトを愚直に保存していって, 毎回線形にナメて既出かどうか調べているので効率は悪い.  いまだったら<code>WeakMap</code>を使うとか, なんらかもっと良いやりようがありそう.

現実世界の言語処理系の実装ではどうか.  たとえばRubyの処理系もだいたい同じことをしていそう.  <a href="https://docs.ruby-lang.org/ja/latest/class/Hash.html">Rubyの<code>Hash</code></a>の同値性は<code>eql_i</code>という関数を再帰的に呼ぶことによって確認していそうで, その際<code>recur</code>というパラメータをチェックするガードが書かれていて, <code>true</code>を返すようになっている.

>https://github.com/ruby/ruby/blob/5e9598baea97c53757f12713bacc7f19f315c846/hash.c#L3712>
>|c|
static VALUE
recursive_eql(VALUE hash, VALUE dt, int recur)
{
    struct equal_data *data;

    if (recur) return Qtrue;	/* Subtle! */
    data = (struct equal_data*)dt;
    data->result = Qtrue;
    rb_hash_foreach(hash, eql_i, dt);

    return data->result;
}
||<
<<

この<code>recur</code>がどこで真になるかと言うとこのあたり:

>https://github.com/ruby/ruby/blob/2d4f29e77e883c29e35417799f8001b8046cde03/thread.c#L5190>
>|c|
    VALUE pair_list = rb_hash_lookup2(list, obj, Qundef);
    if (pair_list == Qundef)
	return Qfalse;
    if (paired_obj_id) {
	if (!RB_TYPE_P(pair_list, T_HASH)) {
	    if (!OBJ_ID_EQL(paired_obj_id, pair_list))
		return Qfalse;
	}
	else {
	    if (NIL_P(rb_hash_lookup(pair_list, paired_obj_id)))
		return Qfalse;
	}
    }
    return Qtrue;
||<
<<

再帰呼び出しで渡してまわるペアに対して, どちらも以前に見たリストに含まれていれば<code>true</code>が返るようになっていそう.  12年前のパッチによってサポートされた模様.

[https://bugs.ruby-lang.org/issues/1448:embed]

このように, 双模倣による同値性判定の実装そのものはそれほど難しくはない.

** まとめ

- 循環するオブジェクトも考慮した同値性の定義はさまざまなバリエーションが考えられる
- その中でも同値になるものが最も多くなる定義を採用すると自然に思える
- 自然と思える根拠は双模倣という概念により説明できる
-- ありうる定義の中で最大のものである
-- 自動的に同値関係になる
-- 双模倣という概念がそもそも観測上の振る舞いの一致を数学的に表現するためのものである
- 双模倣による同値性判定は, そんなに難しいところもなく実装できる
- Rubyはこの意味の同値性判定を実装していて偉い
