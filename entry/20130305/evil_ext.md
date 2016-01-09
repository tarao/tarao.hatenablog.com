---
Title: ' Evil: EmacsをVimのごとく使う - 拡張編'
Category:
- article
- emacs
- evil
Date: 2013-03-05T00:00:00+09:00
URL: http://tarao.hatenablog.com/entry/20130305/evil_ext
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/6653586347149235920
---

><blockquote class="epigraph">
  <p>Roads? Where we're going we don't need roads.</p>
  <cite><a href="http://www.imdb.com/title/tt0088763/">Back to the Future</a></cite>
</blockquote><

Evilの真髄はその拡張性です. 本稿では主にチュートリアルを通して, Evilを拡張する方法を解説します.
=====
><ul class="table-of-contents top">
  <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
  <li>[http://tarao.hatenablog.com/entry/20130304/evil_config:title=設定編]</li>
  <li>
    <strong>拡張編</strong>
[:contents]
  </li>
  <li>[http://tarao.hatenablog.com/entry/20130306/evil_appendix:title=付録]</li>
</ul><

><style type="text/css">
.selection {
   color: white;
   background-color: #8c8ce8;
}
</style><

><h4 id="overview">Evilの拡張</h4><

Evilはもともと拡張性を考慮して設計されています. 最小限のコア部分がEvilの本体だとすると, ステート, コマンド, オペレータ, モーション, テキストオブジェクトなどEvilの具体的な機能はすべて拡張機能として実装されているとも言えます.

これら拡張機能を定義するための機構はあらかじめコアに備わっていて, <code>evil-define-<em>functionality</em></code>といった名前のマクロが用意されています. たとえば, <code>evil-define-state</code>, <code>evil-define-command</code>, <code>evil-define-operator</code>, <code>evil-define-motion</code>, <code>evil-define-text-object</code>といった具合です.

あとは, 必要に応じたEmacs Lispの知識と, Evilの内部的な動作のための補助関数についての知識さえあれば, 簡単にEvilのプラグインを作成できます. このようにして作られた拡張機能の例としてもっとも優れているのは, Evilそのものの機能のソースコードです.  最初のうちは, [https://bitbucket.org/lyro/evil/src/master/evil-commands.el:title=evil-commands.el]などを参考にするのがよいでしょう.

このあとの節では, Evilの拡張機能を定義する方法をチュートリアル形式で解説していきます.

><h4 id="tutorial1">例1: コメントアウトオペレータ</h4><

オペレータを自作する例として, コメントアウト/アンコメントするためのオレータを定義してみましょう. オペレータは範囲選択なしに用いられた場合は, 直後にモーションを伴って, 最初のカーソル位置からモーション終了後の位置までの範囲に, 何らかの操作を施すものです. 選択範囲が与えられた場合は, 文字・行・矩形の選択種別に応じて操作を施します.

Evilでは, <code>(evil-define-operator <em>name</em> (beg end ...) ...)</code>の形でオペレータを定義できます. <code>beg</code>, <code>end</code>には選択範囲の開始位置と終了位置が渡されます. 後続のモーションの扱いはとくに明示的にやる必要はなく, モーション終了後に決定された選択範囲が渡されます.

><h5 id="tutorial1-simple">単純な選択範囲のコメントアウト</h5><

Emacsでは, <code>comment-or-uncomment-region</code>という関数で, 特定の範囲をコメントアウト/アンコメントできます. 選択種別について気にしなければ, これを<code>evil-define-operator</code>の中で呼び出すだけで, オペレータの定義が完成します.
>|lisp|
(evil-define-operator evil-comment-or-uncomment-region (beg end)
  "Comment out text from BEG to END."
  (interactive "<r>")
  (comment-or-uncomment-region beg end))
||<

名前は<code>evil-comment-or-uncomment-region</code>としました. <code>interactive</code>に渡している<code>&lt;r&gt;</code>は, このオペレータの引数として選択範囲を受け取ることを表していて, 範囲が選択されていないときには<code>interactive</code>の行を解釈する途中でモーションの入力を自動的に待ちます.

><h5 id="tutorial1-blockwise">矩形選択範囲のコメントアウト</h5><

上の定義でも, 文字選択(<code>v</code>)や行選択(<code>V</code>)の場合はうまくいきますが, 矩形選択(<code>C-v</code>)された範囲をうまくコメントアウトすることができません. 矩形選択の場合は, 選択範囲に含まれる部分を行ごとに分割して操作を適用する必要があります. たとえば, 以下の文章の<code>|</code>の間が選択範囲だとします.
>|
Assume that the range
between |<span class="selection"> in thi</span>s text
is selece<span class="selection">d. If t</span>he
selection<span class="selection"> is blo</span>ckwise,
then each<span class="selection"> line o</span>f the
selected <span class="selection">text is</span>
restricte<span class="selection">d to th</span>e
columns b<span class="selection">etween </span>|.
|<
矩形選択では<span class="selection">色つき</span>で示した部分が選択範囲ですが, 実際には<code>beg</code>と<code>end</code>には<code>|</code>の位置が渡ってくるだけなので, 矩形選択の場合は行ごとに列の範囲を限定しながら操作を適用する必要があります. 矩形選択範囲を行ごとの範囲に変換して操作を適用できるように, <code>evil-apply-on-block</code>という内部関数が用意されています.

選択種別を受け取れるように<code>interactive</code>に<code>&lt;R&gt;</code>を渡すようにして, 矩形選択のときには<code>evil-apply-on-block</code>を使うようにすると以下のようになります.
>|lisp|
(evil-define-operator evil-comment-or-uncomment-region (beg end type)
  "Comment out text from BEG to END with TYPE."
  (interactive "<R>")
  (if (eq type 'block)
      (evil-apply-on-block #'comment-or-uncomment-region beg end nil)
    (comment-or-uncomment-region beg end)))
||<

実はこのままではうまくいきません. <code>comment-or-uncomment-region</code>は, コメント開始記号を挿入/削除するので, 行ごとの操作を適用する途中で, 列の範囲がずれてしまうのです. Emacsには, 文字列の挿入/削除が起きても位置を正しく保存するために, マーカという仕組みがあります. まずは行ごとに限定すべき範囲に対してマーカを設定して, そのマーカで示された範囲にコメントアウト操作を適用するようにしましょう.
>|lisp|
(defun evil-mark-on-lines (beg end lines)
  (let ((beg-marker (save-excursion (goto-char beg) (point-marker)))
        (end-marker (save-excursion (goto-char end) (point-marker))))
    (set-marker-insertion-type end-marker t)
    (setcdr lines (cons (cons beg-marker end-marker) (cdr lines)))))

(defun evil-apply-on-block-markers (func beg end &rest args)
  "Like `evil-apply-on-block' but first mark all lines and then
call the function on the marked ranges."
  (let ((lines (list nil)))
    (evil-apply-on-block #'evil-mark-on-lines beg end nil lines)
    (dolist (range (nreverse (cdr lines)))
      (let ((beg (car range)) (end (cdr range)))
        (apply func beg end args)
        (set-marker beg nil)
        (set-marker end nil)))))

(evil-define-operator evil-comment-or-uncomment-region (beg end type)
  "Comment out text from BEG to END with TYPE."
  (interactive "<R>")
  (if (eq type 'block)
      (evil-apply-on-block-markers #'comment-or-uncomment-region beg end)
    (comment-or-uncomment-region beg end)))
||<

><h5 id="tutorial1-keymap">キーマップ</h5><

プラグインが定義するキーは, 適宜オン/オフできるとよいでしょう. そのためには, プラグイン専用のマイナーモードを定義し, そのマイナーモードが有効な場合だけキーを割り当てるのがよいでしょう.

マイナーモードは以下のように定義します. これにより, マイナーモード用のキーマップ<code>evil-operator-comment-mode-map</code>が自動的に作られます.
>|lisp|
(defgroup evil-operator-comment nil
  "Comment/uncomment operator for Evil"
  :prefix "evil-operator-comment-"
  :group 'evil)
(define-minor-mode evil-operator-comment-mode
  "Buffer local minor mode of comment/uncomment operator for Evil."
  :lighter ""
  :keymap (make-sparse-keymap)
  :group 'evil-operator-comment
  (evil-normalize-keymaps))
||<

<code>evil-define-key</code>を使うと, このマイナーモードが有効で, かつ特定のステートのときだけ使えるキーを定義できます. ここでは, ノーマルステートとビジュアルステートの<code>C</code>キーに割り当てることにします.
>|lisp|
(evil-define-key 'normal evil-operator-comment-mode-map
                 "C" 'evil-comment-or-uncomment-region)
(evil-define-key 'visual evil-operator-comment-mode-map
                 "C" 'evil-comment-or-uncomment-region)
||<

このマイナーモードを常に有効にしておきたい場合のために, グローバルマイナーモードも定義しておくとよいでしょう.
>|lisp|
(defun evil-operator-comment-mode-install () (evil-operator-comment-mode 1))
(define-globalized-minor-mode global-evil-operator-comment-mode
  evil-operator-comment-mode evil-operator-comment-mode-install
  "Global minor mode of comment/uncomment operator for Evil.")
||<

これで, 使う側では
>|lisp|
(global-evil-operator-comment-mode 1)
||<
とするだけで済むようになります.

><h4 id="tutorial2">例2: 次のシンボルへ移動するモーション</h4><

モーションを自作する例として, 次のシンボルへ移動するモーションを定義してみましょう.

><h5 id="tutorial2-motion">モーションの定義</h5><

Emacsにはもともと<code>forward-symbol</code>(thingatpointパッケージ)というコマンドがあり, Evilを使っていてもこのコマンドはふつうに動作します. しかし, <code>w</code>(<code>evil-forward-word-begin</code>)で単語移動するときなどは単語の先頭で止まりますが, <code>forward-symbol</code>はシンボルの終端(シンボルの最後の文字の次の位置)で止まってしまいます. これはちょうど, Emacsの単語移動コマンド<code>forward-word</code>が単語の終端で止まるのと同じで, EmacsとVimの移動コマンドの流儀の違いに由来します.

この流儀の違いを吸収するために, Evilには<code>evil-move-beginning</code>という内部関数が用意されています. この関数には, 移動する回数と, Emacs流の移動関数を渡します. ただし, 移動関数は指定された回数の移動ができなかった場合(途中でバッファの先頭や末尾に到達してしまった場合)は, 残りの移動回数を返す必要があります. 残り移動回数を返すようにしたシンボル移動のコマンドを仮に<code>evil-forward-symbol</code>とすると, <code>w</code>のようにVimの流儀でシンボル移動するコマンド<code>evil-forward-symbol-begin</code>は以下のように定義できます.
>|lisp|
(evil-define-motion evil-forward-symbol-begin (count)
  "Move the cursor to the beginning of the COUNT-th next symbol."
  :type exclusive
  (evil-move-beginning count #'evil-forward-symbol))
||<
モーションの定義には<code>evil-define-motion</code>マクロを使います. <code>:type</code>には, オペレータを施したときに, 移動後のカーソル位置をオペレータの適用範囲に含めるかどうかを指定します. <code>evil-forward-symbol</code>は次のシンボルの先頭に移動するコマンドなので, 移動後のカーソル位置をオペレータの適用範囲には含めません(<code>exclusive</code>を指定します).

次に<code>evil-forward-symbol</code>を定義しましょう. <code>evil-forward-word</code>の定義を参考に, <code>foward-symbol</code>がバッファの端点に到達してしまったときだけ<code>nil</code>を返すと仮定して, 以下のように定義してみました.
>|lisp|
(defun evil-forward-symbol (&optional count)
  (setq count (or count 1))
  (let* ((dir (if (>= count 0) +1 -1))
         (count (abs count)))
    (while (and (> count 0)
                (forward-symbol dir))
      (setq count (1- count)))
    count))
||<
実はこれはうまくいきません. なぜなら, <code>forward-symbol</code>は<code>forward-word</code>と違って, 端点に到達しても<code>nil</code>を返さない場合があるからです. そこで, <code>forward-symbol</code>の定義を少し修正した<code>evil-forward-smbol1</code>を定義して, そちらを使うようにしましょう.
>|lisp|
(defun evil-forward-symbol1 (dir)
  (if (natnump dir)
      (re-search-forward  "\\(\\sw\\|\\s_\\)+" nil 'move count)
    (prog1 (re-search-backward "\\(\\sw\\|\\s_\\)+" nil 'move)
      (skip-syntax-backward "w_"))))

(defun evil-forward-symbol (&optional count)
  (setq count (or count 1))
  (let* ((dir (if (>= count 0) +1 -1))
         (count (abs count)))
    (while (and (> count 0)
                (evil-forward-symbol1 dir))
      (setq count (1- count)))
    count))
||<

これでおおよそうまくいきますが, このままでは<code>w</code>と違って, 空行を飛ばしてしまいます. Evilには<code>evil-move-empty-lines</code>という, 次の空行まで移動するコマンドがあるので, このコマンドと, いま定義した<code>evil-forward-symbol</code>と, 移動距離の短い方を使えばうまくいきます. Evilには「移動距離の短い方を使う」ためのマクロ<code>evil-define-union-move</code>も用意されているので, これを使いましょう.
>|lisp|
(evil-define-union-move evil-move-symbol (count)
  "Move by symbols."
  (evil-forward-symbol count)
  (evil-move-empty-lines count))
||<

いま定義した<code>evil-move-symbol</code>を, <code>evil-forward-symbol</code>の代わりに<code>evil-move-beginning</code>に渡すように, <code>evil-forward-symbol-begin</code>を修正します.
>|lisp|
(evil-define-motion evil-forward-symbol-begin (count)
  "Move the cursor to the beginning of the COUNT-th next symbol."
  :type exclusive
  (evil-move-beginning count #'evil-move-symbol))
||<

同じようにして, 前のシンボルに移動するコマンドも定義できます.
>|lisp|
(evil-define-motion evil-backward-symbol-begin (count)
  "Move the cursor to the beginning of the COUNT-th previous symbol."
  :type exclusive
  (evil-move-beginning (- (or count 1)) #'evil-move-symbol))
||<
内部で用いるコマンドは<code>count</code>が負数でもいいように定義してあったので, <code>evil-move-beginning</code>に渡す<code>count</code>の符号を反転させるだけで, 何も新しいものは要りません.

さらに, <code>evil-move-beginning</code>の代わりに<code>evil-move-end</code>を使うと, 単語移動における<code>e</code>(<code>evil-forward-word-end</code>)のように, シンボルの末尾で止まるコマンドも定義できます.
>|lisp|
(evil-define-motion evil-forward-symbol-end (count)
  "Move the cursor to the end of the COUNT-th next symbol."
  :type inclusive
  (evil-move-end count #'evil-move-symbol nil t))

(evil-define-motion evil-backward-symbol-end (count)
  "Move the cursor to the end of the COUNT-th previous symbol."
  :type inclusive
  (evil-move-end (- (or count 1)) #'evil-move-symbol nil t))
||<
シンボルの末尾はオペレータの適用範囲に含めて欲しいので, <code>evil-define-motion</code>に渡す<code>:type</code>は<code>inclusive</code>です. また, <code>evil-move-end</code>の呼出しでは, <code>inclusive</code>な場合には第4引数で指定する必要があります.

以上で, シンボル単位の移動コマンドが定義できました. 実際には, たとえば行末のシンボルにカーソルがあるときに, 次の行の空白をオペレータの適用範囲に含めないようにしたりといった細かな調整をすべきかもしれませんが, おおまかなところとしてはこれで完成です.

><h5 id="tutorial2-text-object">テキストオブジェクト</h5><

<code>evil-move-symbol</code>のように, 一定の条件を満たすように作られた関数は, Evilの内部関数<code>evil-an-object-range</code>と<code>evil-inner-object-range</code>を使って, そのままテキストオブジェクトに変換できます.
>|lisp|
(evil-define-text-object evil-a-symbol (count &optional beg end type)
  "Select a symbol."
  (evil-an-object-range count beg end type #'evil-move-symbol))

(evil-define-text-object evil-inner-symbol (count &optional beg end type)
  "Select inner symbol."
  (evil-inner-object-range count beg end type #'evil-move-symbol))
||<

ただし, <code>evil-an-object-range</code>と<code>evil-inner-object-range</code>を使ってテキストオブジェクトを作る場合は, 渡す関数が次の条件を満たす必要があります.
+ 引数として<code>count</code>のみをとる
+ <code>count</code>が正のときは, オブジェクトの末尾が<code>count</code>個過ぎた後の, 最初のオブジェクトの先頭に移動する(ただし, 最初からカーソルがオブジェクトの末尾にあるときは, そのオブジェクト<strong>も数える</strong>)
+ <code>count</code>が負のときは, <code>count</code>個前のオブジェクトの先頭に移動する(ただし, 最初からカーソルがオブジェクトの先頭にあるときはそのオブジェクト<strong>は数えない</strong>)
+ 返り値は, <code>count</code>で指定された回数と実際に移動できた回数の差(つまり成功時は0になる)
<code>evil-move-symbol</code>はこの条件を満たすように作っていたわけです.

><h5 id="tutorial2-keymap">キーマップ</h5><

定義されたコマンドを簡単なキーで使えるようにする場合, たとえば<code>,w</code>等に割り当てるなら次のようにします.
>|lisp|
(define-key evil-motion-state-map (kbd ",") (make-sparse-keymap))
(define-key evil-motion-state-map (kbd ",w") #'evil-forward-symbol-begin)
(define-key evil-motion-state-map (kbd ",b") #'evil-backward-symbol-begin)
(define-key evil-motion-state-map (kbd ",e") #'evil-forward-symbol-end)
(define-key evil-motion-state-map (kbd ",ge") #'evil-backward-symbol-end)
(define-key evil-outer-text-objects-map (kbd ",w") #'evil-a-symbol)
(define-key evil-inner-text-objects-map (kbd ",w") #'evil-inner-symbol)
||<

><h4 id="tutorial3">例3: 同じ文字の間を表すオブジェクト</h4><

<code>evil-an-object-range</code>や<code>evil-inner-object-range</code>は用いずに, いちからテキストオブジェクトを自作する例として, 同じ文字の間を表すオブジェクトを定義してみましょう. <code>vif<em>c</em></code>で文字<code><em>c</em></code>に囲まれた範囲を選択できるようにするのが目標です. <code>vaf<em>c</em></code>のときは<code><em>c</em></code>そのものも選択されるように, <code>vif<em>c</em></code>した後にもう一度<code>if<em>c</em></code>した場合は選択範囲を拡大することにしましょう.

テキストオブジェクトを定義するには<code>evil-define-text-object</code>を使います. 定義されるテキストオブジェクトは引数<code>count</code>と, オプショナル引数<code>beg</code>, <code>end</code>, <code>type</code>を取ります. オプショナル引数は, 既に選択範囲がある場合に渡されます. 実際の処理は<code>evil-between-range</code>という関数に任せることにしましょう.
>|lisp|
(evil-define-text-object evil-a-between (count &optional beg end type)
  "Select range between a character by which the command is followed."
  (evil-between-range count beg end type t))

(evil-define-text-object evil-inner-between (count &optional beg end type)
  "Select inner range between a character by which the command is followed."
  (evil-between-range count beg end type))
||<
<code>evil-between-range</code>の第5引数で, 入力された文字<code><em>c</em></code>そのものも選択範囲に含めるかどうかを指定することにしました.

><h5 id="tutorial3-simple">単純な実装</h5><

<code>evil-between-range</code>がやるべきことは, <code>count</code>個前と後の文字<code><em>c</em></code>を見つけて, 見つけた位置を表す範囲オブジェクトを返すことです. 文字の入力を受け取るには<code>evil-read-key</code>, 回数指定で文字を探すには<code>evil-find-char</code>, 範囲オブジェクトをつくるには<code>evil-range</code>を使います.
>|lisp|
(defun evil-between-range (count beg end type &optional inclusive)
  (ignore-errors
    (let ((count (abs (or count 1)))
          (ch (evil-read-key))
          beg-inc end-inc)
      (save-excursion
        (evil-find-char (- count) ch)
        (setq beg-inc (point)))
      (save-excursion
        (backward-char)
        (evil-find-char count ch)
        (setq end-inc (1+ (point))))
      (if inclusive
          (evil-range beg-inc end-inc)
        (evil-range (1+ beg-inc) (1- end-inc))))))
||<
文字が見つからなかった場合はエラーになるので, 全体を<code>ignore-errors</code>で囲っておきます. 文字を探す間のカーソル移動が後に影響しないように<code>save-excursion</code>で囲うようにもします. 負の回数には意味がないので絶対値を取ります. 後方に文字を探すときは, カーソルがその文字の上にあるときを考慮して1文字戻ってから探します. 第5引数が<code>t</code>のときは, 見つかった文字の位置も含めた範囲を, それ以外のときは1文字ずつ縮小した範囲を返します.

><h5 id="tutorial3-extend">選択範囲の拡大</h5><

この定義のままだと, 常にカーソル位置から始めて文字を探しますが, <code>beg</code>と<code>end</code>が指定されたとき(既に範囲が選択されているとき)は, 既存の選択範囲の開始位置から前に, 終了位置から後ろに探して, 結果的に選択範囲が拡大されるようにしなければなりません. また, 選択範囲がちょうど指定の文字の間の範囲を表している場合は, すぐ外側の文字が見つかって, 選択範囲が拡大されなくなってしまうので, 特別な取り扱いが必要です. このような点を考慮して修正したのが以下のコードです.
>|lisp|
(defun evil-between-range (count beg end type &optional inclusive)
  (ignore-errors
    (let ((count (abs (or count 1)))
          (beg (and beg end (min beg end)))
          (end (and beg end (max beg end)))
          (ch (evil-read-key))
          beg-inc end-inc)
      (save-excursion
        (when beg (goto-char beg))
        (evil-find-char (- count) ch)
        (setq beg-inc (point)))
      (save-excursion
        (when end (goto-char end))
        (backward-char)
        (evil-find-char count ch)
        (setq end-inc (1+ (point))))
      (if inclusive
          (evil-range beg-inc end-inc)
        (if (and beg end (= (1+ beg-inc) beg) (= (1- end-inc) end))
            (evil-range beg-inc end-inc)
          (evil-range (1+ beg-inc) (1- end-inc)))))))
||<

><h5 id="tutorial3-keymap">キーマップ</h5><

できあがったテキストオブジェクトを使うためのキーを割り当てるには, <code>evil-outer-text-objects-map</code>と<code>evil-inner-text-objects-map</code>を使います.
>|lisp|
(define-key evil-outer-text-objects-map "f" 'evil-a-between)
(define-key evil-inner-text-objects-map "f" 'evil-inner-between)
||<

><h4 id="tutorial4">例4: かなステート</h4><

ステートを自作する例として, ひらがな/カタカナを簡単に入力できるステートを定義してみましょう. Vimにはもともと[http://vim-jp.org/vimdoc-ja/digraph.html:title=ダイグラフ]の機能があり, Evilもこれをサポートしているので, 入力されたキーのかな変換にはこれを使うことにしましょう.

><h5 id="tutorial4-state">ステートの定義</h5><

新しいステートを定義するには<code>evil-define-state</code>マクロを使います. これを使うだけで, ステート・キーマップ・フックといったものが自動的に作成されます.
>|lisp|
(evil-define-state kana
  "Kana input state."
  :tag " <かな> "
  :cursor (bar . 2)
  :message "-- かな --"
  :enable (insert)
  (cond
   ((evil-kana-state-p)
    ;; かなステートに入ったときにすることはここに書く
    )
   (t
    ;; かなステートを抜けるときにすることはここに書く
    )))
||<
かなステートに入ったときは, 挿入ステートのキーマップも使えるようにしました(<code>:enable</code>キーワード) これによって<code>ESC</code>でノーマルステートに戻るなどの操作は自動的に有効になります. 他にどんなキーワードがあるかは, <code>M-x describe-function RET evil-define-state RET</code>や[https://bitbucket.org/lyro/evil/src/master/evil-states.el:evil-states.el]を参照して下さい. 今回は使いませんが, <code>evil-define-state</code>の本体では, ステートの有効化/無効化時に実行するコードを書くことができます.

><h5 id="tutorial4-command">かな変換コマンド</h5><

ダイグラフそのものを表すキーが入力されたときに, ダイグラフが表す文字を挿入するコマンド<code>evil-insert-kana</code>を作りましょう. たとえば,
>|lisp|
(define-key some-map (kbd "ka") #'evil-insert-kana)
||<
と割り当てることで, <code>ka</code>の入力に対して「か」が挿入されるようにします.

<code>this-command-keys</code>関数を使うと, 現在のコマンドを実行するために入力したキーを取得できるので, これをダイグラフとみなして文字を挿入するようにします.
>|lisp|
(evil-define-command evil-insert-kana ()
  "Insert Kanas."
  (interactive)
  (let ((str (this-command-keys)))
    (if (= 2 (length str))
        (let ((ch (evil-digraph (list (aref str 0) (aref str 1)))))
          (if (characterp ch)
              (insert-char ch)
            (error "Invalid digraph \"%s\"" str)))
      (error "Invalid keys \"%s\"" str))))
||<
入力されたキーが2文字でない場合, 有効なダイグラフでない場合はエラーにしています.

><h5 id="tutorial4-keymap">キーマップ</h5><

まず, 日本語のダイグラフの範囲に, <code>evil-insert-kana</code>を割り当てます. 割り当てるキーマップは<code>evil-kana-state-map</code>です.
>|lisp|
(defun evil-kana-make-digraph-patterns (c1 c2)
  (let (result)
    (dolist (first (list (string c1) (upcase (string c1))))
      (dolist (second (list (string c2) (upcase (string c2))))
        (push (concat first second) result)))
    result))

(let* ((vowels '(?a ?i ?u ?e ?o))
       (consonants '(?k ?s ?t ?n ?h ?m ?y ?r ?w ?g ?z ?d ?b ?v))
       (singles '(?n)))
  (dolist (v vowels)
    (define-key evil-kana-state-map (format "%c5" v) #'evil-insert-kana)
    (define-key evil-kana-state-map (format "%c6" v) #'evil-insert-kana)
    (define-key evil-kana-state-map (upcase (format "%c5" v))
      #'evil-insert-kana)
    (define-key evil-kana-state-map (upcase (format "%c6" v))
      #'evil-insert-kana)
    (dolist (c consonants)
      (dolist (str (evil-kana-make-digraph-patterns c v))
        (define-key evil-kana-state-map (read-kbd-macro str)
          #'evil-insert-kana))))
  (dolist (s singles)
    (define-key evil-kana-state-map (format "%c5" s) #'evil-insert-kana)
    (define-key evil-kana-state-map (format "%c6" s) #'evil-insert-kana)))
||<

ノーマルステートから<code>C-k</code>でかなステートに移れるようにします.
>|lisp|
(define-key evil-normal-state-map (kbd "C-k") #'evil-kana-state)
||<

><h4 id="reference">リファレンス</h4><

><h5 id="reference-state">ステート</h5><

:<code>(evil-define-state <em>name</em> <em>doc</em> <em>keywords...</em> <em>body...</em>)</code>:ステートを定義します. ステート切り替えコマンド<code>evil-<em>name</em>-state</code>, ステート判別コマンド<code>evil-<em>name</em>-state-p</code>, キーマップ<code>evil-<em>name</em>-state-map</code>などが自動的に定義されます. <code><em>keywords</em></code>にキーワードを指定することで, モードラインに表示するステートのタグ, カーソル形状, フックなども設定できます. 詳しくは<code>M-x describe-function RET evil-define-state RET</code>を参照して下さい.

><h5 id="reference-command">コマンド</h5><

:<code>(evil-define-command <em>name</em> (<em>args...</em>) <em>doc</em> <em>keywords...</em> <em>body</em>)</code>:コマンドを定義します. キーワードには<code>:repeat</code>, <code>:keep-visual</code>など, コマンドをEvil化するための設定を書きます. コマンドが引数をとる場合は, <code><em>body</em></code>の先頭に<code>(interactive "<em>code</em>")</code>と書きます. <code><em>code</em></code>に指定する内容は引数の種類によってに決まり, <code>evil-define-interactive-code</code>で定義されているものか, [http://www.gnu.org/software/emacs/manual/html_node/elisp/Interactive-Codes.html:title=Emacs標準のもの]を指定します. <code>evil-define-interactive-code</code>で定義されているものにどんなものがあるかは, [https://bitbucket.org/lyro/evil/src/master/evil-types.el:title=evil-types.el]を参照して下さい.
:<code>(evil-define-interactive-code "<em>code</em>" (<em>prompt</em>) <em>doc</em> <em>keywords...</em> <em>body</em>)</code>:コマンド引数のコードを定義します. <code>(<em>prompt</em>)</code>は省略可能で, ユーザからの入力を待つ場合に指定します. <code><em>prompt</em></code>は, <code>interactive</code>で指定されたプロンプト文字列を受け取るための変数名です. キーワードはExコマンドの引数のコードを定義する場合に使います. <code><em>body</em></code>には, 実際に引数を計算するための式を書きます.

><h5 id="reference-operator">オペレータ</h5><

:<code>(evil-define-operator <em>name</em> (<em>beg</em> <em>end</em> &optional <em>type</em> <em>args...</em>) <em>doc</em> <em>keywords...</em> <em>body</em>)</code>:オペレータを定義します. 最初の2つの引数は選択範囲の開始位置と終了位置を受け取ります. 3つ目の引数を受け取る場合は, 選択種別が渡されます. これ以外の方法で引数を受け取る場合は<code><em>body</em></code>の先頭に<code>(interactive "<em>code</em>")</code>と書きます. <code><em>code</em></code>の意味は<code>evil-define-command</code>と同じです. キーワードには<code>:repeat</code>, <code>:motion</code>, <code>:move-point</code>, <code>:jump</code>などを指定します. <code>:keep-visual t</code>と<code>:suppress-operator t</code>は自動的に設定されます. <code><em>body</em></code>にはオペレータの操作を書きます.

><h5 id="reference-motion">モーション</h5><

:<code>(evil-define-motion <em>name</em> (<em>count</em> <em>args...</em>) <em>doc</em> <em>keywords...</em> <em>body</em>)</code>:モーションを定義します. 最初の引数は繰り返し回数を受け取ります. これ以外の方法で引数を受け取る場合は<code><em>body</em></code>の先頭に<code>(interactive "<em>code</em>")</code>と書きます. <code><em>code</em></code>の意味は<code>evil-define-command</code>と同じです. キーワードには<code>:type</code>, <code>:jump</code>などを指定します. <code>:keep-visual t</code>は自動的に設定されます. <code><em>body</em></code>には実際の移動操作を書きます.
:<code>(evil-define-union-move <em>name</em> (<em>count</em>) <em>moves...</em>)</code>:複数の移動操作のうち, 移動距離が最小のものを採用する移動操作を定義します. <code><em>moves</em></code>には移動操作を書きます. それぞれの移動操作は, 指定回数の移動に成功したときは0を返さなければなりません. 0以外の値を返した場合, その移動操作は採用されません.

><h5 id="reference-text-object">テキストオブジェクト</h5><

:<code>(evil-define-text-object <em>name</em> (<em>count</em> &optional <em>beg</em> <em>end</em> <em>type</em> <em>args...</em>) <em>doc</em> <em>keywords...</em> <em>body</em>)</code>:テキストオブジェクトを定義します. 最初の引数は繰り返し回数, 2番目以降は選択範囲を受け取ります. これ以外の方法で引数を受け取る場合は<code><em>body</em></code>の先頭に<code>(interactive "<em>code</em>")</code>と書きます. <code><em>code</em></code>の意味は<code>evil-define-command</code>と同じです. キーワードには<code>:type</code>, <code>:extend-selection</code>などを指定します. デフォルトでは<code>:extend-selection t</code>に設定されます. <code><em>body</em></code>には新しい選択範囲を表すオブジェクトを返す式を書きます. 選択範囲は, 選択開始位置, 終了位置で始まるリストで, 3つ目の要素がある場合は選択種別です. このリストは通常<code>evil-range</code>関数で作ります.

><h5 id="reference-utility">補助関数</h5><

:<code>(evil-apply-on-block <em>func</em> <em>beg</em> <em>end</em> <em>pass-columns</em> <em>args...</em>)</code>:関数<code><em>func</em></code>を, <code><em>beg</em></code>から<code><em>end</em></code>までの矩形範囲に適用します. <code><em>func</em></code>は2引数以上の関数でなければなりません. <code><em>pass-columns</em></code>が<code>nil</code>のときは<code><em>func</em></code>の最初の2引数にはバッファ内の位置が渡され, それ以外の場合は列位置が渡されます. <code><em>args...</em></code>は<code><em>func</em></code>への3つ目以降の引数です.
:<code>(evil-move-beginning <em>count</em> <em>forward</em> &optional <em>backward</em>)</code>:次の<code><em>count</em></code>番目のオブジェクトの先頭まで移動します. <code><em>forward</em></code>と<code><em>backward</em></code>にはオブジェクト間を移動する関数を渡します. どちらの関数も回数を引数に取ります. もしどちらかの関数が指定されなかった場合は, 指定された方の関数に符号が逆の回数を指定したものがもう片方の関数として使われます. 関数は指定された回数の移動が成功すれば0を, そうでなければ残り回数を返す必要があります.
:<code>(evil-move-end <em>count</em> <em>forward</em> &optional <em>backward</em> <em>inclusive</em>)</code>:次の<code><em>count</em></code>番目のオブジェクトの末尾まで移動します. <code><em>forward</em></code>と<code><em>backward</em></code>にはオブジェクト間を移動する関数を渡します. どちらの関数も回数を引数に取ります. もしどちらかの関数が指定されなかった場合は, 指定された方の関数に符号が逆の回数を指定したものがもう片方の関数として使われます. 関数は指定された回数の移動が成功すれば0を, そうでなければ残り回数を返す必要があります. カーソルをオブジェクトの末尾に移動する場合は<code><em>inclusive</em></code>に<code>t</code>を指定します.それ以外の場合はカーソルはオブジェクトの末尾の次の位置に移動します.
:<code>(evil-an-object-range <em>count</em> <em>beg</em> <em>end</em> <em>type</em> <em>forward</em> &optional <em>backward</em> <em>range-type</em> <em>newlines</em>)</code>:移動関数からオブジェクトの範囲を作ります. <code><em>beg</em></code>, <code><em>end</em></code>, <code><em>type</em></code>には現在の選択範囲を指定します. <code><em>forward</em></code>, <code><em>backward</em></code>には移動関数を指定します. 移動関数は回数を引数に取り, 指定された回数の移動が成功すれば0を, そうでなければ残り回数を返す必要があります.
:<code>(evil-inner-object-range <em>count</em> <em>beg</em> <em>end</em> <em>type</em> <em>forward</em> &optional <em>backward</em> <em>range-type</em> )</code>:移動関数からオブジェクトの範囲を作ります. <code><em>beg</em></code>, <code><em>end</em></code>, <code><em>type</em></code>には現在の選択範囲を指定します. <code><em>forward</em></code>, <code><em>backward</em></code>には移動関数を指定します. 移動関数は回数を引数に取り, 指定された回数の移動が成功すれば0を, そうでなければ残り回数を返す必要があります.
:<code>(evil-range <em>beg</em> <em>end</em> &optional <em>type</em> <em>properties...</em>)</code>:範囲オブジェクトを作ります.
:<code>(evil-read-key &optional <em>prompt</em>)</code>:ユーザからのキー入力を読み取ります. <code><em>prompt</em></code>を指定すると入力待ちの間プロンプトを表示します. Emacs標準の<code>read-key</code>関数と違い, <code>evil-read-key-map</code>が有効になります.
:<code>(evil-with-single-undo <em>body</em>)</code>:<code><em>body</em></code>内でのバッファの変更を, 1回で元に戻せるようにします.

** おわりに

Evilの拡張方法を修得すれば, ますますEvilを自由に使うことができます. また, 拡張機能を書いていくことでEvilの内部のしくみにも詳しくなります. 一通り理解が深まったら, ぜひEvilのプラグインの開発, 本家Evilの開発への参加を目指してみて下さい.

><ol class="local-pager">
   <li>[http://tarao.hatenablog.com/entry/20130303/evil_intro:title=導入編]</li>
   <li>[http://tarao.hatenablog.com/entry/20130304/evil_config:title=設定編]</li>
   <li class="current">拡張編</li>
   <li>[http://tarao.hatenablog.com/entry/20130306/evil_appendix:title=付録]</li>
</ol><
