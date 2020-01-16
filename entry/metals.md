---
Title: 'LSP時代のScala開発環境: Metals, Bloop (on Emacs / lsp-mode)'
Category:
- scala
- emacs
Date: 2019-06-27T21:27:40+09:00
URL: https://tarao.hatenablog.com/entry/metals
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/17680117127209727440
---

これまでScalaでの開発には[http://ensime.github.io/:title=ENSIME]を使ってきたけど, もうそろそろ頃合いだとおもうので[https://scalameta.org/metals/:title=Metals]に乗り換えた.  エディタ側でLSPのサポートが充実してきているのでこれはだいぶ簡単で, さっくり乗り換えることができた.

Metalsはビルド部分は裏側で[https://scalacenter.github.io/bloop/:title=Bloop]を使っているので, テストの実行なんかもこれに乗っかるとだいぶ楽になる.  けどEmacsからBloopを利用するにはまだちょっと面倒なところもあったので, この際いろいろ整備してみた.

====
[https://cdn-ak.f.st-hatena.com/images/fotolife/t/tarao/20190627/20190627211424.png:image]

** Metals + Bloop

MetalsはまぁふつうにScalaのlanguage serverという感じだけど, ENSIMEと比べると以下の点が強力(個人的な視点).

- コンパイラがホンモノなのでエラーを誤検知しない<br>(従来の開発環境では)
-- presentation compilerと呼ばれる, ちょっとインチキしてるやつだった
-- 誤検知で真っ赤になって困っていた
- [https://scalameta.org/docs/semanticdb/guide.html:title=SemanticDB]を使っていてソースコードの情報を高速に扱える
-- ENSIMEもlucene使ってたりして頑張ってたけどね...
- ビルドまわりはBloopを使っているので他との連携がスムーズ
-- エディタにハイライトするためにコンパイルが終わってれば結果を流用できる
--- つまりをテストを実行したりREPLを開くときにコンパイルを待たなくてよい!
-- 自分でビルド情報が必要なツール作りたくなっても比較的かんたん

** Metalsでできること・できないこと

(個人的な視点で欲しかったものベースなので網羅性はない)

*** できる

- 型情報やコンパイルエラーの情報の表示
- ドキュメントやメソッドパラメータの表示
- 定義へのジャンプ
- 補完
- シンボルの利用箇所の列挙

*** できない

- テストの実行やREPLの起動
-- Bloopの守備範囲
- リファクタリング機能
-- [https://github.com/scalameta/metals-feature-requests/issues/18:title=#18 Refactoring support]
- 使われてる<code>implicit</code>の表示
-- [https://github.com/scalameta/metals-feature-requests/issues/21:title=#21 Show information derived from code]
- Javaのコードへ飛んだときにもいい具合になる
-- [https://github.com/scalameta/metals-feature-requests/issues/5:title=#9 Interoperability with Java language servers]
- 抽象メソッドの実装コードの挿入
-- [https://github.com/scalameta/metals-feature-requests/issues/25:title=#25 Automatic implementation for abstract methods in concrete classes]

激しく必要な機能は揃っていて, 残りもissue化されていてやる気はありそうなので今後に期待!

** Emacsで使う

基本的には[https://scalameta.org/metals/docs/editors/emacs.html:title=公式の設定方法]に従ったらよいだけ.  とはいえ面倒な部分もあったので<strong>自分でいろいろ整えた</strong>.

*** MetalsとBloopをインストールするのすら面倒

どこにインストールするかとか考えたくもない.  バージョンアップするときどのファイルを置き換えたらいいかも考えたくない.  そこで, Emacs側で設定してあれば裏で勝手にダウンロードしてきてBloopサーバも起動しといてくれるようにするやつを作った.

[https://github.com/tarao/scala-bootstrap-el:embed]

>|lisp|
(require 'scala-bootstrap)
(require 'lsp-mode)

(add-hook 'scala-mode-hook
          '(lambda ()
             (scala-bootstrap:with-metals-installed
              (scala-bootstrap:with-bloop-server-started
               (lsp)))))
||<

という感じにすると<code>scala-mode</code>でいますぐMetals生活が始められる(実際に使ってる設定は[https://github.com/tarao/dotfiles/blob/ed5ffa0cd4a4f4324d425bc68c291583c8757832/.emacs.d/init/scala.el:title=こういうかんじ]).  最新版にするときは<code>M-x scala-bootstrap:reinstalll-metals</code>とかでイケる.  カスタム変数を設定すればバージョン固定もできる.

*** Bloopを使うのが面倒

コマンドラインで<code>bloop test --only \*<var>MySpec</var> <var>myproject</var></code>みたいにするのはめんどくさすぎる.  プロジェクト内のファイルを編集中になにかてきとうなコマンドを叩いたらさっと実行してほしい.  <code>sbt-mode</code>ではそういうのができた.

調べてみると[https://github.com/tues/emacs-bloop:title=emacs-bloop]というのがあるけど, だいぶ雑で<code>require</code>できるようにすらなってないし, <code>bloop console</code>には対応していなかった.  ということでforkして足りないところを補った.

[https://github.com/tarao/emacs-bloop:embed]

<code>README.md</code>までちゃんとする余力はなかったので[https://github.com/tarao/dotfiles/blob/4b772a0c29585c9a933f38b7feb3c0eb1f53523f/.emacs.d/init/evil.el#L359-L364:title=このへん]の設定から使い方を感じとってほしい.

がんばったところとしては<code>bloop console</code>を<code>comint-mode</code>でうまく扱えるようにいろいろしてるところ.  具体的には以下のあたり.

- 色指定以外の[https://en.wikipedia.org/wiki/ANSI_escape_code:title=ANSIエスケープコード](CSI)もやってくるので無視する
- プロンプトになぜかスペースが一つ余計につくので除く
- 複数行入力を適切に扱う
- エコーバックされてくると二重に出ちゃうのでなんとかする

副作用としてプロンプトをEmacs側で任意にカスタマイズできるようになった.

** 感想

めちゃくちゃ快適.  定義にジャンプするときとか早い気がするし, 都度都度コンパイルされ続けているので改めてコンパイル待ちする機会はほぼ皆無. ブランチ切り替えて依存が変わったときとかも, インポートしなおしのフローがなんかわかりやすい.  [https://github.com/emacs-lsp/lsp-ui:title=lsp-ui]よくできてる.  もう戻れない.

** 追記 2019-06-29

[https://2019.scalamatsuri.org/:title=ScalaMatsuri 2019]のアンカンファレンスで「ScalaをEmacs + Ensimeで開発するノウハウを教えてほしい」というのがあったので, 「いや, そこはもうMetalsを使っていきましょう」という話をさせてもらった.  EmacsでScalaを書いていると珍しがられて「物好き」のレッテルを貼られてしまうけど, 仲間が10人くらいはいた.  このまま, EmacsでScala書くのぜんぜんいけるけど? という感じを出していきたい.
