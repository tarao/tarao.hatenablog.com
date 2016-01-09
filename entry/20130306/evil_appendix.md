---
Title: ' Evil: EmacsをVimのごとく使う - 付録 (カスタム変数一覧)'
Category:
- article
- emacs
- evil
Date: 2013-03-06T00:00:00+09:00
URL: http://tarao.hatenablog.com/entry/20130306/evil_appendix
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/6653586347149235911
---

Evilのカスタム変数の一覧です.

=====
><ul class="table-of-contents top">
  <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
  <li>[http://tarao.hatenablog.com/entry/20130304/evil_config:title=設定編]</li>
  <li>[http://tarao.hatenablog.com/entry/20130305/evil_ext:title=拡張編]</li>
  <li>
    <strong>付録</strong>
[:contents]
  </li>
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

><h4 id="customize">カスタム変数一覧</h4><

><h5 class="custom" id="evil-auto-indent">evil-auto-indent</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:autoindent

自動的にインデントするかどうか. Vimと違いデフォルトで有効です.

><h5 class="custom" id="evil-shift-width">evil-shift-width</h5><

:型:integer
:初期値:4
:バージョン:1.0.0
:Vim:shiftwidth

<code>&lt;</code>オペレータ(<code>evil-shift-left</code>)と<code>&gt;</code>オペレータ(<code>evil-shift-right</code>)で使うオフセット値. Vimと違い初期値は<code>4</code>です.

><h5 class="custom" id="evil-shift-round">evil-shift-round</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:shiftround

<code>&lt;</code>オペレータ(<code>evil-shift-left</code>)と<code>&gt;</code>オペレータ(<code>evil-shift-right</code>)を使用したとき, <code>evil-shift-width</code>の値で丸めるかどうか. <code>t</code>の場合は, シフトした後のインデントの深さが<code>evil-shift-width</code>の倍数になるようにします. Vimと違いデフォルトで有効です.

><h5 class="custom" id="evil-cross-lines">evil-cross-lines</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:whichwrap

カーソル移動時に行をまたぐかどうか. <code>t</code>はVimの<code>whichwrap=b,s,h,l,<,>,[,]</code>に相当(バージョン1.0-devからは<code>whichwrap=b,s,h,l,<,>,[,],~</code>に相当)し, <code>nil</code>はVimの<code>whichwrap=[,]</code>に相当します. Vimの初期値とは異なります.

><h5 class="custom" id="evil-move-cursor-back">evil-move-cursor-back</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

挿入ステートを抜けるときにカーソルを後退するかどうか. また, <code>nil</code>の場合, Vimとは違ってカーソルを改行文字の上に移動することができるようになります.

><h5 class="custom" id="evil-repeat-move-cursor">evil-repeat-move-cursor</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

繰り返しコマンド<code>.</code>(<code>evil-repeat</code>)がカーソルを移動するかどうか. <code>nil</code>に設定すると, 繰り返し操作の後もカーソルが元の場所に留まります.

><h5 class="custom" id="evil-kbd-macro-suppress-motion-error">evil-kbd-macro-suppress-motion-error</h5><

:型:nil | t | record | replay
:初期値:nil
:バージョン:1.0.0
:Vim:-

キーボードマクロの記録(繰り返しコマンド<code>.</code>で使う操作の記録)時に, エラーを無視するかどうか. <code>nil</code>の場合は無視せず, <code>record</code>の場合はキーボードマクロの記録中のみ無視, <code>replay</code>の場合はキーボードマクロの実行中のみ無視, <code>t</code>の場合はどちらの場合も無視します.

><h5 class="custom" id="evil-track-eol">evil-track-eol</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

<code>$</code>コマンド(<code>evil-end-of-line</code>)を実行した後の操作で行末に留まるかどうか. これは[https://bitbucket.org/lyro/evil/issue/120/appending-text-in-block-visual-mode-does:title=矩形選択の挙動をVimに合わせる]ために導入されました.

><h5 class="custom" id="evil-bigword">evil-bigword</h5><

:型:string
:初期値:"^ \t\r\n"
:バージョン:1.0.0
:Vim:-

<code>W</code>コマンド(<code>evil-forward-WORD-begin</code>)が単語とみなす文字集合. 正規表現の<code>[]</code>の中に指定する文字列を指定します.

><h5 class="custom" id="evil-mouse-word">evil-mouse-word</h5><

:型:symbol
:初期値:evil-move-word
:バージョン:1.0.0
:Vim:-

マウスでダブルクリックした際の単語選択に使用する関数.

><h5 class="custom" id="evil-cjk-emacs-word-boundary">evil-cjk-emacs-word-boundary</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

日本語等の単語境界をEmacsのものにするかどうか. <code>nil</code>の場合はVimと互換で, 漢字・ひらがな等の種類が変わるところが単語境界になります. <code>t</code>の場合はEmacsと互換で, 漢字の直後にひらがなが続く場合は, 漢字部分とひらがな部分を同一の単語とみなします(日本語の文節に近いものを単語とみなします).

><h5 class="custom" id="evil-cjk-word-separating-categories">evil-cjk-word-separating-categories</h5><

:型:(list (cons (character . character)))
:初期値:...
:バージョン:1.0.0
:Vim:-

単語の境界を判別するためのリスト. 2つの連続する文字のカテゴリをペアにしたものが, このリストに入っている場合は, その部分は単語境界とみなされます. デフォルトではVimの単語境界を表す値が入っています. 詳細は<code>M-x describe-variable RET word-separating-categories RET</code>を, 文字のカテゴリについて<code>M-x describe-categories RET</code>を参照して下さい.

><h5 class="custom" id="evil-cjk-word-combining-categories">evil-cjk-word-combining-categories</h5><

:型:(list (cons (character . character)))
:初期値:...
:バージョン:1.0.0
:Vim:-

単語の途中を判別するためのリスト. 2つの連続する文字のカテゴリをペアにしたものが, このリストに入っている場合は, その部分は単語の途中とみなされます. デフォルトではVimの単語の途中を表す値が入っています. 詳細は<code>M-x describe-variable RET word-combining-categories RET</code>を, 文字のカテゴリについて<code>M-x describe-categories RET</code>を参照して下さい.

><h5 class="custom" id="evil-want-fine-undo">evil-want-fine-undo</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

<code>cw</code>などの操作を元に戻す単位を細かくするかどうか. <code>nil</code>の場合, 元に戻す単位はVim同様, 挿入ステートに入ってから挿入ステートを抜けるまでの操作になります. <code>t</code>の場合, 元に戻す単位はEmacs本来のものとなり, より細かくなります.

><h5 class="custom" id="evil-want-change-word-to-end">evil-want-change-word-to-end</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

<code>cw</code>が<code>ce</code>のように振る舞うようにするかどうか. <code>nil</code>に設定すると, <code>cw</code>は<code>w</code>で移動する位置までを変更するようになります.

><h5 class="custom" id="evil-want-C-i-jump">evil-want-C-i-jump</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

<code>C-i</code>をジャンプコマンド(<code>evil-jump-forward</code>)に割り当てるかどうか. デフォルトでVimと同様に<code>C-i</code>はジャンプコマンドに割り当てられます. デフォルト設定のままだと, <code>TAB</code>と<code>C-i</code>の区別がないため, ノーマルステートやビジュアルステートでは<code>TAB</code>で自動インデントなどの操作ができません.

><h5 class="custom" id="evil-want-C-u-scroll">evil-want-C-u-scroll</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

<code>C-u</code>を上スクロール(<code>evil-scroll-up</code>)に割り当てるかどうか. Vimと違い, デフォルトではEmacsのプレフィックス引数を無効化しないように, 上スクロールには割り当てません.

><h5 class="custom" id="evil-want-C-w-delete">evil-want-C-w-delete</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

挿入ステートで<code>C-w</code>を単語削除(<code>evil-delete-backward-word</code>に割り当てるかどうか). <code>nil</code>に設定すると挿入ステートでも<code>C-w</code>はウィンドウ移動コマンドになります.

><h5 class="custom" id="evil-want-C-w-in-emacs-state">evil-want-C-w-in-emacs-state</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

Emacsステートでも<code>C-w</code>をウィンドウ移動コマンドに割り当てます.

><h5 class="custom" id="evil-want-visual-char-semi-exclusive">evil-want-visual-char-semi-exclusive</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

限定的に選択範囲からカーソル位置を除くかどうか. <code>nil</code>の場合は, Vimと同様に, 選択範囲にはカーソル位置を含みます. <code>t</code>の場合は, 行頭と行末ではEmacsと同様に選択範囲にカーソル位置を含みません.

><h5 class="custom" id="evil-show-paren-range">evil-show-paren-range</h5><

:型:integer
:初期値:0
:バージョン:1.0.0
:Vim:-

対応する括弧をハイライトする範囲. 括弧からカーソルまでの距離がこの値以下のときに括弧をハイライトします.

><h5 class="custom" id="evil-highlight-closing-paren-at-point-states">evil-highlight-closing-paren-at-point-states</h5><

:型:(list symbol)
:初期値:(not emacs insert replace)
:バージョン:1.0.0
:Vim:-

カーソル下に閉じ括弧があるときに括弧をハイライトするステートのリスト. このリストに入っているステートでは, Vimのノーマルモードのように, カーソル下に閉じ括弧がある場合に括弧をハイライトします. これ以外のステートでは, Emacsのように, カーソルが閉じ括弧の直後にある場合に括弧をハイライトします. リストの先頭が<code>not</code>の場合は条件を反転します.

><h5 class="custom" id="evil-search-module">evil-search-module</h5><

:型:isearch | evil-search
:初期値:isearch
:バージョン:1.0.0
:Vim:-

検索に使うモジュール. <code>isearch</code>はEmacsに組み込みの検索モジュール, <code>evil-search</code>はEvil独自の検索モジュールです. <code>evil-search</code>を選択すると, Vimの検索により近い挙動になります.

><h5 class="custom" id="evil-regexp-search">evil-regexp-search</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

検索に正規表現を使うかどうか. この設定は<code>evil-search-module</code>が<code>isearch</code>の場合のみ有効です.

><h5 class="custom" id="evil-search-wrap">evil-search-wrap</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:wrapscan

検索時にバッファ末尾に到達した場合にバッファ先頭に戻るかどうか. この設定は<code>evil-search-module</code>が<code>isearch</code>の場合のみ有効です.

><h5 class="custom" id="evil-flash-delay">evil-flash-delay</h5><

:型:number
:初期値:2
:バージョン:1.0.0
:Vim:-

検索にマッチした部分のハイライトをやめるまでの秒数. この設定は<code>evil-search-module</code>が<code>isearch</code>の場合のみ有効です.

><h5 class="custom" id="evil-symbol-word-search">evil-symbol-word-search</h5><

:型:nil | t
:初期値:nil
:バージョン:1.0-dev
:Vim:-

<code>*</code>や<code>#</code>で単語単位ではなくシンボル単位で検索するかどうか.

><h5 class="custom" id="evil-magic">evil-magic</h5><

:型:very-magic | t | nil | very-nomagic
:初期値:t
:バージョン:1.0.0
:Vim:magic

検索パターンで使える特殊文字の範囲. この範囲に含まれている文字は, <code>\</code>をつけない限り特殊文字として扱われます. また, このオプションを変更しなくても, Vim同様, 検索パターン中に<code>\v</code>, <code>\m</code>, <code>\M</code>, <code>\V</code>のいずれかのトークンが現れると, それ以降のパターンでは特殊文字の範囲が切り替わります(ただしこれらのトークン自身に関しては変わりません). 設定値と範囲, トークン, Vimのオプションの対応は以下の通りです.
|*設定値|*トークン|*Vimオプション|*範囲|
|<code>very-magic</code>|<code>\v</code>| |<code>0-9A-Za-z_</code>以外|
|<code>t</code>|<code>\m</code>|<code>magic</code>|<code>][{}*+?.&~$^</code>|
|<code>nil</code>|<code>\M</code>|<code>nomagic</code>|<code>][}{*+?$^</code>|
|<code>very-nomagic</code>|<code>\V</code>| |<code>\</code>|
この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-search-case">evil-ex-search-case</h5><

:型:sensitive | insensitive | smart
:初期値:smart
:バージョン:1.0.0
:Vim:ignorecase, smartcase

検索時に大文字・小文字を区別するかどうか. <code>sensitive</code>の場合は区別し, <code>insensitive</code>の場合は区別せず, <code>smart</code>の場合は, 先頭が大文字のときは区別し, 先頭が小文字のときは区別しません. Vimと違い, 初期値は<code>smart</code>です. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-search-vim-style-regexp">evil-ex-search-vim-style-regexp</h5><

:型:boolean
:初期値:nil
:バージョン:1.0.0
:Vim:-

Vim風の正規表現を使うかどうか. <code>nil</code>の場合は従来のEmacsの正規表現を用います. <code>t</code>の場合はVimと互換な正規表現を内部でEmacsの正規表現に変換します. Vimと違い, デフォルトではEmacsの正規表現を用います. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です. <code>evil-search-module</code>が<code>isearch</code>の場合は常にEmacsの正規表現を用います.

><h5 class="custom" id="evil-ex-hl-update-delay">evil-ex-hl-update-delay</h5><

:型:number
:初期値:0.02
:バージョン:1.0.0
:Vim:-

検索結果のハイライトをするまでの待ち時間. この値をキーリピートの間隔よりも短くすることで, (<code>j</code>や<code>C-e</code>による)スクロール中にもハイライトが更新されるようになります. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-search-interactive">evil-ex-search-interactive</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:(incsearch)

検索をインタラクティブにするかどうか. <code>t</code>の場合は検索結果がハイライトされ, 検索パターンの入力中にもハイライトが更新されます. この設定はVimの<code>incsearch</code>に似ていますが, <code>nil</code>の場合もハイライトされないだけで検索そのものはこの設定によらず常にインクリメンタルにする点が異なります. Vimと違い, デフォルトで有効です. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-search-highlight-all">evil-ex-search-highlight-all</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:hlsearch

検索結果をすべてハイライトするかどうか. <code>t</code>の場合, <code>:nohlsearch</code>するか, 別のパターンを検索するまでハイライトされ続けます. Vimと違いデフォルトで有効です. この設定は, <code>evil-search-module</code>が<code>evil-search</code>で, <code>evil-ex-search-interactive</code>が<code>t</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-interactive-search-highlight">evil-ex-interactive-search-highlight</h5><

:型:all-windows | selected-window | nil
:初期値:all-windows
:バージョン:1.0.0
:Vim:-

検索結果をハイライトする範囲. <code>all-windows</code>の場合はすべてのウィンドウでハイライトされます. <code>selected-window</code>の場合は現在のウィンドウでハイライトします. <code>nil</code>の場合はハイライトしません. この設定は<code>evil-search-module</code>が<code>evil-search</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-substitute-case">evil-ex-substitute-case</h5><

:型:nil | sensitive | insensitive | smart
:初期値:nil
:バージョン:1.0.0
:Vim:-

置換時に大文字・小文字を区別するかどうか. <code>sensitive</code>の場合は区別し, <code>insensitive</code>の場合は区別せず, <code>smart</code>の場合は, 先頭が大文字のときは区別し, 先頭が小文字のときは区別しません. <code>nil</code>の場合は, <code>evil-ex-search-case</code>の設定に従います.

><h5 class="custom" id="evil-ex-substitute-global">evil-ex-substitute-global</h5><

:型:boolean
:初期値:nil
:バージョン:1.0-dev
:Vim:gdefault

置換時にデフォルトで複数回マッチするかどうか. <code>t</code>にすると, <code>g</code>フラグをつけなくても複数回マッチするようになり, 逆に<code>g</code>をつけたときは複数回マッチをやめるようになります.

><h5 class="custom" id="evil-ex-substitute-highlight-all">evil-ex-substitute-highlight-all</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

置換対象をハイライトするかどうか. <code>t</code>の場合, 置換(<code>:substitute</code>)の際に置換パターンにマッチする部分をすべてハイライトします.

><h5 class="custom" id="evil-ex-substitute-interactive-replace">evil-ex-substitute-interactive-replace</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

置換結果を表示するかどうか. <code>t</code>の場合, 置換(<code>:substitute</code>)の結果を, 置換対象の部分の直後にインタラクティブに表示します. この設定は<code>evil-ex-substitute-highlight-all</code>が<code>t</code>の場合のみ有効です.

><h5 class="custom" id="evil-ex-visual-char-range">evil-ex-visual-char-range</h5><

:型:boolean
:初期値:nil
:バージョン:1.0-dev
:Vim:-

<code>:</code>コマンドを文字単位の選択範囲に対して適用するかどうか. <code>nil</code>の場合はVimと同様に, <code>'&lt;,'&gt;</code>が<code>:</code>によって挿入されます. <code>t</code>の場合は, Vimにはない<code>`&lt;,`&gt;</code>が挿入されます. Evilでは, <code>:'&lt;,'&gt;</code>は行単位の選択範囲, <code>:`&lt;,`&gt;</code>は文字単位の選択範囲に対してコマンドを適用することを表します.

><h5 class="custom" id="evil-ex-complete-emacs-commands">evil-ex-complete-emacs-commands</h5><

:型:nil | t | in-turn
:初期値:in-turn
:バージョン:1.0-dev
:Vim:-

コマンドライン(<code>:</code>)でEmacsのコマンドを補完の対象にするかどうか. <code>nil</code>の場合はVimのコマンドのみ補完します. <code>t</code>の場合は常にEmacsのコマンドも補完します. <code>in-turn</code>の場合は, Vimのコマンドにマッチするものがない場合のみ, Emacsのコマンドを補完対象にします.

><h5 class="custom" id="evil-complete-all-buffers">evil-complete-all-buffers</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:(complete)

補完の際にすべてのバッファから候補を探すかどうか.

><h5 class="custom" id="evil-complete-next-func">evil-complete-next-func</h5><

:型:function
:初期値:...
:バージョン:1.0.0
:Vim:-

挿入ステートの<code>C-n</code>コマンド(<code>evil-complete-next</code>)で使う補完関数.

><h5 class="custom" id="evil-complete-previous-func">evil-complete-previous-func</h5><

:型:function
:初期値:...
:バージョン:1.0.0
:Vim:-

挿入ステートの<code>C-p</code>コマンド(<code>evil-complete-previous</code>)で使う補完関数.

><h5 class="custom" id="evil-complete-next-minibuffer-func">evil-complete-next-minibuffer-func</h5><

:型:function
:初期値:minibuffer-complete
:バージョン:1.0.0
:Vim:-

ミニバッファの<code>C-n</code>コマンド(<code>evil-complete-next</code>)で使う補完関数.

><h5 class="custom" id="evil-complete-previous-minibuffer-func">evil-complete-previous-minibuffer-func</h5><

:型:function
:初期値:minibuffer-complete
:バージョン:1.0.0
:Vim:-

ミニバッファの<code>C-p</code>コマンド(<code>evil-complete-previous</code>)で使う補完関数.

><h5 class="custom" id="evil-complete-next-line-func">evil-complete-next-line-func</h5><

:型:function
:初期値:...
:バージョン:1.0.0
:Vim:-

ミニバッファの<code>C-x C-n</code>コマンド(<code>evil-complete-next-line</code>)で使う補完関数.

><h5 class="custom" id="evil-complete-previous-line-func">evil-complete-previous-line-func</h5><

:型:function
:初期値:evil-complete-next-line-func
:バージョン:1.0.0
:Vim:-

ミニバッファの<code>C-x C-p</code>コマンド(<code>evil-complete-previous-line</code>)で使う補完関数.

><h5 class="custom" id="evil-lookup-func">evil-lookup-func</h5><

:型:function
:初期値:woman
:バージョン:1.0.0
:Vim:-

キーワード検索コマンド<code>K</code>(<code>evil-lookup</code>)で使う関数.

><h5 class="custom" id="evil-default-cursor">evil-default-cursor</h5><

:型:(set ...)
:初期値:("black" t)
:バージョン:1.0.0
:Vim:-

デフォルトのカーソル. <code>cursor-type</code>に設定するカーソルの形状, カーソル色を表す文字列, あるいはカーソルを変更する0引数関数を要素として含むリストを設定します. デフォルトのカーソル色は<code>(frame-parameter nil 'cursor-color)</code>の値(もし<code>nil</code>のときは<code>"black"</code>)です.

><h5 class="custom" id="evil-mode-line-format">evil-mode-line-format</h5><

:型:before | after | (after . symbol) | (before . symbol)
:初期値:before
:バージョン:1.0.0
:Vim:-

ステートをモードラインのどの位置に表示するか. <code>before</code>はモードリストの前, <code>after</code>はモードリストの後です. <code>cons</code>セルを指定することで, <code>mode-line-format</code>中の特定のシンボルの前か後を指定することもできます.

><h5 class="custom" id="evil-echo-state">evil-echo-state</h5><

:型:boolean
:初期値:t
:バージョン:1.0.0
:Vim:-

エコー領域にステートの変更を通知するかどうか.

><h5 class="custom" id="evil-fold-level">evil-fold-level</h5><

:型:integer
:初期値:0
:バージョン:1.0.0
:Vim:foldlevel

デフォルトの折り畳みレベル.

><h5 class="custom" id="evil-auto-balance-windows">evil-auto-balance-windows</h5><

:型:boolean
:初期値:t
:バージョン:1.0-dev
:Vim:-

Evilのコマンドによってウィンドウが作成/削除されたとき, ウィンドウの大きさを揃え直すかどうか. ただし, Emacsの組込み関数が直接あるいは他のパッケージから間接的に呼び出された場合は, 揃え直しません.

><h5 class="custom" id="evil-esc-delay">evil-esc-delay</h5><

:型:number
:初期値:0.01
:バージョン:1.0.0
:Vim:-

<code>ESC</code>が押された際に後続のキーを待つ秒数. 端末では単体の<code>ESC</code>と<code>M-</code>を区別することができないため, ごく短い間に(事実上同時に)他のキーが押された場合のみ<code>M-</code>として認識する必要があります. 端末そのもの(<code>screen</code>や<code>tmux</code>)でも同様に短い時間待つ設定がある場合は, そちらの調整も必要です.

><h5 class="custom" id="evil-intercept-esc">evil-intercept-esc</h5><

:型:nil | t | always
:初期値:always
:バージョン:1.0-dev
:Vim:-

<code>ESC</code>キーを<code>input-decode-map</code>で特別扱いするかどうか. ターミナル上のEmacs(<code>emacs -nw</code>)では<code>ESC</code>とメタキーが同じイベントになるため, <code>input-decode-map</code>で<code>ESC</code>を特別扱いし, イベントを変換する必要があります. この変数はどういう場合にイベントを変換するかを制御します. <code>nil</code>の場合は一切変換しません. <code>t</code>の場合はターミナル上のEmacsでのみ変換します. <code>always</code>の場合はGUI版のEmacsでも変換します. GUI版でのイベント変換は, <code>C-[</code>を<code>ESC</code>の代わりに使いたい場合に必要です.

><h5 class="custom" id="evil-toggle-key">evil-toggle-key</h5><

:型:string
:初期値:"C-z"
:バージョン:1.0.0
:Vim:-

Emacsステートと行き来するキー.

><h5 class="custom" id="evil-default-state">evil-default-state</h5><

:型:symbol
:初期値:normal
:バージョン:1.0.0
:Vim:-

初期状態のステート.

><h5 class="custom" id="evil-buffer-regexps">evil-buffer-regexps</h5><

:型:(alist string symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

バッファの初期状態のステートを表す連想配列. キーはバッファ名の正規表現で, 値はステートまたは<code>nil</code>です. <code>nil</code>の場合, そのバッファではEvilが完全に無効になります.

><h5 class="custom" id="evil-emacs-state-modes">evil-emacs-state-modes</h5><

:型:(repeat symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

初期状態でEmacsステートになるべきメジャーモードのリスト.

><h5 class="custom" id="evil-insert-state-modes">evil-insert-state-modes</h5><

:型:(repeat symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

初期状態で挿入ステートになるべきメジャーモードのリスト.

><h5 class="custom" id="evil-motion-state-modes">evil-motion-state-modes</h5><

:型:(repeat symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

初期状態で移動ステートになるべきメジャーモードのリスト.

><h5 class="custom" id="evil-overriding-maps">evil-overriding-maps</h5><

:型:(alist symbol symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

Evilのキーバインドを上書きするキーマップの連想配列. キーにはキーマップのシンボルを, 値にはどのステートで上書きするかを指定します. ステートに<code>nil</code>を指定するとすべてのステートで上書きします.

><h5 class="custom" id="evil-intercept-maps">evil-intercept-maps</h5><

:型:(alist symbol symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

Evilのキーバインドを遮るキーマップの連想配列. キーにはキーマップのシンボルを, 値にはどのステートで遮るかを指定します. ステートに<code>nil</code>を指定するとすべてのステートで遮ります. ここで指定したキーマップはEvilのあらゆるキーより優先的に処理され, デバッグ用途に用いられることを想定しています. 通常は<code>evil-overriding-maps</code>を使うべきです.

><h5 class="custom" id="evil-motions">evil-motions</h5><

:型:(repeat symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

移動コマンドとして認識されるべきコマンドのリスト. ここに指定したコマンドは, Evilの移動コマンドと同様に, 移動コマンドと認識されます. 繰り返し操作などで非Evilな移動コマンドがうまく扱われない場合はこのリストに追加するとよいでしょう.

><h5 class="custom" id="evil-visual-newline-commands">evil-visual-newline-commands</h5><

:型:(repeat symbol)
:初期値:...
:バージョン:1.0.0
:Vim:-

選択範囲に対するコマンドのうち, 末尾の改行を含めてはいけないもののリスト. 行選択で末尾に改行が含まれるとうまく動作しない場合はこのリストに追加するとよいでしょう.

><ol class="local-pager">
   <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
   <li>[http://tarao.hatenablog.com/entry/20130304/evil_config:title=設定編]</li>
   <li>[http://tarao.hatenablog.com/entry/20130305/evil_ext:title=拡張編]</li>
   <li class="current">付録</li>
</ol><
