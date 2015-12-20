---
Title: ' Evil: EmacsをVimのごとく使う - 導入編'
Date: 2013-03-03T00:00:00+09:00
URL: http://tarao.hatenablog.com/entry/20130303/evil_intro
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/6653586347149235984
---



><blockquote class="epigraph" cite="http://starwars.com/explore/the-movies/episode-vi/" title="Star Wars: Episode VI - Return of the Jedi">
  You underestimate the power of the dark side.
</blockquote><

EmacsはLispで自由自在に拡張でき, エディタの枠におさまらず, コンピュータ上でのあらゆる創造的活動のための環境として発達してきました. しかし, 少なくともファイルを閲覧し編集するという操作に関しては, vi/Vimが非常に優れたインタフェースであることもまた事実です. 両者はそれぞれが根強いファンを抱え, 長らく宗教戦争を繰り返してきました.

この対立が止揚された結果として生まれたのが<a href="http://emacswiki.org/emacs/Evil">Evil</a>です. Emacsのなんでもありな環境の上でVimをエミュレートすることで, EmacsでありながらVimの操作性を実現したのです.

本稿では, Evilとは何かということに始まり, 具体的な導入方法について解説します.
=====
><div class="toc float">
  <h4>目次</h4>
  <ol>
    <li>
      <strong>導入編</strong>
      <ul>
        <li><a href="#history">歴史</a></li>
        <li>
          <a href="#design">設計思想</a>
          <ul>
            <li><a href="#design-extensibility">高い拡張性の確保</a></li>
            <li><a href="#design-interoperability">Emacsの機能とうまく共存</a></li>
          </ul>
        </li>
        <li>
          <a href="#install">インストール</a>
          <ul>
            <li><a href="#install-novice">とにかくお試し</a></li>
            <li><a href="#install-release">リリース版のインストール</a></li>
            <li><a href="#install-enable">起動時に有効化</a></li>
            <li><a href="#install-devel">開発版のインストール</a></li>
          </ul>
        </li>
        <li>
          <a href="#usage">使い方</a>
          <ul>
            <li><a href="#usage-emacs">Emacsからの移民もしくはvi/Vim初心者</a></li>
            <li><a href="#usage-vim">Vimからの移民</a></li>
          </ul>
        </li>
        <li><a href="#support">サポート等</a></li>
      </ul>
    </li>
    <li>[http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]</li>
    <li>[http://d.hatena.ne.jp/tarao/20130305/evil_ext:title=拡張編]</li>
    <li>[http://d.hatena.ne.jp/tarao/20130306/evil_appendix:title=付録]</li>
  </ol>
</div><

><!--
|2012-03-03|evil_intro|
|2012-03-04|evil_config|
|2012-03-05|evil_ext|
|2012-03-06|evil_appendix|
--><

><h4 id="history">歴史</h4><

Evilについての理解を深めるためには, その前にずっと続いてきた開発の歴史について触れずにはいられません.

Emacsにおけるvi/Vimエミュレーションの歴史は, GNU Emacsの最初の公開リリースの翌年, 1986年((これは<code>vip.el</code>のコメントとして書かれた年であって, 初期の開発はもっと前に始まっていた可能性もあります))まで遡ります. 現在は[http://openlab.jp/skk/index-j.html:title=日本語入力システムSKK]の作者として知られる佐藤雅彦先生((筆者の元指導教員のため「先生」の敬称を使わせていただきます))によって, viの操作性をエミュレートする[http://www.gnu.org/software/emacs/manual/html_node/vip/:title=VIP]が開発されました. 佐藤先生がEmacsを愛用していらっしゃるのは, EmacsがLispインタプリタを備えているから(そしておそらくインタラクティブUIをLispで実装する環境として優れているから)((現在もご自身の<a href="http://www.fos.kuis.kyoto-u.ac.jp/~masahiko/papers/ez.pdf">計算と論理に関する理論の実装をEmacs上で</a>やっていらっしゃいます))ですが, UIの操作性, とくにCtrlキーを多用して指が疲れるという点はあまり好ましく思っていなかったようで((これは御本人から酒の席で伺ったことなので, 正確に意図を汲み取れたかどうかは自信がありません)), これはEmacsよりもvi/Vimを好む人たちにとっては常套句かと思います. けれど, Emacsが指の痛くなる馬鹿げたやり方に固執していたと考えるのは早計です. Richard Stallmanのもとに佐藤先生より<code>vip.el</code>が送られ, 興味深いことに, [http://cvs.savannah.gnu.org/viewvc/emacs/lisp/emulation/vip.el?revision=1.1&root=emacs&view=markup:title=1988年にはVIP 3.5がEmacsの標準パッケージとして取り込まれた]ので, 彼は少なくともこのような代替的な操作方法が提供されることを疎ましく思っていたわけではないのでしょう.

佐藤先生の<code>vip.el</code>はその後も標準パッケージとしてメンテナンスされ続けますが, それとは別に[https://groups.google.com/forum/?fromgroups=#!topic/gnu.emacs.help/dRU9YQHFUSI:title=1991年にAamod SaneによってVIP 4系統に改良され], 1993年頃まで開発が続きます. さらに, これらの実装を元に1994年にMichael KiferによってVIP-19((Emacs 19に対応したVIPという意味のようです))が作られ, 1995年に[http://www.gnu.org/software/emacs/manual/html_node/viper/:title=Viper]という名前でふたたび[http://cvs.savannah.gnu.org/viewvc/emacs/lisp/emulation/viper.el?revision=1.1&root=emacs&view=markup:title=Emacsの標準パッケージに取り入れ]られます. この後しばらく, Emacsをviキーバインドで操作するにはViperを使うという時代が続きますが, Viperはあくまでviのエミュレーションであるため, Vimの機能性に差をつけらていくこととなります. [http://web.archive.org/web/20080210045550/http://www.geocities.jp/emacsjjjj/viper/:title=2005年ごろからViperにオブジェクト単位の選択やビジュアルモードの機能を追加するパッチを当てるなどの試み]もなされましたが, Viperの機能が本質的にVimに追いつくことはありませんでした.

2006年になって, Brad Beveridgeが, Viperを拡張してVimに近づけるプロジェクト[http://common-lisp.net/project/vial/darcs/extended-viper/:title=extended-viper]を立ち上げ, Alessandro Pirasのvimper((<a href="http://www.emacswiki.org/emacs/Vimpulse">EmacsWiki: Vimpulse</a>の情報で, オンライン上にはないようです))に引き継がれた後, 2007年にJason Spiroによって[http://www.emacswiki.org/emacs/Vimpulse:title=Vimpulse]というプロジェクトになります. その後, メンテナが2009年にAlessandro, 2010年にVegard &Oslash;yeに交代しつつ, VimpulseはVimの主要な機能を実装し, 完成度を高めていきます. これらはすべて, 標準添付のViperには直接手を加えず, 追加機能の差分だけ実装していました. しかし, Viperのコードはレガシーで, 新しい機能と衝突する部分も多く, <a href="https://gitorious.org/vimpulse/vimpulse/blobs/master/vimpulse-viper-function-redefinitions.el"><code>advice</code>や関数の再定義でViperのコードを修正する</a>こと(いわゆるモンキーパッチ)も多くなり, メンテナのVegardは, Viperをコアとして継ぎ足していくやり方に限界を感じはじめます.

これらの流れとは全く別に, 2009年からFrank Fischerによって[http://www.emacswiki.org/emacs/VimMode:title=vim-mode]が開発されます. vim-modeの目的は, Viperに頼らないことで拡張性の高いVimエミュレーションを提供することでしたが, Frankは個人的に日常のワークフローで必要な機能やユーザから要望のあった機能を優先的に実装していたため, 決して十分な完成度が達成できているとは思っていなかったようです.

2011年2月に, Vegardが[https://gitorious.org/vimpulse/vimpulse/commit/124786f7639e8396eeff9e77bf36202214e48f34:title=新たなVimpulseのコアを実装]しはじめ, Viperへの依存をなくすことにしたのを受けて, FrankがVegardにVimpulseとvim-modeの[https://lists.ourproject.org/pipermail/implementations-list/2011-February/000694.html:title=2つのプロジェクトの合流を提案]します. そして, 同じ目的のパッケージをそれぞれ独立に実装してきた知見を活かし, より拡張性が高く, Emacsに無理なく統合できるようなやり方でコア部分を再設計し, 残りの部分はVimpulseとvim-modeの実装を取り入れることでできるだけ多くのVimの機能をカバーしていく, ということで合意します. こうして, 2011年3月にスタートしたのが[http://emacswiki.org/emacs/Evil:title=Evil]プロジェクトです. Evilは新しいコアを使ってVimの操作性に関する主要な機能を実装し終え, 2013年2月に[https://lists.ourproject.org/pipermail/implementations-list/2013-February/001775.html:title=バージョン1.0.0がリリース]されました.

f:id:tarao:20130207142509p:image

><h4 id="design">設計思想</h4><

Evilの目的は, できる限り多くのVimの機能をエミュレートすることですが, ただやみくもにVimをエミュレートするのではなく, 過去の失敗に学んだ設計思想に沿った実装となっています. この設計思想と相容れない使い方をしようとすればただ不満が募るばかりで, ユーザにとっても開発者にとっても不幸なことです.

><h5 id="design-extensibility">高い拡張性の確保</h5><

Evilが<b>e</b>xtensible <b>vi</b> <b>l</b>ayerの略であることからもわかるように, Vimpulseやvim-modeを捨てた理由, あるいはViperをコアとすることをやめて一から実装することになった理由は, 高い拡張性を確保するためです. この拡張性というのは, 将来あらたな機能を増やしたり, プラグインの作成を容易にするという面もありますが, どちらかといえばVimの豊富な機能を実装していくにあたり, 開発の後期に破綻しないためです.

Viperはviのエミュレーションのために作られたこともあって, Vimpulseの開発末期にはViperがコアであることがかえって不都合なことがしばしばあり, 欠けているVimの機能を実装するコストが大幅に増していきました. ViperはVimの機能を想定して開発されたわけではないので当然と言えば当然です. それでも高い完成度を達成できていたことは賞賛に値すると思います.

このような経緯から, EvilではVimの様々な機能を視野に入れて, それぞれの機能がなるべく独立に実装できるように, コア部分のコンセプトを明確にし, またできる限りコアを小さくするという目標が最初に掲げられました. コアを小さく保つことの一つの帰結として, Emacsにも備わっている機能はできるだけそのまま使い, EvilはあくまでEmacsとのインタフェースの差異を吸収することに注力する, ということが挙げられます.

><h5 id="design-interoperability">Emacsの機能とうまく共存</h5><

Emacsに備わっている機能をそのまま使えることはそれ自体に意味があり, またEvilが作られた理由でもあります.

たとえば, Vimpulseの初期のバージョンでは, ビジュアルモードがEmacsのリージョンとは別に実装されていたために, Vimpulseの選択範囲にEmacsのリージョンコマンドを適用したり, EmacsのリージョンにVimpulseのオペレータを適用するということができませんでした. これでは何のためにわざわざEmacsを使っているのかわかりません.

VimpulseとEvilのユーザであるTim Harperによりメーリングリストに投稿された次の文章は, Evilの目指すところをよく言い表していると思います((この投稿に対して開発者のVegardとFrankも同意の旨を返信しています)).

>https://lists.ourproject.org/pipermail/implementations-list/2011-March/000714.html:title=implementations-list - Vimpulse and vim-mode from Tim Harper>
if it's a choice between interoperability with Emacs at the cost of 97% compatibility with vim, and 100% compatibility with them at the cost of reduced interoperability with Emacs commands, I would prefer sacrificing the vim compatibility.

If 100% vim compatibility were important to me, I would use vim.

:(拙訳):Emacs(の機能)と相互運用できてVimとの互換性が97%になってしまうのと, 100%互換な代わりにEmacsのコマンドが使えないのと, もしどちらかを選ぶなら, 互換性を犠牲にする方がいい. もし100%の互換性が大事なら, Vimを使えばいい.
<<

Emacs本来の機能を利用できるようにするには, 大きくわけて2つのアプローチが考えられます.

+ Vimのあらゆる機能を独自に実装し, Emacsのすべての機能に対してVim風のインタフェースを用意する
+ 設計をEmacsと親和性の高いものにしておく(Emacsの機能はEmacsのやり方で使えるままにしておく)

もし, Vimとの互換性にこだわるなら, 前者を選ぶことになるでしょう. 前述の例で言えば, Emacsのすべてのリージョンコマンドに対して対応するオペレータを用意しておくということです. しかし, Emacsの標準パッケージは膨大であり, それらはすべてEmacsの哲学で実装されています. ひとくちに「Vim風のインタフェースを用意する」と言っても, 単にキーバインドを入れ替えればいい場合もあれば, リージョンコマンドとオペレータのように近い概念に置き換える必要があるもの, そもそも使い方のレベルで再設計が必要なものもあるでしょう. また, 非標準のパッケージにも何らかの形で対応しなければなりません. これらのすべてのコストをEvilの開発者が負うのは事実上不可能です. このような理由から, Evilでは後者のアプローチが採用されました.

とくに元々Vimを使っていたユーザからの不満の声として多いと感じるのは, Evilを使っていると何かの拍子にEmacsくささが姿を見せるという点です. 彼らはこれをEvilの欠陥であると見ているのかもしれませんが, これは意図されたことだと言えます.

それでも開発者のFrankは, この設計思想の下にユーザからの不満をすべて退けるつもりはなく, Timの投稿に対しても次のように述べています.

>https://lists.ourproject.org/pipermail/implementations-list/2011-March/000721.html:title=implementations-list - Vimpulse and vim-mode from Frank Fischer>
I completely agree that we should not focus on 100% vim compatibility (even sometimes Vim does things wrong IMO). We should just be prepared that sometimes incompatibilities or inconveniences may arise that are no problem for one user but are important for the other (and this with a good reason as the example above illustrates). So if the effort is not too big we should try to be as close as possible to vim without overemphasizing this goal.

:(拙訳):100%の互換性に注力すべきでない(し, 個人的にはときにVimのやり方は間違っている((たとえば, このメールの少し前のやりとりで, Vimの繰り返し処理のやり方では本質的に扱えないものがあることが<a href="https://lists.ourproject.org/pipermail/implementations-list/2011-February/000706.html">指摘</a>されています))と思うことさえある)ことには完全に同意する. それでも, なにか非互換だったり不便だったりする部分が, 誰かにとって問題ないことでも, 他の誰かにとってはとても重大な問題かもしれないということは覚悟しておかないといけない. だから, 労力の許す限り, できるだけVimに近づけて, あまりこのこと(訳注:互換性にこだわらないこと)を強調しすぎない方がいい.
<<

実際, ユーザからの要望があれば, Vim風の使い方を維持できるように調整する場合があります. まずは要望を出してみて, 開発者にとって荷が重いようであれば, ユーザ自ら調整部分をプラグインとして実装することで貢献するというのが望ましいと思います.

><h4 id="install">インストール</h4><

それではEvilをインストールしてみましょう. Evilは(いまのところ)標準パッケージではないので, 自分でインストールする必要があります. ここではインストール方法と, Evilを有効化する設定について説明します. それ以外の細かい設定については[http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]を見て下さい.

><h5 id="install-novice">とにかくお試し</h5><

ここでは, 真っ新なEmacs環境にEvilをインストールして有効化する方法を解説します. 次のいずれかに当てはまる人は, この手順に従うとよいでしょう.
- Emacsをはじめて使う人
- 自分のEmacsの設定を汚さずに試してみたい人
- 後述のインストール方法がよくわからなかった人

まず, 準備としてEmacs本体(バージョン24以降)がインストールされていることを確認して下さい.

次に, 以下のように書いたファイルを作り, <code>init.el</code>というファイル名でてきとうに新しく作ったディレクトリに保存しましょう.
>|lisp|
;; Emacs directory
(when load-file-name
  (setq user-emacs-directory (file-name-directory load-file-name)))

;; Package management
(require 'package)
(add-to-list 'package-archives
             '("marmalade" . "http://marmalade-repo.org/packages/"))
(package-initialize)

(defun package-install-with-refresh (package)
  (unless (assq package package-alist)
    (package-refresh-contents))
  (unless (package-installed-p package)
    (package-install package)))

;; Install evil
(package-install-with-refresh 'evil)

;; Enable evil
(require 'evil)
(evil-mode 1)
||<

続いて, <code>init.el</code>を保存したディレクトリで以下のコマンドを実行すると, Emacsの既存の設定を汚すことなく, Evilをインストールして試すことができます. はじめて実行したときは<strong>必要なファイルをダウンロードするため時間がかかります</strong>.
>||
emacs -q -l init.el
||<

Emacsを使うときに常にEvilを使うようにするには, <code>~/.emacs.d/init.el</code>に上記の設定を書きます. (ただし, 既にEmacsを使っていて<code>~/.emacs</code>に設定を書いている場合は, <code>~/.emacs.d/init.el</code>に移動する必要があります.) これで,
>||
emacs
||<
とするだけで, Evilが有効な状態のEmacsが起動するようになります.

以上の手順に従った場合は, これ以降のインストール方法を読む必要はありません.

><h5 id="install-release">リリース版のインストール</h5><

Emacs 24以降もしくは<a href="http://repo.or.cz/w/emacs.git/blob_plain/1a0a666f941c99882093d7bd08ced15033bc3f0c:/lisp/emacs-lisp/package.el"><code>package.el</code></a>を入れたEmacs 23が必要です. まず設定ファイルに以下のように書きます.
>|lisp|
(require 'package)
(add-to-list 'package-archives
             '("marmalade" . "http://marmalade-repo.org/packages/"))
(package-initialize)
||<
Emacsを起動したら, <code>M-x package-install RET evil RET</code> (<code>M-x</code>は<code>Alt</code>と<code>x</code>の同時押し)と入力します. これでEvilがインストールされます.

Evilを現在のEmacsセッションで有効にするには, <code>M-x evil-mode RET</code>とします.

><h5 id="install-enable">起動時に有効化</h5><

設定ファイルに以下のように書くと, 次回起動時にはEvilが自動的に有効になります.
>|lisp|
(require 'evil)
(evil-mode 1)
||<

><h5 id="install-devel">開発版のインストール</h5><

リリース版ではなく開発版をインストールするには, 3つの方法があります.
- [http://melpa.milkbox.net/#development:title=MELPA]を使う方法
- [https://github.com/dimitri/el-get:title=El-Get]を使う方法
- 手動

MELPAは, Emacs標準のパッケージ管理の仕組みを使うもので, インストール方法やビルド方法はパッケージ提供者側((必ずしもパッケージの開発者ではなく, パッケージを登録し, バージョンアップに追従する作業をやっている人のことです))が決めたものに従い, パッケージ提供者によって登録されたバージョンがインストールされます. El-Getは, ユーザが自らパッケージのダウンロード元, ビルド方法などをレシピという形で指定する((EvilのレシピはEl-Getそのものに含まれています))パッケージ管理方法で, Emacs標準のやり方ではありませんが自由度が高く玄人向きです.

MELPAを使う場合は, リリース版と同様で, <code>package-archives</code>に指定するURLが異なるだけです. すなわち,
>|lisp|
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "http://melpa.milkbox.net/packages/"))
(package-initialize)
||<
とすれば, あとは<code>M-x package-install RET evil RET</code>でインストールされます.

El-Getを使う場合は, まず, [https://github.com/dimitri/el-get:title=El-Getのドキュメント]の手順に従ってEl-Getをインストールする設定を書きます. Evilのインストールそのものは, 設定ファイルで<code>el-get</code>関数を呼ぶようにするか, あるいはEmacs起動後に<code>M-x el-get-install RET evil RET</code>とします.
>|lisp|
;; El-Get
(add-to-list 'load-path (locate-user-emacs-file "el-get/el-get"))
(unless (require 'el-get nil 'noerror)
  (with-current-buffer
      (url-retrieve-synchronously
       "http://raw.github.com/dimitri/el-get/master/el-get-install.el")
    (goto-char (point-max))
    (eval-print-last-sexp)))

;; Install evil
(el-get-bundle evil)
||<
El-Getでのインストールには, <code>git</code>コマンドと<code>make</code>コマンドが必要です. デフォルトでinfoファイルも生成するようになっているので, [http://www.gnu.org/software/texinfo/:title=Texinfo](の<code>makeinfo</code>コマンドと<code>texi2pdf</code>コマンド)も必要です. もしinfoファイルの生成が不要なら,
>|lisp|
;; El-Get
(add-to-list 'load-path (locate-user-emacs-file "el-get/el-get"))
(unless (require 'el-get nil 'noerror)
  (with-current-buffer
      (url-retrieve-synchronously
       "http://raw.github.com/dimitri/el-get/master/el-get-install.el")
    (goto-char (point-max))
    (eval-print-last-sexp)))

;; Install evil
(el-get-bundle evil :info nil)
||<
とすることで, Texinfoがなくてもインストールできます.

手動でインストールするには, Evilと[http://www.emacswiki.org/emacs/UndoTree:title=Undo Tree]をリポジトリから<code>git clone</code>して, それぞれのディレクトリを<code>load-path</code>に追加します. リポジトリのURLは以下の通りです. バイトコンパイルしたい場合は, Undo Treeは手動(<code>emacs --batch -f batch-byte-compile undo-tree.el</code>)で, Evilは<code>make</code>します.
:Evil:git://gitorious.org/evil/evil.git
:Undo Tree:http://www.dr-qubit.org/git/undo-tree.git

><h4 id="usage">使い方</h4><

Evilを実際に使うには, EmacsとVimについてどれくらい知っているかによって, 知るべきことが変わってきます.

><h5 id="usage-emacs">Emacsからの移民もしくはvi/Vim初心者</h5><

f:id:tarao:20130207150701p:image:right

Evilには<strong>ステート</strong>と呼ばれる状態((Vimではこれをモードと呼びますが, Emacsには既に(メジャー/マイナー)モードの概念があるので, 混同しないためにステートと名付けられています))があります. 起動時にはノーマルステートになっていて, モードラインに<code>&lt;N&gt;</code>と表示されます. ノーマルステートは, 主に移動やコマンドの実行のための状態で, 文字を打ってもそのまま入力されません. 文字を入力するには<code>i</code>, <code>a</code>, <code>o</code>等のキーを入力して挿入ステート(<code>&lt;I&gt;</code>と表示されます)に移る必要があります. 挿入ステートで<code>ESC</code>キーを押すとノーマルステートに戻ります.

Emacsに慣れ親しんでいて, 万が一操作方法がよくわからなくなってしまった場合は, <code>C-z</code>でEmacsステートに移ることができます. EmacsステートではEvilの機能は無効化されて, 従来通りのEmacsとして操作できます. ふたたびEvilに戻るにはもう一度<code>C-z</code>を入力します. しかしながら, Emacsステートを使わなくとも, <code>C-x</code>, <code>M-x</code>, <code>C-g</code>等のキーは, 従来のEmacsと同じように動作し, 挿入ステートでのキーバインドも従来のEmacsのキーバインドに近い(異なる部分をEmacs本来のものに戻す方法は[http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]で解説します)ので, ノーマルステートと挿入ステートの違いをきちんとおさえておけば, よほどのことがない限りEmacsステートのお世話になることはないでしょう. また, 一時的にEmacsのキーバインドを使いたいだけなら, <code>\</code>キー(<code>evil-execute-in-emacs-state</code>コマンド)で, その次のコマンドをEmacsステートで実行し, 直後に元のステートに戻すことができます.

他にどんなステートがあるか, 効率よい編集操作をするにはどうしたらよいかを学ぶには, Vimのチュートリアルを利用するとよいでしょう. <a href="http://vim.googlecode.com/hg/runtime/tutor/tutor"><code>tutor</code></a>(英語)もしくは<a href="http://vim.googlecode.com/hg/runtime/tutor/tutor.ja.utf-8"><code>tutor.ja.utf-8</code></a>(日本語)をダウンロードして, Evilを有効にしたEmacsで開きましょう. ただし, これは本来Vimのためのチュートリアルなので, いくつか注意する点があります.

- 2.7 Evilでは未実装: <code>U</code>((課題として登録されています: https://bitbucket.org/lyro/evil/issue/144/u-does-not-do-anything ))
- 3.2 日本語版ではうまくいかないかも
- 4.1 Evilでは<code>C-g</code>はキャンセルコマンド
-- ファイル情報はモードラインに表示される
- 6.5 Evilでは未実装: <code>:set</code>, <code>\c</code>
-- 検索オプションは別の方法で設定可能([http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]を参照)
- 7.1 EvilではEmacsのヘルプコマンドが呼び出される: <code>:help</code>
- 7.2 Evilでは<code>~/.emacs</code>や<code>~/.emacs.d/init.el</code>を使う
- 7.3 Evilではバージョン1.0-dev以降

><h5 id="usage-vim">Vimからの移民</h5><

Vimを使っていたユーザにとってEvilを使うことは容易いでしょう. ただし, Emacs固有の部分に関しては多少気にする必要があります.

- 機能の名称の違い
-- Vimの「モード」はEvilでは「ステート」
-- yank/paste(Vim)とkill/yank(Emacs)等
- 設定は<code>~/.emacs</code>や<code>~/.emacs.d/init.el</code>に[http://www.gnu.org/software/emacs/manual/elisp.html:title=Emacs Lisp]で書く
-- Vimスクリプトでは書けず, 今後も書けるようになる可能性は極めて低い((Frankがメーリングリストで理由を述べています: https://lists.ourproject.org/pipermail/implementations-list/2011-September/001213.html ))
-- <code>:set</code>でのオプション設定もいまのところできない((有志で実装されるかもしれません: https://lists.ourproject.org/pipermail/implementations-list/2013-February/001776.html ))
- おかしなことになったらとにかく<code>C-g</code>(キャンセル)を連打
- <code>C-c</code>は<code>ESC</code>とは違う意味(同じ意味にする方法は[http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]を参照)
- ヘルプの代わりに<code>M-x describe-<var>*</var></code> (<code><var>*</var></code>は<code>key</code>, <code>bindings</code>, <code>function</code>など)

Evilの最初の目標は, Vimの操作性に関する主要な機能を実装することで, 先日これが達成されバージョン1.0.0がリリースされました. しかし, マイナーな機能やプラグインで提供されるべき機能などは実装されていない場合もあります. マイナーな機能のうち優先的に実装してほしいものがある場合は, [https://bitbucket.org/lyro/evil/issues?status=new&status=open:title=課題トラッカー]に要望を出しましょう.

Evilで実装されていなくても, Vimの一部の機能や, Vimのプラグインの機能の多くには, Emacsの標準/非標準のパッケージで代替できるものがあります. 以下に主なものを挙げておきます. Evil用に開発されたプラグインについては, [http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]で利用方法を解説します.

|*機能/プラグイン|*Emacsのパッケージ|
|<code>C-a</code>, <code>C-x</code>|[https://github.com/cofi/evil-numbers:title=evil-numbers]|
|<code>g;</code>, <code>g,</code>|<a href="http://emacswiki.org/emacs/download/goto-chg.el"><code>goto-chg.el</code></a>|
|<code>&lt;Leader&gt;</code>|[https://github.com/cofi/evil-leader:title=evil-leader]|
|<code>:grep</code>|<code>grep.el</code> (標準) (<code>M-x grep</code>), <a href="http://www.bookshelf.jp/elc/color-moccur.el"><code>color-moccor.el</code></a>|
|<code>:make</code>|<code>compile.el</code> (標準) (<code>M-x compile</code>)|
|タグ検索|<a href="http://emacswiki.org/emacs/EmacsTags"><code>etags.el</code></a> (標準)|
|スペルチェック|<code>ispell.el</code> (標準), <code>flyspell.el</code> (標準)|
|差分モード|[http://www.gnu.org/software/emacs/manual/html_mono/ediff.html:title=Ediff] (標準)|
|タブページ|[http://www.morishima.net/~naoto/elscreen-ja/:title=ElScreen]|
|<code>vundle.vim</code>, <code>neobundle.vim</code>|<code>package.el</code> (標準), [https://github.com/dimitri/el-get:title=El-Get]|
|<code>vimfiler.vim</code>|<code>dired.el</code> (標準), <code>dired-x.el</code> (標準)|
|<code>project.vim</code>|[http://emacswiki.org/emacs/eproject:title=eproject]|
|<code>vcs.vim</code>, <code>fugitive.vim</code>|<code>vc.el</code> (標準), [http://philjackson.github.com/magit/:title=Magit]|
|<code>rails.vim</code>|[https://github.com/antono/evil-rails:title=evil-rails]|
|<code>surround.vim</code>|[https://github.com/timcharper/evil-surround:title=evil-surround]|
|<code>align.vim</code>|<code>align.el</code> (標準)|
|<code>NERD_commenter.vim</code>|[https://github.com/redguardtoo/evil-nerd-commenter:title=evil-nerd-commenter]|
|<code>commentop.vim</code>|[https://github.com/tarao/evil-plugins:title=evil-operator-comment]|
|<code>camelcasemotion.vim</code>|[https://github.com/tarao/evil-plugins:title=evil-little-word]|
|<code>textobj/between.vim</code>|[https://github.com/tarao/evil-plugins:title=evil-textobj-between]|
|<code>neosnippet.vim</code>|[http://capitaomorte.github.com/yasnippet/:title=YASnippet]|
|<code>neocomplcache.vim</code>|[http://cx4a.org/software/auto-complete/index.ja.html:title=Auto Complete Mode]|
|<code>unite.vim</code>, <code>fuf.vim</code>|[http://www.emacswiki.org/Anything:title=Anything], [https://github.com/emacs-helm/helm:title=Helm]|
|<code>vimshell.vim</code>|[http://www.gnu.org/software/emacs/manual/html_node/eshell/index.html:title=Eshell] (標準)|
|<code>quickrun.vim</code>|<a href="https://github.com/syohex/emacs-quickrun"><code>quickrun.el</code></a>|
|<code>syntastic.vim</code>, <code>watchdogs.vim</code>|[http://www.gnu.org/software/emacs/manual/html_node/flymake/index.html:title=Flymake] (標準)|
|<code>xxd</code>, <code>vinarise.vim</code>|<code>hexl.el</code> (標準)|

[f:id:tarao:20130209160344g:image:right]

EvilにあってVimにない機能もあります. たとえば, Evilでは置換の際にもパターンにマッチした部分をハイライトでき, さらに置換結果をバッファ内にプレビューできます.

><h4 id="support">サポート等</h4><

英語のみですが, PDFの公式ドキュメントも提供されています. 他のユーザから寄せられた使い方のヒントなどはEmacsWikiのページにまとめられています. バグの報告や機能の要望は課題トラッカーとメーリングリストの両方で受け付けています(どちらも登録なしに投稿できます). バグを報告する際には開発Wikiに書かれた注意点に目を通すとよいでしょう.

:公式ドキュメント:https://gitorious.org/evil/evil/blobs/raw/doc/doc/evil.pdf
:Wiki:http://emacswiki.org/emacs/Evil
:開発Wiki:https://bitbucket.org/lyro/evil/wiki/Home
:課題トラッカー:https://bitbucket.org/lyro/evil/issues?status=new&status=open
:メーリングリスト:<a href="mailto:implementations-list@lists.ourproject.org">implementations-list@lists.ourproject.org</a>
:メーリングリスト(購読):http://lists.ourproject.org/cgi-bin/mailman/listinfo/implementations-list
:メーリングリスト(ログ):https://lists.ourproject.org/pipermail/implementations-list/

** おわりに

EvilはEmacs上でVimをエミュレートするためのプロジェクトですが, その源流はVimが生まれる前まで遡り, GNU Emacsの歴史と同じくらい古くから受け継がれてきたものです. 長い歴史と現代のモダンな設計によりその完成度は今までにないものとなっており, 多くの人がこのプロジェクトに期待し, また貢献しています.

本稿は, Evilへの貢献の一環として, 導入方法を日本語で提供することを目的として書かれました. これにより, 日本人によるEvilへの貢献が増すことを願っています.

*** 謝辞

本稿にレビューコメントを寄せて下さったid:supermomonga:detailさんとid:hakobe932:detailさんに感謝いたします.

><ol class="local-pager">
   <li class="current">導入編</li>
   <li>[http://d.hatena.ne.jp/tarao/20130304/evil_config:title=設定編]</li>
   <li>[http://d.hatena.ne.jp/tarao/20130305/evil_ext:title=拡張編]</li>
   <li>[http://d.hatena.ne.jp/tarao/20130306/evil_appendix:title=付録]</li>
</ol><
