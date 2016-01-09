---
Title: ' Evil: EmacsをVimのごとく使う - 設定編'
Category:
- article
- emacs
- evil
Date: 2013-03-04T00:00:00+09:00
URL: http://tarao.hatenablog.com/entry/20130304/evil_config
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/6653586347149235955
---

><blockquote class="epigraph">
  <p>Then you will see, it is not the spoon that bends, it is only yourself.</p>
  <cite><a href="http://www.imdb.com/title/tt0133093/">The Matrix</a></cite>
</blockquote><

EmacsあるいはVimに慣れ親しんでいれば, Evilを使うのにある程度は勝手がわかるものの, 逆にしっくりこない点も多いでしょう. EvilはEmacsの機能との相互運用性を重視していることから, Vimユーザにとって不慣れな点が生じることは避けられず, EvilがVimをエミュレートする以上, Emacsユーザにとって不慣れな点が生じることも避けられません. 本稿では, どちらに慣れ親しんだユーザにとっても快適に使えるようにEvilをカスタマイズするためのヒントを紹介します. ただし, いくらカスタマイズしても完全なVimや完全なEmacsになることはありません. 使い勝手をよくして自分の好みに合わせながら, できるだけEvilのやり方に慣れていくように努めることが大切でしょう.
=====
><ul class="table-of-contents top">
  <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
  <li>
    <strong>設定編</strong>
[:contents]
  </li>
  <li>[http://tarao.hatenablog.com/entry/20130305/evil_ext:title=拡張編]</li>
  <li>[http://tarao.hatenablog.com/entry/20130306/evil_appendix:title=付録]</li>
</ul><

><style type="text/css">
h4 {
   clear: left;
}
h5.custom {
   font-family: monospace;
   clear: left;
}
h5.custom + dl {
   display: inline-block;
   float: left;
   font-size: 14px;
   border: 4px solid #cccccc;
   border-radius: 5px;
   padding: 0.4em;
   margin: 0 1em 2em 0 !important;
}
h5.custom + dl dt {
   height: 24px;
   width: 6em;
   padding: 0 !important;
   word-break: keep-all;
   float: left;
   clear: left;
}
h5.custom + dl dd {
   min-height: 24px;
   width: 12em;
   margin-left: 0 !important;
   padding: 0 0 0 6em !important;
   border-bottom: 1px solid #cccccc;
   font-family: monospace;
}
h5.custom + dl dd:last-child {
   border-style: none !important;
}
h5.custom + dl + p {
   margin-top: 0 !important;
}
.hatena-image-right {
   clear: both;
}
</style><

><h4 id="keymaps">キーマップ</h4><

Evilの操作性を好みに合うようにするために一番多用するのは, キー設定でしょう. キー設定をするためには, それぞれのステートでどのようなキーマップが使用されるかに加えて, Emacsのキーマップの優先順位について理解する必要があります.

Emacsのキーマップは階層化されていて, あるキーを押したときには, 上の層から順に探索し, 条件に従って使うキーマップを決めます((<a href="http://www.gnu.org/software/emacs/manual/html_node/elisp/Searching-Keymaps.html">http://www.gnu.org/software/emacs/manual/html_node/elisp/Searching-Keymaps.html</a>)).

+ <code>overriding-terminal-local-map</code>が定義されていればこれを使う
+ <code>overriding-local-map</code>が定義されていればこれを使う
+ 以下のいずれかのキーマップにキーが存在していれば, そのキーマップを使う
++ カーソル位置の文字の<code>keymap</code>プロパティ
++ <code>emulation-mode-map-alists</code>中のアクティブなモードのマップ
++ <code>minor-mode-overriding-map-alist</code>中のアクティブなモードのマップ
++ <code>minor-mode-map-alist</code>中のアクティブなモードのマップ
++ カーソル位置の文字の<code>local-map</code>プロパティ
++ <code>(current-local-map)</code>

これらの条件によりキーマップが選択され, 結果としてそのキーマップ中に該当するキーがなければ, <code>(current-global-map)</code>を使います. <code>(current-global-map)</code>にも該当するキーがなければ, 「<code><var>key</var> is undefined</code>」のようなメッセージが表示されます.

ふつうのメジャーモードのキーマップは<code>(current-local-map)</code>, マイナーモードのキーマップは<code>minor-mode-map-alist</code>に設定されています. そして<code>minor-mode-overriding-map-alist</code>は, メジャーモードがマイナーモードのキーマップよりも優先するキーを設定するためにあります.

Evilは内部的にはマイナーモードで実装されていますが, その役割はEmacs全体のキー操作を根本から変えるものであるため, ふつうのメジャーモードやマイナーモードよりも優先してキーを受け取れなければなりません. このため, Evilに関するキーはすべて<code>emulation-mode-map-alists</code>に設定され((<code>emulation-mode-map-alists</code>はViperの頃から使われているキーマップ層で, まさにこのような用途のためにあります)), ふつうのメジャーモードやマイナーモードよりもEvilのキーの方が基本的には優先されます. 逆に, Evilのキーが他のキーバインドに奪われてしまう場合は, <code>emulation-mode-map-alists</code>よりも上位の層にそのキーがないか確認するとよいでしょう. たとえば, カーソル位置の文字の<code>keymap</code>プロパティが設定されている場合はこれが優先されることを知らないと, このキーマップの存在に思い至ることすら困難です.

><h5 id="keymap-state-key">特定のステートでのキー</h5><

あるステートで使うキーを設定するには, 設定ファイル(<code>~/.emacs</code>や<code>~/.emacs.d/init.el</code>等)の<code>(require 'evil)</code>の後に, <code>(define-key evil-<var>state</var>-state-map <var>key</var> <var>definition</var></code>)</code>の行を追加します. たとえば, モーションステートで<code>;</code>を<code>:</code>(<code>evil-ex</code>)に割り当てるには,
>|lisp|
(define-key evil-motion-state-map (kbd ";") #'evil-ex)
||<
とします.

<code><var>state</var></code>は<code>normal</code>, <code>insert</code>, <code>visual</code>, <code>operator</code>, <code>replace</code>, <code>motion</code>のいずれかです. <code>normal</code>はノーマルステート, <code>insert</code>は挿入ステート, <code>visual</code>はビジュアルステート(範囲選択中のステート), <code>operator</code>はオペレータ待機ステート(オペレータを入力してから後続のキーを入力するまでのステート), <code>replace</code>は置換ステート(<code>R</code>したときのステート), <code>motion</code>は編集を伴わない状態を表すためのステートです. <code>motion</code>ステートで定義されたキーは<code>normal</code>や<code>visual</code>でも使えます. <code>normal</code>で定義されたキーは<code>visual</code>でも使えます.

<code><var>definition</var></code>には上の例のようなコマンドのシンボルの他に, 文字列(キーボードマクロとして動作), キーマップ(プレフィックスキーの定義になる)なども渡すことができます. 詳しくは<code>M-x describe-function RET define-key RET</code>を参照して下さい.

既に定義されているキーを削除するには, <code>define-key</code>に<code><var>definition</var></code>として<code>nil</code>を渡します. たとえば, 挿入ステートで<code>C-y</code>をふつうのEmacsのコマンド(<code>yank</code>)にするには, <code>evil-insert-state-map</code>から<code>C-y</code>を削除します.
>|lisp|
(define-key evil-insert-state-map (kbd "C-y") nil)
||<

初期状態でどのステートにどんなキーが定義されているかは<a href="https://bitbucket.org/lyro/evil/src/master/evil-maps.el"><code>evil-maps.el</code></a>を見るとわかります. また, <code>evil-<var>state</var>-state-map</code>の代わりに<code>evil-<var>state</var>-state-local-map</code>を使うと, バッファローカルなキーバインドを定義できます.

初期状態では, 挿入ステートでもいくつかのキーにはVimの挿入モードと同様のコマンドが割り当てられています. もし挿入ステートでのキーバインドを完全にEmacs互換にしたい場合は, 次のようにします.
>|lisp|
(setq evil-insert-state-map nil)
||<

><h5 id="keymap-mode-state-key">特定のモードの特定のステートでのキー</h5><

特定のモードが有効な場合には通常, <code><var>mode-name</var>-map</code>のようなキーマップで定義されたキーが有効になりますが, これらは<code>emulation-mode-map-alists</code>で定義されているEvilのキーバインドより優先順位が低く, またEvilのステートごとに違ったキーバインドにすることができません.

<code>evil-define-key</code>コマンドを使うと, 特定のモードが有効な場合のキーを, 優先順位を気にすることなく特定のステートでだけ定義できます. <code>(evil-define-key <var>state</var> <var>mode-map</var> <var>key</var> <var>definition</var>)</code>の形で用いて, <code><var>state</var></code>にEvilのステート, <code><var>mode-map</var></code>に特定のモードで有効なキーマップを指定する以外は, <code>define-key</code>と同様です. たとえば, <code>view-mode</code>のモーションステートでの<code>v</code>に, <code>view-mode</code>を終了するコマンドを割り当てるには, 次のようにします.
>|lisp|
(evil-define-key 'motion view-mode-map (kbd "v")
  #'(lambda () (interactive) (view-mode 0)))
||<
<code><var>state</var></code>に<code>nil</code>を指定すると, すべてのステートで有効なキーを定義できます.

><h5 id="keymap-misc">その他のEvilのキーマップ</h5><

Evilにはステートごとのキーマップ以外に以下のようなキーマップがあります.
:<code>evil-window-map</code>:ウィンドウ移動キー(<code>C-w</code>)を押した後のキーを定義するキーマップ
:<code>evil-outer-text-objects-map</code>:<code><var>OP</var> a</code>に続く, テキストオブジェクトを選択するキー(例: <code>daw</code>の<code>w</code>)を定義するキーマップ
:<code>evil-inner-text-objects-map</code>:<code><var>OP</var> i</code>に続く, テキストオブジェクトを選択するキー(例: <code>vib</code>の<code>b</code>)を定義するキーマップ
:<code>evil-ex-search-keymap</code>:検索(<code>/</code>や<code>?</code>等)ミニバッファでのキーマップ(検索モジュールが<code>evil-search</code>の場合のみ有効)
:<code>evil-ex-completion-map</code>:コマンドライン(<code>:</code>)でのキーマップ
:<code>evil-read-key-map</code>:後続の文字を利用するオペレータ(<code>&quot;</code>や<code>f</code>)で, 文字を読み取るときのキーマップ

これらのキーマップを変更する場合は<code>define-key</code>を使います.

><h5 id="keymap-override">ふつうのEmacsのキーマップを優先</h5><

前述の通り, Evilのキーは他のメジャーモード・マイナーモードのキーよりも優先されます. もし特定のモードで, Evilのキーよりも優先してそのモードのすべてのキーを使いたい場合は, <code>evil-make-overriding-map</code>を使って, そのモードのキーマップの優先順位を上げる必要があります. <code>(evil-make-overriding-map <var>map</var> <var>state</var>)</code>の形で用いて, <code><var>map</var></code>には優先したいキーマップ, <code><var>state</var></code>にはどのステートのときに優先するかを指定します. <code><var>state</var></code>を省略すると全ステートで優先されます. たとえば, ノーマルステートで<code>howm-menu-mode-map</code>の全てのキーを優先するには次のようにします.
>|lisp|
(evil-make-overriding-map howm-menu-mode-map 'normal)
||<

むやみに優先順位を上げると, Evilの移動キー(<code>hjkl</code>)などが覆い隠されて, 優先されたモードのコマンドが呼び出されるようになってしまうことがあります. これを避け, 移動キーはEvilのコマンドのままにするには, <code>evil-add-hjkl-bindings</code>を使います. <code>(evil-add-hjkl-bindings <var>map</var> <var>state</var>)</code>の形で用い, 引数の意味は<code>evil-make-overriding-map</code>のものと同じです. たとえば, ノーマルステートで<code>mew-message-mode-map</code>の<code>hjkl</code>以外の全てのキーを優先するには以下のようにします.
>|lisp|
(evil-make-overriding-map mew-message-mode-map 'normal)
(evil-add-hjkl-bindings mew-message-mode-map 'normal)
||<

移動キーを設定するついでに, その他のキーを設定することもできます. 次の例では, ノーマルステートで<code>mew-summary-mode-map</code>の<code>hjkl</code>以外の全てのキーを優先した上で, <code>hl</code>は<code>mew-summary-mode-map</code>のキー, <code>GJK;</code>は<code>evil-motion-satate-map</code>のキーを使うようにしています.
>|lisp|
(evil-make-overriding-map mew-summary-mode-map 'normal)
(evil-add-hjkl-bindings mew-summary-mode-map 'normal
  "h" (lookup-key mew-summary-mode-map "h")
  "l" (lookup-key mew-summary-mode-map "l")
  "G" (lookup-key evil-motion-state-map "G")
  "J" (lookup-key evil-motion-state-map "J")
  "K" (lookup-key evil-motion-state-map "K")
  ";" (lookup-key evil-motion-state-map ";"))
||<

キーに関すること以外のEmacsの機能との相互運用については<a href="#emacs">後の節</a>で触れます.

><h4 id="customize">カスタマイズ</h4><

続いて, Emacs標準のカスタマイズ機構を用いて設定できるオプションについて解説します.

Evilのカスタマイズオプションは, Evilの挙動をVimに近づけるか, あるいはEmacsに近づけるかを制御するためのオプションが中心ですが, Vimのオプションに相当するものもあります. Vimに慣れ親しんだユーザのために, 相当するVimのオプションも明記しました. Vimの挙動にできる限り近づけたい場合は, <a href="#vim">後の節</a>も参照して下さい.

オプションを実際に設定する場合, 2通りの方法があります. Emacs標準のカスタマイズ機構を専用のUIで操作する方法と, 設定ファイルにLisp式を書く方法です. 前者の場合, <code>M-x customize-group RET evil RET</code> (<code>M-x</code>は<code>Alt</code>と<code>x</code>の同時押し)とすると, 設定項目一覧が表示され, マウスクリックで設定を変更できます(設定を変更した場合は保存するのを忘れないようにしましょう). 後者の場合は, <code>~/.emacs</code>や<code>~/.emacs.d/init.el</code>で,
>|lisp|
(setq evil-cross-lines t
      evil-search-module 'evil-search
      evil-ex-search-vim-style-regexp t)
(require 'evil)
(evil-mode 1)
||<
のように, Evilを<code>require</code>するよりも前に, <code>setq</code>で設定します. もしもEl-Getを使っている場合は,
>|lisp|
(setq evil-cross-lines t
      evil-search-module 'evil-search
      evil-ex-search-vim-style-regexp t)
(el-get-bundle evil)
(require 'evil)
(evil-mode 1)
||<
のように, <code>(el-get-bundle evil)</code>よりも前に設定しなければならないことに注意しましょう.

<code>setq</code>にはオプションの名前と設定値を書いていきます. 以下の説明では, 設定値としてのシンボルはそのままシンボル名を書いていますが, Lispの式として表現する際には<code>'</code>をつける必要があります(ただし<code>'(...)</code>の中ではつけません).

すべてのカスタマイズオプションの一覧は[http://d.hatena.ne.jp/tarao/20130306/evil_appendix:title=付録]に記載してあります. ここでは重要なもののみを紹介します.

><h5 class="custom" id="evil-cross-lines">evil-cross-lines</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:whichwrap

カーソル移動時に行をまたぐかどうか. <code>t</code>はVimの<code>whichwrap=b,s,h,l,<,>,[,]</code>に相当し, <code>nil</code>はVimの<code>whichwrap=[,]</code>に相当します. Vimの初期値とは異なります.

><h5 class="custom" id="evil-move-cursor-back">evil-move-cursor-back</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

挿入ステートを抜けるときにカーソルを後退するかどうか. また, <code>nil</code>の場合, Vimとは違ってカーソルを改行文字の上に移動することができるようになります.

><h5 class="custom" id="evil-want-C-i-jump">evil-want-C-i-jump</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

<code>C-i</code>をジャンプコマンド(<code>evil-jump-forward</code>)に割り当てるかどうか. デフォルトでVimと同様に<code>C-i</code>はジャンプコマンドに割り当てられます. デフォルト設定のままだと, <code>TAB</code>と<code>C-i</code>の区別がないため, ノーマルステートやビジュアルステートでは<code>TAB</code>で自動インデントなどの操作ができません.

><h5 class="custom" id="evil-search-module">evil-search-module</h5><

:型:isearch | evil-search
:初期値:isearch
:バージョン:1.0.0
:Vim:-

検索に使うモジュール. <code>isearch</code>はEmacsに組み込みの検索モジュール, <code>evil-search</code>はEvil独自の検索モジュールです. <code>evil-search</code>を選択すると, Vimの検索により近い挙動になります.

><h5 class="custom" id="evil-ex-search-vim-style-regexp">evil-ex-search-vim-style-regexp</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

Vim風の正規表現を使うかどうか. <code>nil</code>の場合は従来のEmacsの正規表現を用います. <code>t</code>の場合はVimと互換な正規表現を内部でEmacsの正規表現に変換します. Vimと違い, デフォルトではEmacsの正規表現を用います. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です. <code>evil-search-module</code>が<code>isearch</code>の場合は常にEmacsの正規表現を用います.

><h5 class="custom" id="evil-esc-delay">evil-esc-delay</h5><

:型:number
:初期値:0.01
:バージョン:1.0.0
:Vim:-

<code>ESC</code>が押された際に後続のキーを待つ秒数. 端末では単体の<code>ESC</code>と<code>M-</code>を区別することができないため, ごく短い間に(事実上同時に)他のキーが押された場合のみ<code>M-</code>として認識する必要があります. 端末そのもの(<code>screen</code>や<code>tmux</code>)でも同様に短い時間待つ設定がある場合は, そちらの調整も必要です.

><h4 id="emacs">Emacsの機能との共存</h4><

Emacsの機能のキーマップとEvilのキーマップの衝突を防ぐ方法は<a href="#keymaps">前の節</a>で説明した通りです. この節ではそれ以外にEmacsの機能をEvilから扱いやすくする方法を解説します. EvilでEmacs本来の機能をうまく扱うための設定の一部は, Evil本体に<a href="https://bitbucket.org/lyro/evil/src/master/evil-integration.el"><code>evil-integration.el</code></a>として組み込まれていて, ここで解説する内容の具体例になっています.

ここで解説する設定は<code>(require 'evil)</code>の行よりも後に書きます.

><h5 id="emacs-states">モードごとのデフォルトのステート</h5><

特定のモードでの初期ステートを指定するには, <code>(evil-set-initial-state <var>mode</var> <var>state</var>)</code>を使います. バッファ内容の編集を伴わないようなモードをモーションステートにしたいといったような場合に使います. たとえば, <code>view-mode</code>がモーションステートになるようにするには,
>|lisp|
(evil-set-initial-state 'view-mode 'motion)
||<
とします(実際にはこの設定は不要で, <code>view-mode</code>は元からモーションステートになるように設定されています). モードには本来メジャーモードを指定すべきですが, 上記の例のようにマイナーモードを指定することもできます.

><h5 id="emacs-evilize">EmacsのコマンドをEvil化</h5><

ほとんどのEmacsのコマンドは, Evilからもふつうに使えますが, Evil独自の機能(繰り返しやビジュアルステート)との親和性を高めるための調整方法も用意されています. Evilに慣れてきて, Evilの機能とEmacsのコマンドがうまく組み合わさっていないことに気づいたら, 以下の宣言が必要かどうか検討してみましょう.

:<code>(evil-declare-not-repeat <var>command</var>)</code>:<code><var>command</var></code>を繰り返し操作の対象外にします.
:<code>(evil-declare-abort-repeat <var>command</var>)</code>:<code><var>command</var></code>を実行したら繰り返す操作の記録を中止します.
:<code>(evil-declare-change-repeat <var>command</var>)</code>:繰り返し操作のために<code><var>command</var></code>によるバッファの変更点を記録します. 通常はキー入力しか記録せず, 多くの場合はそれで問題ありません. 自動補完メニューが表示されるのを待ってから選択する場合など, キー入力の再現がバッファの変更内容を再現しないようなコマンドに対してはこれを設定するとよいでしょう.
:<code>(evil-declare-motion <var>command</var>)</code>:<code><var>command</var></code>を移動コマンドとして宣言します. 移動コマンドはビジュアルステートを終了しません.

EmacsのリージョンコマンドをEvilのオペレータにするといったような, 高度なEvil化の方法については[http://d.hatena.ne.jp/tarao/20130305/evil_ext:title=拡張編]を参照して下さい.

><h5 id="emacs-case-skk">事例紹介: SKKとの共存</h5><

Emacsのパッケージの機能とEvilの機能はしばしば衝突します. キーマップの定義やコマンドのEvil化で済む場合は簡単ですが, それでは済まない場合も多々あります. 開発者にメーリングリストで対策を仰ぐのも一つの方法ですが, ここでは自力でうまく動くようにする一例を紹介します.

日本語入力に[http://openlab.ring.gr.jp/skk/index-j.html:title=SKK]を使っていると, いくつかの機能がEvilと衝突します.

SKKには入力モードに応じてカーソルの色を自動的に変更する機能がありますが, Evilもステートに応じてカーソル形状を変更するため, 調整が必要です. 挿入ステートの場合だけ, カーソルの制御をSKKに任せるのがよいでしょう. <code>defadvice</code>で関数そのものを置き換えて, <code>ad-do-it</code>で元の関数を呼び出すというイディオムを使うと, 以下のように書けます.
>|lisp|
(defadvice update-buffer-local-cursor-color
  (around evil-update-buffer-local-cursor-color-in-insert-state activate)
  ;; SKKによるカーソル色変更を, 挿入ステートかつ日本語モードの場合に限定
  "Allow ccc to update cursor color only when we are in insert
state and in `skk-j-mode'."
  (when (and (eq evil-state 'insert) (bound-and-true-p skk-j-mode))
    ad-do-it))
(defadvice evil-refresh-cursor
  (around evil-refresh-cursor-unless-skk-mode activate)
  ;; Evilによるカーソルの変更を, 挿入ステートかつ日本語モードではない場合に限定
  "Allow ccc to update cursor color only when we are in insert
state and in `skk-j-mode'."
  (unless (and (eq evil-state 'insert) (bound-and-true-p skk-j-mode))
    ad-do-it))
||<

Evilの検索(<code>evil-search</code>モジュール)は, 入力するごとに表示をアップデートしていくので, 日本語の変換途中にもアップデートが起きてしまい入力が困難です. SKKの未確定状態ではアップデートを抑制するとよいでしょう. 上の例と同じように, <code>defadvice</code>を使って以下のように書けます.
>|lisp|
(defadvice evil-ex-search-update-pattern
  (around evil-inhibit-ex-search-update-pattern-in-skk-henkan activate)
  ;; SKKの未確定状態(skk-henkan-mode)ではない場合だけ, 検索パターンをアップデート
  "Inhibit search pattern update during `skk-henkan-mode'.
This is reasonable since inserted text during `skk-henkan-mode'
is a kind of temporary one which is not confirmed yet."
  (unless (bound-and-true-p skk-henkan-mode)
    ad-do-it))
||<

><h4 id="vim">Vimの再現性を高める</h4><

Emacs本来の機能を損なわないために, デフォルト設定ではEmacs寄りの設定とVim寄りの設定の中間になっています. このため, Vimと全く同じ挙動を期待した場合にはしばしばうまくいかないでしょう. この節では, Vimに慣れ親しんだユーザのために, できる限りVimに近い挙動にするためにどんな設定をすればいいか解説します.

多くはVimの機能や設定に対応するEmacsのやり方の紹介ですが, 中にはEmacsの機能を犠牲にしてVimの挙動を愚直に再現しようとするものもあります. そのようにVimのやり方に固執すると, Emacsの機能も最大限に利用するという目標から外れるおそれがあることには留意しておきましょう.

><h5 id="vim-customize">カスタマイズ</h5><

カスタマイズオプションで設定できる範囲で, できる限りVimに近づけるには以下のようにします(これは<code>(require 'evil)</code>よりも前に書きます).
>|lisp|
(setq evil-want-C-u-scroll t
      evil-search-module 'evil-search
      evil-ex-search-vim-style-regexp t)
||<
ただし, これはVimのデフォルト設定と対応しているわけではなく, Vimで以下のように設定した状態に近いものとなります.
>|vim|
set autoindent
set smartcase
set hlsearch
set incsearch
set shiftwidth=4
set shiftround
set whichwrap=[,]
||<

><h5 id="vim-show-paren">括弧の対応をハイライト</h5><

デフォルトでは対応する括弧のハイライトは有効になっていないので, 設定ファイルで次のようにします.
>|lisp|
(show-paren-mode t)
||<

><h5 id="vim-eof">バッファの終端を明示</h5><

Vimのように<code>~</code>でバッファの終わりを明示する方法はありませんが, 他のやり方で同等のことが実現できます.

[f:id:tarao:20130225203855p:image:right]
GUI版のEmacsを使っている場合は, 以下のようにするとバッファの終わり以降が左側の縁のところに明示されます.
>|lisp|
(setq indicate-empty-lines t)
||<

><div class="hatena-image-right">
  [f:id:tarao:20130225203857p:image]
  [f:id:tarao:20130225203858p:image]
</div><
GUI版ではもう少し凝った方法でバッファの範囲を明示することもできます. バッファがまだ上下に続く場合は矢印が, 上限または下限の場合は鉤印が表示されます. 以下のように設定します.
>|lisp|
(setq indicate-buffer-boundaries 'left)
||<

[f:id:tarao:20130225203856p:image:right]
また, 拙作の<a href="https://raw.github.com/tarao/elisp/master/end-mark.el"><code>end-mark.el</code></a>を使うと, GUI版・コンソール版(<code>emacs -nw</code>)によらず, バッファの終端に<code>[EOF]</code>と表示できます. インストールするには, 上記ファイルを<code>~/.emacs.d/</code>等に保存し, 以下の設定を書きます.
>|lisp|
(add-to-list 'load-path user-emacs-directory)
(require 'end-mark)
(global-end-mark-mode)
||<

><h5 id="vim-final-newline">ファイル末尾で必ず改行</h5><

設定ファイルに以下のように書きます.
>|lisp|
(setq require-final-newline t)
||<

><h5 id="vim-word">単語境界をVim互換に</h5><

単語の境界をどう判別するかは, 基本的にはEmacsの仕組みをそのまま用いているため, Vimとはわずかに異なります. 具体的には, Vimと違い「<code>_</code>」を単語の境界とみなします. Vimと同じように「<code>_</code>」を単語の一部とみなすようにするには, 設定ファイルに以下のように書きます.
>|lisp|
(modify-syntax-entry ?_ "w" (standard-syntax-table))
||<

><h5 id="vim-c-c"><code>C-c</code>を<code>ESC</code>に, <code>C-c</code>/<code>ESC</code>でキャンセル</h5><

Vimのように<code>C-c</code>を<code>ESC</code>のような役割にするには, 以下のようにします.

>|lisp|
(defun evil-escape-or-quit (&optional prompt)
  (interactive)
  (cond
   ((or (evil-normal-state-p) (evil-insert-state-p) (evil-visual-state-p)
        (evil-replace-state-p) (evil-visual-state-p)) [escape])
   (t (kbd "C-g"))))
(define-key key-translation-map (kbd "C-c") #'evil-escape-or-quit)
(define-key evil-operator-state-map (kbd "C-c") #'evil-escape-or-quit)
(define-key evil-normal-state-map [escape] #'keyboard-quit)
||<

この設定では, Vimの挙動により近づけるために, <code>C-c</code>と<code>ESC</code>ともにキャンセルの意味も持たせています.

><h5 id="vim-number"><code>C-a</code>/<code>C-x</code>でインクリメント/デクリメント</h5><

Evilそのものには<code>C-a</code>/<code>C-x</code>によるインクリメント/デクリメントは含まれていませんが, [https://github.com/cofi/evil-numbers:title=evil-numbers]で提供されています.

[http://melpa.milkbox.net/:title=MELPA]を使っている場合は<code>M-x package-install RET evil-numbers RET</code>でインストールされます. それ以外の場合は, <a href="https://raw.github.com/cofi/evil-numbers/master/evil-numbers.el"><code>evil-numbers.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存しましょう.

初期設定ではどのキーにも割り当てられないので, 以下のように設定します.
>|lisp|
(define-key evil-normal-state-map (kbd "C-a") #'evil-numbers/inc-at-pt)
(define-key evil-normal-state-map (kbd "C-x") #'evil-numbers/dec-at-pt)
||<
ただし, この設定はEmacsの<code>C-x</code>を無効にするため, 多くのEmacsの機能が利用できなくなります. 提供元の設定例では, 以下のように<code>C-c +</code>/<code>C-c -</code>に割り当てる設定が推奨されています.
>|lisp|
(define-key evil-normal-state-map (kbd "C-c +") #'evil-numbers/inc-at-pt)
(define-key evil-normal-state-map (kbd "C-c -") #'evil-numbers/dec-at-pt)
||<

><h5 id="vim-mapleader"><code>mapleader</code></h5><

EvilそのものにはVimの<a href="http://vim-jp.org/vimdoc-ja/map.html#mapleader"><code>mapleader</code></a>に相当する機能は実装されていませんが, [https://github.com/cofi/evil-leader:title=evil-leader]で提供されています.

[http://melpa.milkbox.net/:title=MELPA]を使っている場合は<code>M-x package-install RET evil-leader RET</code>でインストールされます. それ以外の場合は, <a href="https://raw.github.com/cofi/evil-leader/master/evil-leader.el"><code>evil-leader.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存しましょう.

たとえば, Vimの<code>let leader=","</code>, <code>map <Leader>e :edit </code>, <code>map <Leader>b :buffer </code>に相当する設定をするには,
>|lisp|
(require 'evil-leader)
(evil-leader/set-leader ",")
(evil-leader/set-key
 "e" #'find-file
 "b" #'switch-to-buffer)
||<
とします.

><h5 id="vim-commandline-registers">コマンドラインでレジスタから貼り付け</h5><

いまのところ, Evil本体のコマンドラインでは<code>C-r</code>がレジスタからの貼り付けに割り当てられていない(バージョン1.0.0)か, 割り当てられてはいるもののVimのコマンドラインで使えるカーソル下のオブジェクトの貼り付け(<code>C-r C-w</code>等)は実装されていません(バージョン1.0-dev). <a href="https://raw.github.com/tarao/evil-plugins/master/evil-ex-registers.el"><code>evil-ex-registers.el</code></a>を用いると, これらが使えるようになります.

インストールするには, <a href="https://raw.github.com/tarao/evil-plugins/master/evil-ex-registers.el"><code>evil-ex-registers.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'evil-ex-registers)
(define-key evil-ex-search-keymap (kbd "C-r") #'evil-ex-paste-from-register)
(define-key evil-ex-completion-map (kbd "C-r") #'evil-ex-paste-from-register)
||<

><h5 id="vim-tabpage">タブページ</h5><

[http://vim-jp.org/vimdoc-ja/tabpage.html:title=Vimのタブページ]に相当するものはEmacsの標準パッケージには含まれていませんが, [http://www.morishima.net/~naoto/elscreen-ja/:title=ElScreen]を使うとほぼ同等のことができます.

ElScreenのインストールは[http://melpa.milkbox.net/:title=MELPA]を使うのが簡単でしょう. まずは設定ファイルに以下のように書きます(既に同じことが書いてある場合は必要ありません).
>|lisp|
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "http://melpa.milkbox.net/packages/") t)
(package-initialize)
||<
一度Emacsを起動して<code>M-x package-install RET elscreen RET</code>とするとインストールされます.

そのままのElScreenはEmacs用のインタフェースしかないので, <code>:tabnew</code>などで操作したい場合は設定が必要です. 以下の設定を設定ファイルに書くと, Vimのタブページに関する操作が概ね有効になります. ただし, タブ番号が0で始まる等, 些細な差異はあります.

><script src="https://gist.github.com/5019545.js"></script><

><h5 id="vim-set-listchars"><code>set listchars</code></h5><

Vimでは空白文字を表示するのに<code>listchars</code>オプションを使いますが, Emacsでは[http://www.gnu.org/software/emacs/manual/html_node/elisp/Character-Display.html:title=ディスプレイテーブル]を使います. この仕組みは, 原理上あらゆる文字の表示方法を制御できます.

空白文字に限れば, 簡単な設定をするだけで, 裏側でディスプレイテーブル(やフォントロック)を使った制御をしてくれるパッケージ<code>whitespace.el</code>がEmacs 23から標準搭載されているので, これを使うのがよいでしょう.

以下が, Vimの設定に対応するディスプレイテーブルもしくは<code>whitespace.el</code>の設定です.
>|lisp|
(require 'whitespace)
(setq whitespace-style '(face tabs tab-mark space space-mark newline))

;; set lcs=eol:$
(setcar (nthcdr 2 (assq 'newline-mark whitespace-display-mappings)) [?$ ?\n])

;; set lcs=tab:^\ ,
(setcar (nthcdr 2 (assq 'tab-mark whitespace-display-mappings)) [?^ ?\t])

;; set lcs=tab:^-,
(setcar (nthcdr 2 (assq 'tab-mark whitespace-display-mappings)) [?^ ?- ?- ?- ?- ?- ?- ?-])

;; set lcs=extends:<,precedes:<
(set-display-table-slot standard-display-table 'truncation ?<)

;; set nbsp:%
(setcar (nthcdr 2 (assq 'space-mark whitespace-display-mappings)) [?%])

(global-whitespace-mode)
||<

<code>whitespace.el</code>が表示を制御するのは<code>whitespace-style</code>に入っている要素のみです. 要素の意味と, 他にどんなものの表示を制御できるかは, <code>M-x describe-variable RET whitespace-style RET</code>を参照して下さい.

<code>set-display-table-slot</code>を使う場合, 文字のフェイス(フォントや色など)も同時に変更することができます. この場合は<code>(set-display-table-slot standard-display-table <var>slot</var> (make-glyph-code <var>c</var> <var>face</var>))</code>のようにします. <code><var>face</var></code>にはフェイスを表すシンボルを指定します. 既に定義されているフェイスは<code>M-x list-face-display RET</code>で一覧できます. 自分で新しいフェイスを作るには<code>defface</code>マクロを使います.

このような設定を用いても, 以下の点はVimと異なります.
- <code>set lcs=tab:<var>xy</var></code>の<code><var>y</var></code>を設定する場合はタブ幅が固定
- <code>set lcs=trail:<var>c</var></code>は設定できない
-- 色だけ変えることは可能(ただし, <code>whitespace-style</code>から<code>space-mark</code>を外しておく)<br><code>(setq show-trailing-whitespace t)</code><br><code>(set-face-background 'trailing-whitespace "#cc9900")</code>
- <code>set lcs=extends:<var>c</var></code>と<code>set lcs=precedes:<var>c</var></code>を個別には設定できない
- <code>set lcs=conceal:<var>c</var></code>はconcealに完全に対応する概念がないため設定できない
-- ただし, [http://www.gnu.org/software/emacs/manual/html_node/elisp/Text-Properties.html:title=テキストプロパティ]を使って[http://www.gnu.org/software/emacs/manual/html_node/elisp/Replacing-Specs.html:title=同様のこと]は可能
- Emacsでは折り返し記号も変更可能<br><code>(set-display-table-slot standard-display-table 'wrap <var>c</var>)</code>
- Emacsでは非表示の行を表す記号も変更可能<br><code>(set-display-table-slot standard-display-table 'selective-display <var>c</var>)</code>
- Emacsでは<code>standard-display-table</code>を直接書き換えればあらゆる文字の表示のしかたを変更可能

><h5 id="vim-set-tabstop"><code>set tabstop</code></h5><

<code>:set tabstop=<var>num</var></code>相当のことをするには<code>:(setq tab-width <var>num</var>) RET</code>とします. デフォルトのタブ幅を指定するには, 設定ファイルに以下のように書きます.
>|lisp|
(setq tab-width 4)
||<

><h5 id="vim-set-expandtab"><code>set expandtab</code></h5><

<code>:set expandtab RET</code>相当のことをするには<code>:(setq indent-tabs-mode nil) RET</code>とします. デフォルト設定を変えるには, 設定ファイルに以下のように書きます.
>|lisp|
(setq indent-tabs-mode nil)
||<

既に入力されているタブを<code>tab-width</code>の幅のスペースに展開するには, 展開する範囲を選択して<code>M-x untabify RET</code>とします.

><h5 id="vim-set-statusline"><code>set statusline</code></h5><

VimのステータスラインはEmacsでは[http://www.gnu.org/software/emacs/manual/html_node/elisp/Mode-Line-Format.html:title=モードライン]と呼びます. モードラインは<code>mode-line-format</code>という変数に[http://www.gnu.org/software/emacs/manual/html_node/elisp/Mode-Line-Data.html:title=決まったデータ構造]の値を設定することで変更できます. このデータ構造はシンボルや文字列のリストが入れ子になったもので, 文字列には[http://www.gnu.org/software/emacs/manual/html_node/elisp/Text-Properties.html:title=プロパティ]も設定できるため, 見せ方を自由に変更できます. 文字列中の<code>%<var>c</var></code>は特別な意味を持ち表示する際に展開されますが, Vimとは意味が異なるため, Emacsの[http://www.gnu.org/software/emacs/manual/html_node/elisp/_0025_002dConstructs.html:title=マニュアル]を参照して下さい.

><h5 id="vim-set-number"><code>set number</code></h5><

<code>:set number RET</code>相当のことをするには<code>M-x linum-mode RET</code>とします. もう一度同じことをすると<code>:set nonumber RET</code>になります. 行番号を常に表示にするには設定ファイルに以下のように書きます.
>|lisp|
(global-linum-mode)
||<

><h5 id="vim-color">色テーマ</h5><

Vimでは<code>colorscheme</code>で色設定をしますが, Emacsでは2つのやり方があります. 一つは[http://www.nongnu.org/color-theme/:title=color-theme]を使う方法で, もう一つはEmacs 24から標準搭載された[http://www.gnu.org/software/emacs/manual/html_node/emacs/Custom-Themes.html:title=カスタムテーマ]を使う方法です. color-themeは近頃はメンテナンスされていないので, 古いEmacsを使う予定がなければ使わない方がよいでしょう.

color-themeは, [http://melpa.milkbox.net/:title=MELPA]を使っている場合は<code>M-x package-install RET color-theme RET</code>でインストールできます. テーマそのものは初期状態でもいくつか定義されていて, 追加でインストールすることもできます. 特定のテーマを適用にするには設定ファイルに以下のように書きます.
>|lisp|
(color-theme-initialize)
(color-theme-dark-laptop) ; テーマdark-laptopを適用
||<

カスタムテーマを使うには, <code>load-theme</code>コマンドを使います. デフォルトで提供されているテーマか, 自分で<code>~/.emacs.d/</code>((正確には<code>custom-theme-directory</code>の値が表すディレクトリ))に保存した<code><var>name</var>-theme.el</code>ファイルを読み込むことができます. 設定ファイルに書く場合は以下のようにします.
>|lisp|
(load-theme 'zenburn t) ; テーマzenburnを適用
||<

Vimで人気の配色はEmacsにも移植されています. 以下にcolor-theme用, カスタムテーマ用の対応するEmacsパッケージを挙げておきます.

|*colorscheme|*color-theme|*カスタムテーマ|
|desert|<a href="https://github.com/superbobry/color-theme-desert"><code>color-theme-desert.el</code></a>|<a href="https://github.com/emacs-jp/replace-colorthemes"><code>desert-theme.el</code></a>|
|molokai|<a href="https://github.com/alloy-d/color-theme-molokai"><code>color-theme-molokai.el</code></a>| |
|solarized|<a href="https://github.com/sellout/emacs-color-theme-solarized"><code>color-theme-solarized.el</code></a>|<a href="https://github.com/sellout/emacs-color-theme-solarized"><code>solarized-dark-theme.el</code></a>, <a href="https://github.com/sellout/emacs-color-theme-solarized"><code>solarized-light-theme.el</code></a>|
|zenburn|<a href="https://raw.github.com/bbatsov/zenburn-emacs/5f4b790731c17d717dfc818a8f30b99b56c89f68/color-theme-zenburn.el"><code>color-theme-zenburn.el</code></a>|<a href="https://github.com/bbatsov/zenburn-emacs"><code>zenburn-theme.el</code></a>((あちこちに様々な実装がありますが, これが完成度が高くよくメンテナンスされています))|

><h5 id="vim-modeline">モードライン</h5><

EmacsにもVimのモードライン相当のものがありますが, 記法が異なります. Vimのモードラインの記法(の一部)をEmacsでも利用できるようにするには[https://github.com/cinsk/emacs-vim-modeline:title=emacs-vim-modeline]を使います.

><h4 id="misc">その他のべんり設定</h4><

><h5 id="misc-physical-line">物理行移動と論理行移動を入れ替え</h5><

Vimと同じく, デフォルトでは<code>j</code>, <code>k</code>は論理行移動(改行文字単位の行移動)で, <code>gj</code>, <code>gk</code>が物理行移動(見たままの行移動)になっています. 通常は物理行で移動した方がわかりやすいので, Vimユーザでこれを入れ替えている人も多いでしょう. Evilでは以下のようにすると入れ替えることができます(この設定は<code>(require 'evil)</code>の行よりも後に書きます).
>|lisp|
(defun evil-swap-key (map key1 key2)
  ;; MAP中のKEY1とKEY2を入れ替え
  "Swap KEY1 and KEY2 in MAP."
  (let ((def1 (lookup-key map key1))
        (def2 (lookup-key map key2)))
    (define-key map key1 def2)
    (define-key map key2 def1)))
(evil-swap-key evil-motion-state-map "j" "gj")
(evil-swap-key evil-motion-state-map "k" "gk")
||<

><h5 id="misc-yank-pop"><code>C-n</code>/<code>C-p</code>をハイブリッドに</h5><

デフォルトでは, Evilのノーマルステートの<code>C-n</code>と<code>C-p</code>は<a href="http://www.vim.org/scripts/script.php?script_id=1234"><code>YankRing.vim</code></a>のようにヤンク履歴の操作に割り当てられています((ヤンク履歴そのものはEmacsでは標準機能(kill ring)です)). <code>p</code>などで貼り付けた直後に, <code>C-p</code>で一つ前にヤンク(コピー)した内容を貼り付け直し, <code>C-n</code>で一つ後のものを貼り付け直すという機能です.

これはこれで大変べんりな機能ですが, 本来の行移動コマンドをつぶす割に, 直前のコマンドが貼り付けではない場合にはエラーメッセージが出るだけで, 勿体ない感じがします. 直前が貼り付けコマンドでなければふつうに上下移動してくれてもよいでしょう.

そういうわけで, <code>C-n</code>, <code>C-p</code>の直前が貼り付けコマンドでないときに上下移動になるようにするには以下のようにします.

>|lisp|
(defadvice evil-paste-pop (around evil-paste-or-move-line activate)
  ;; evil-paste-popできなかったらprevious-lineする
  "If there is no just-yanked stretch of killed text, just move
to previous line."
  (condition-case err
      ad-do-it
    (error (if (eq this-command 'evil-paste-pop)
               (call-interactively 'previous-line)
             (signal (car err) (cdr err))))))
(defadvice evil-paste-pop-next (around evil-paste-or-move-line activate)
  ;; evil-paste-pop-nextできなかったらnext-lineする
  "If there is no just-yanked stretch of killed text, just move
to next line."
  (condition-case err
      ad-do-it
    (error (if (eq this-command 'evil-paste-pop-next)
               (call-interactively 'next-line)
             (signal (car err) (cdr err))))))
||<

><h4 id="plugins">プラグイン</h4><

Evilのために書かれたプラグインを紹介します. ここで紹介されている以外にも, Vimのプラグインと同等の機能を持ったEmacsパッケージは多数存在します(対応表は[http://d.hatena.ne.jp/tarao/20130303/evil_intro#usage-vim:title=導入編]を参照)が, それらの導入方法は個別のパッケージのドキュメントを参照して下さい. ここではEvilに特化したプラグインのみを挙げます. これらはEvil公式のものではなく, ユーザたちが作成して公開しているものです.

><h5 id="plugin-evil-mode-line">evil-mode-line</h5><

><div class="hatena-image-right">
  [f:id:tarao:20130225203854p:image]<br>
  [f:id:tarao:20130225203853p:image]<br>
  [f:id:tarao:20130225203852p:image]
</div><

EmacsのモードラインにEvilのステートを表示し, ステートに応じてモードラインの背景色を切り替える機能を提供します.

インストールするには, <a href="https://raw.github.com/tarao/evil-plugins/master/evil-mode-line.el"><code>evil-mode-line.el</code></a>と<a href="https://raw.github.com/tarao/elisp/master/mode-line-color.el"><code>mode-line-color.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'evil-mode-line)
||<

カスタマイズ方法などは[https://github.com/tarao/evil-plugins:title=tarao/evil-plugins]のページを参照して下さい.

><h5 id="plugin-evil-surround">evil-surround</h5><

<a href="http://www.vim.org/scripts/script.php?script_id=1697"><code>surround.vim</code></a>の移植版です. 選択範囲に対して<code>S<var>type</var></code>とすると, <code><var>type</var></code>で指定された種類の要素を追加します. たとえば, <code>S&lt;tag&gt;</code>とすると, 選択範囲が<code>&lt;tag&gt;</code>タグで囲まれます.

インストールするには, <a href="https://raw.github.com/timcharper/evil-surround/master/evil-surround.el"><code>evil-surround.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'evil-surround)
(global-evil-surround-mode 1)
||<

カスタマイズ方法などは[https://github.com/timcharper/evil-surround:title=evil-surround]のページを参照して下さい.

><h5 id="plugin-evil-nerd-commenter">evil-nerd-commenter</h5><

<a href="http://www.vim.org/scripts/script.php?script_id=1218"><code>NERD_commenter.vim</code></a>の移植版です. 行ごとのコメントアウト/アンコメントのためのプラグインです.

インストールするには, <a href="https://raw.github.com/redguardtoo/evil-nerd-commenter/master/evil-nerd-commenter.el"><code>evil-nerd-commenter.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと, <code>M-;</code>でコメントアウトできるようになります.
>|lisp|
(require 'evil-nerd-commenter)
(evilnc-default-hotkeys)
||<

カスタマイズ方法などは[https://github.com/redguardtoo/evil-nerd-commenter:title=evil-nerd-commenter]のページを参照して下さい.

これをインストールしなくても, Emacsでは元々<code>M-;</code>が<code>comment-dwim</code>に割り当てられていて, 選択範囲のコメントアウト/アンコメントができます. evil-nerd-commenterは, 範囲を選択しなかったときに現在の行をコメントアウトする, 行数を指定できる, という点が異なります.

><h5 id="plugin-evil-operator-comment">evil-operator-comment</h5><

<a href="http://www.vim.org/scripts/script.php?script_id=2708"><code>commentop.vim</code></a>の移植版です. コメントアウト/アンコメントのためのオペレータを定義します. オペレータなので, 同じキー2回で行に対して適用, 行数指定での適用, オブジェクト単位での適用, 文字・行・矩形選択範囲に対しての適用などが可能です.

インストールするには<a href="https://raw.github.com/tarao/evil-plugins/master/evil-operator-comment.el"><code>evil-operator-comment.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと, <code>C</code>でコメントアウトできるようになります.
>|lisp|
(require 'evil-operator-comment)
(global-evil-operator-comment-mode 1)
||<

カスタマイズ方法などは[https://github.com/tarao/evil-plugins:title=tarao/evil-plugins]のページを参照して下さい.

これをインストールしなくても, Emacsでは元々<code>M-;</code>が<code>comment-dwim</code>に割り当てられていて, 選択範囲のコメントアウト/アンコメントができ, 選択範囲に対してはEvilのオペレータとして用いることができます. ただし, 矩形選択時にも正しく動作させるためにはevil-operator-commentが必要です.

><h5 id="plugin-evil-operator-moccur">evil-operator-moccur</h5><

選択範囲などに対して複数ファイル/ディレクトリにまたがる検索(<code>moccur-grep-find</code>)をするためのオペレータです.

インストールするには, <a href="http://www.emacswiki.org/emacs/download/color-moccur.el"><code>color-moccur.el</code></a>と<a href="https://raw.github.com/tarao/evil-plugins/master/evil-operator-moccur.el"><code>evil-operator-moccur.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと, <code>M</code>で検索できるようになります.
>|lisp|
(require 'evil-operator-moccur)
(global-evil-operator-moccur-mode 1)
||<

カスタマイズ方法などは[https://github.com/tarao/evil-plugins:title=tarao/evil-plugins]のページを参照して下さい.

><h5 id="plugin-evil-relative-linum">evil-relative-linum</h5><

[f:id:tarao:20130225223144p:image:right]

オペレータを入力した直後から適用範囲が確定するまでの間, 相対行番号を表示します.

インストールするには, <a href="http://github.com/tarao/elisp/raw/master/linum+.el"><code>linum+.el</code></a>と<a href="https://raw.github.com/tarao/evil-plugins/master/evil-relative-linum.el"><code>evil-relative-linum.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'evil-relative-linum)
||<

Vimの<code>relativenumber</code>オプションと違い, オペレータが入力されたときだけ相対行番号を表示します.

><h5 id="plugin-evil-little-word">evil-little-word</h5><

<a href="http://www.vim.org/scripts/script.php?script_id=1905"><code>camelcasemotion.vim</code></a>の移植版です. 通常の単語単位よりも小さい部分を単語として扱う移動コマンド, オブジェクトを提供します. たとえば, "CamelCase"を"Camel"と"Case"の2語として, "snake_case"を"snake"と"case"の2語として扱うような単語単位です.

インストールするには, <a href="https://raw.github.com/tarao/evil-plugins/master/evil-little-word.el"><code>evil-little-word.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと, <code>glw</code>/<code>glb</code>による移動, <code>ilw</code>/<code>alw</code>のオブジェクトが定義されます.
>|lisp|
(require 'evil-little-word)
||<

オリジナルの<code>camelcasemotion.vim</code>と違い, 英語のアルファベット以外の文字の大文字・小文字も正しく扱われます. たとえば, "Tietokoneen&Auml;&auml;ni"という文字列は, <code>camelcasemotion.vim</code>では1語になりますが, evil-little-wordでは"Tietokoneen"と"&Auml;&auml;ni"の2語になります.

><h5 id="plugin-evil-textobj-between">evil-textobj-between</h5><

<a href="https://github.com/thinca/vim-textobj-plugins/blob/between/plugin/textobj/between.vim"><code>textobj/between.vim</code></a>の移植版です. <code>if<var>c</var></code>や<code>af<var>c</var></code>で, 文字<code><var>c</var></code>に囲まれた部分を選択できるようになります.

インストールするには, <a href="https://raw.github.com/tarao/evil-plugins/master/evil-textobj-between.el"><code>evil-textobj-between.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'evil-textobj-between)
||<

><h5 id="plugin-evil-paredit">evil-paredit</h5><

[http://www.emacswiki.org/emacs/ParEdit:title=ParEdit]をEvilから使えるようにするためのプラグインです. 括弧の対応を保った編集が強制されるようになります.

インストールするには[http://melpa.milkbox.net/:title=MELPA]を使って<code>M-x package-install RET evil-paredit RET</code>とします. 有効にするには<code>M-x evil-paredit-mode RET</code>とします. たとえば, Emacs Lispファイルを編集する際に有効にするには設定ファイルに以下のように書きます.
>|lisp|
(require 'evil-paredit)
(add-hook 'emacs-lisp-mode-hook 'evil-paredit-mode)
||<

><h5 id="plugin-evil-rails">evil-rails</h5><

<a href="https://github.com/tpope/vim-rails"><code>rails.vim</code></a>の移植版です. Railsプロジェクトを管理するためのインタフェースを提供します.

インストールするには, <code>M-x package-install RET rinari RET</code>で[http://rinari.rubyforge.org/:title=Rinari]をインストールし, <a href="https://raw.github.com/antono/evil-rails/master/evil-rails.el"><code>evil-rails.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'rinari)
(require 'evil-rails)
||<

><h5 id="plugin-hexl-evil-patch">hexl-evil-patch</h5><

Emacsのバイナリエディタモード(<code>hexl-mode</code>)をEvilから問題なく使えるようにするためのプラグインです.

インストールするには<a href="https://raw.github.com/tarao/evil-plugins/master/hexl-evil-patch.el"><code>hexl-evil-patch.el</code></a>をダウンロードして, <code>load-path</code>内のディレクトリに保存します. 設定ファイルに以下のように書くと有効になります.
>|lisp|
(require 'hexl-evil-patch)
||<

><h4 id="user-config">参考になりそうな個人設定</h4><

本来は, Evilを使う上でのおすすめ設定をすぐに使える形で紹介したいところですが, それにはVimやEmacsの使い方にまで踏み込む必要があり, Evilの解説という範疇から外れてしまうため, また, VimやEmacsでのベストプラクティスそのものが多様であり意見の別れるところなので, 本稿ではそのような設定の紹介はあえてしないことにしました. むしろ, 本稿の読者の皆様が設定自慢の記事を書いて盛り上げてくれることを願っています.

ここでは代わりに, 既にEvilを使っているユーザの個人設定のうち, Web上で公開されているものをピックアップしました. これらを参考に是非とも自分だけの究極の設定を見出して下さい.

以下, ユーザIDの敬称は省略します.

*** [https://github.com/tarao/dotfiles/blob/master/.emacs.d/init/evil.el:title=tarao]

本稿の筆者の個人設定です. [https://github.com/tarao/dotfiles/tree/master/.emacs.d/init:title=同じディレクトリ]に他のEmacs設定のファイルがあります. この設定のリポジトリそのものを<code>git clone</code>して, <a href="https://github.com/tarao/dotfiles/blob/master/.emacs.d/init.el"><code>dotfiles/.emacs.d/init.el</code></a>に対して以下のようにEmacsを起動すると, 必要なパッケージが(リポジトリディレクトリ内に)自動的にダウンロードされて, 既存のEmacsの設定を汚すことなく設定を試すことができます(ただし, ダウンロードには少し不安になるくらい時間がかかります).
>||
emacs -q -l init.el
||<

*** [https://github.com/cofi/dotfiles/blob/master/emacs.d/config/cofi-evil.el:title=cofi]

いくつかのプラグインの作者でもある方の個人設定です. 挿入ステートで<code>jk</code>と素早く入力すると挿入ステートを抜けられるようになっているのが特徴的です.

*** [https://github.com/fukamachi/emacs-config/blob/master/inits/10-evil.el:title=fukamachi]

ふだんLispを書いている方の設定なので, [https://github.com/fukamachi/emacs-config/tree/master/inits:title=同じディレクトリ]のEmacs設定も含めて参考になります.

*** [https://github.com/mikio/dotfiles/blob/master/emacs/mikio/mikio-evil.el:title=mikio]

いろいろなパッケージのコマンドが細かにEvil化されていて参考になります.

*** [https://github.com/patbl/colemak-evil:title=patbl]

[http://colemak.com/:title=Colemak配列]のための設定です.

*** [https://github.com/yukihr/dotfiles/blob/master/.emacs.d/inits/12_evil.el:title=yukihr]

おそらくDvorak配列でEvilを使っている方だと思います. [https://github.com/yukihr/dotfiles/tree/master/.emacs.d/inits:title=同じディレクトリ]の他のEmacs設定も参考になります.

** おわりに

本稿では, Evilのカスタマイズ方法, とくに, よりEmacs的な設定にする方法, あるいは, よりVim的な設定にする方法を紹介しました. 元々EmacsやVimを使っていたユーザがEvilに乗り換えるには, どんなにカスタマイズをほどこしたところで, なお不慣れな点が残ると思います. Evilを使うということは, Emacsの良さとVimの良さを両方とも取り入れる代償に, 今まで慣れ親しんだ環境は諦めて, Evilのやり方を受け入れていく覚悟が必要なのかもしれません. 新たな環境への移行が少しでもスムーズにいくように, 本稿を役立ててもらえれば幸いです.

*** 謝辞

本稿にレビューコメントを寄せて下さったid:hakobe932:detailさんに感謝いたします.

><ol class="local-pager">
   <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
   <li class="current">設定編</li>
   <li>[http://tarao.hatenablog.com/entry/20130305/evil_ext:title=拡張編]</li>
   <li>[http://tarao.hatenablog.com/entry/20130306/evil_appendix:title=付録]</li>
</ol><
