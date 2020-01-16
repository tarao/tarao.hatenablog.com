---
Title: 'プログラムで解く数学パズル: 囚人とスイッチの部屋の問題 - 解答の自動チェックのしくみ'
Category:
- golang
- article
Date: 2018-12-18T10:24:47+09:00
URL: https://tarao.hatenablog.com/entry/prisoners-switch
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/10257846132686591795
---

この記事は[https://qiita.com/advent-calendar/2018/hatena:title=はてなエンジニア Advent Calendar 2018]の18日目の記事です. 昨日はid:Windymelt:detailの[https://blog.3qe.us/entry/2018/12/17/145555:title=Smart::Argsのパーサを書いた]でした. 明日の担当はid:hokkai7go:detailです.

他の担当者の記事は割と業務っぽいものが多いですが, 今回は趣味っぽいゆるゆるのネタです. 社内でとある数学パズルを紹介したところAdvent Calendarに書いてくれとリクエストがあったので, 紹介します. 単に問題を紹介するだけでは面白くないので, コードを書いて解答できるようにしてみました.

====
** 問題

>>
あなたは100人の囚人の一人です. 全員で以下のようなゲームをして, 見事勝利できれば全員釈放, 負ければ全員死刑となります.

- ゲーム開始と同時に全員別々の独房に入ります
-- 独房内や通路で他の囚人とやりとりすることはできません
- ランダムに1人ずつ<strong>スイッチの部屋</strong>に呼ばれます
-- 十分な時間待てば同じ人が何度でも呼ばれます
-- いま誰が呼ばれているかは本人以外にはわかりません
-- 部屋には<strong>スイッチ</strong>が2つ(AとB)ある以外はなにもありません
-- ゲームに参加中の囚人以外がこの部屋に入ることはありません
- <strong>スイッチの部屋</strong>では以下のことができます
-- 2つある<strong>スイッチ</strong>のon/offの状態を確認する
-- 2つある<strong>スイッチ</strong>のいずれか, もしくは両方のon/offを切り替える
- 囚人はいつでも「全員<strong>スイッチの部屋</strong>に入った!」と<strong>宣言</strong>することができます
-- 本当であれば勝利となります
-- まだ一度も部屋に入っていない人がいたらその時点で負けです
- ゲーム開始時点での<strong>スイッチ</strong>の状態はランダムに決められます

ゲームを開始する前に, 囚人全員で集まって作戦を立てることができます. 必ず勝利できる作戦を考えてください.

答えがわかって物足りなければ以下についても考えてください.

- <strong>スイッチの部屋</strong>に入った総回数(の期待値)がなるべく少なくなるような作戦を立ててください
- <strong>スイッチ</strong>を1つしか使わずに勝利する方法を考えてください
<<

*** プログラマ的発想

プログラマなら脊髄反射で「スイッチ2つ=情報2ビットで4通りの値しか持てないんだから100数えるのは無理!」って思っちゃうやつですね. もちろん問題になっているからにはちゃんと解けます. べつになぞなぞとかでもありません.

なんらか答えはあるということで, 気にせずプログラマ脳でちょっと考えてみると, この「作戦を考えてください」というのは「アルゴリズムを考えてください」ということのように思えます. 100人の囚人がそれぞれどのようなアルゴリズムに沿って行動すれば勝利できるかという問題.

そうなるとコードを書いて解答してみたくなります. コードを書いて解答できるなら, 解答が正しいかどうかのチェックも自動化したくなります. ということで, しました!

[https://github.com/tarao/prisoners-switch:embed]

Pull Requestを送ると自動で採点されて入室回数の少なさや使用したスイッチの少なさでスコアがつき, 順位なども[https://tarao.github.io/prisoners-switch/?pr=7&url=https%3A%2F%2Fgithub.com%2Ftarao%2Fprisoners-switch%2Fpull%2F7&user=tarao&avatar=https%3A%2F%2Favatars2.githubusercontent.com%2Fu%2F132569%3Fv%3D4&state=success&score=83964&steps=1603593&sw=2&msg=All%20game%20passed&rank=23&timestamp=2018-12-16T12%3A58%3A51Z:title=こんなかんじ]で出ます.

><iframe src="https://tarao.github.io/prisoners-switch/?pr=7&url=https%3A%2F%2Fgithub.com%2Ftarao%2Fprisoners-switch%2Fpull%2F7&user=tarao&avatar=https%3A%2F%2Favatars2.githubusercontent.com%2Fu%2F132569%3Fv%3D4&state=success&score=83964&steps=1603593&sw=2&msg=All%20game%20passed&rank=23&timestamp=2018-12-16T12%3A58%3A51Z" width="800px" height="400px" frameborder="no" scrolling="no"></iframe><

<img src="https://cdn-ak.f.st-hatena.com/images/fotolife/t/tarao/20181217/20181217001954_original.png" style="display:none" />

*** 解答方法

実際に, 誰でも以下のようにして解答することができるので, 是非やってみてください.

+ [https://golang.org/:title=Golang]環境を用意
+ [https://github.com/tarao/prisoners-switch:title=tarao/prisoners-switch]を<code><var>yourname</var>/prisoners-switch</code>にFork
+ <code>go get github.com/tarao/prisoners-switch</code>
-- <code><var>yourname</var>/prisoners-switch</code>ではなく<code>tarao/prisoners-switch</code>な点に注意
+ <code>cd "$GOPATH"/src/github.com/tarao/prisoners-switch</code>
+ <code>git remote add <var>yourname</var> git@github.com:<var>yourname</var>/prisoners-switch</code>
+ <code>git checkout -b <var>your-answer</var></code>
+ <code>strategy/my_strategy.go</code>を編集して解答して<code>git commit</code>
+ <code>git push --set-upstream <var>yourname</var> <var>your-answer</var></code>
+ <code><var>your-answer</var></code>ブランチを<code>tarao/prisoners-switch</code>の<code>master</code>ブランチにPull Requestする

解答は以下の制限に合致している必要があります.

- 書き換えてよいのは<code>strategy/</code>以下のみ
-- ファイルを追加してもよい
-- 変更ファイル数の上限は20
- <code>github.com/tarao/prisoners-switch/rule</code>以外を<code>import</code>してはいけない

うまく解けているか手元で確認したい場合は<code>verifier/run</code>を実行すると確認できます.

Pull Requestで解答する方式なので, 当たり前ですが他の人の解答も見えます. 自力で解いた方が当然おもしろいでしょう. より高いスコアを目指すというのもよさそうです.

** 出典

僕がこの問題を知ったのは10年くらい前にid:tozima:detailさんに聞いたからで, id:tozima:detailさんはバーで自称数学科5回生((関西なので5回生というのは5年生のことです))に絡まれて問題を出されたそうで, 長らく出自不明な問題でした. 今回この記事を書くにあたって改めて調べてみたら, ある程度まで判明したので書き残しておきます.

結論から言うと, インターネットで辿れる範囲の初出は[http://www.research.ibm.com/haifa/ponderthis/index.shtml:title=IBM Researchの月刊数学パズル]でした.

[https://www.research.ibm.com/haifa/ponderthis/challenges/July2002.html:title=IBM Research | Ponder This | July 2002 Challenge]

細かな差異はあって,

- 人数が23人
- 一度の入室では1つのスイッチしか操作できず, また, いずれかのスイッチを操作しなければならない((操作してもしなくてもよいという問題設定でスイッチ1つの解があるなら, 必ず操作しなければならない問題設定でスイッチ2つの解があるのは自明ですね. 操作したくないときに身代わりとして操作する方のスイッチを決めておく, とすればよいからです.))

というものだったようです.

>http://www.research.ibm.com/haifa/ponderthis/index.shtml:title>
This puzzle has been making the rounds of Hungarian mathematicians'parties.
<<

とあり, この月刊パズルのために新規に考案したものではないのも確かですが, これ以上の出典は辿れません.

これより最近のもので言及しているものも多くあります.

- [https://www.ocf.berkeley.edu/~wwu/papers/100prisonersLightBulb.pdf:title=100 Prisoners and a Light Bulb. William Wu. December 2002.]
-- 100人
-- 勝利するとMENSAに入れるというバリエーションはこの人によるもの
- [https://www.amazon.co.jp/dp/1568812019:title=Mathematical Puzzles. Peter Winkler. 2003.]
-- 書籍
-- [http://www.cut-the-knot.org/Probability/LightBulbs.shtml:title=ここ]によると著者は出典としてIBM(おそらく上に挙げたもの)や[http://www.msri.org/:title=MSRI]を挙げているらしい
- [https://www.amazon.co.jp/dp/B011I9GWDK:title=One Hundred Prisoners and a Light Bulb (English Edition). Hans van Ditmarsch, Barteld Kooi, Elancheziyan. 2015.]
-- 原著はオランダ語らしい (ちょっとどれなのかわからなかった)
-- [http://www.msri.org/system/cms/files/204/files/original/Emissary-2016-Spring-Web.pdf:title=MSRIのニュースレター]によると原著は2004年のMath HorizonsなのでWinklerの方が先

** 自動チェックのしくみ

自動チェックされるとこのPull Requestのようにチェックマークがつきます:
https://github.com/tarao/prisoners-switch/pull/7
(正しい解答のコードは含めていませんが, 例示のためにインチキして無理矢理に正解のステータスにしています)

自動チェックは次のようになっています.

+ Pull RequestがくるとTravis CIで<code>verifier/run</code>が実行される
-- <code>strategy/</code>以下に<a href="https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/run#L14">余計な<code>import</code>がないかチェック</a>
-- [https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/main.go#L63-L73:title=ゲームを複数回実行してすべて勝利することを確認], スコアなどを出力
+ Travis CIの[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/.travis.yml#L12..L14:title=webhookがGoogle Apps Script (GAS)をトリガ]
+ GASが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L63-L91:title=ビルド情報やPull Request情報を集める]
+ GASが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L243-L254:title=改竄チェック]
+ GASが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L312-L368:title=スコアなどの情報をGoogle Sheetsに保存]
+ Google Sheetsが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L354:title=スコアから順位を計算]
+ GASが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L184-L203:title=GitHubのコミットステータスを更新]
+ (GASが[https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L370-L412:title=Slackに通知])

これとは別に, [https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/report.gs#L100-L157:title=順位の変動をGitHubのコミットステータスに反映するための定期処理]もGASで動いています.

GitHubのコミットステータスの"Details"で見られるスコアや順位の出る画面は, 実はGitHub Pagesに配置した静的ファイルで, クエリパラメータからすべての情報を受け取ってJavaScriptでレンダリングしているだけです. なので「順位の変動をGitHubのコミットステータスに反映」というのは単にリンクURLを変えているだけです.

Google Sheets (Googleスプレッドシート)に保存するのは, もともとは単に解答した人たちのスコアを一覧したいだけでしたが, いろいろ作っているうちに順位も出せたら面白そうと思い, どうにかできないかと思ったら<code>RANK</code>関数使うだけでできて, とにかくべんりですね.

こうして, とくに採点や結果の表示のために専用のサーバを立てるようなことはしないで済みました.

** チート対策

今回いちばん面白かったところです. コードで解答となると, 解答する側がかなりいろいろなことができてしまうので, 題意に沿わない方法で「正解」の判定を出すことができてしまう可能性があり, うまく対策する必要があります. どんな対策をしたか, 例とともに見ていきましょう.

対策の漏れを見つけた人は<code>framework</code>ブランチに対して対策をPull Requestしてください(<code>master</code>ブランチにすると自動チェックが走ってしまうからというだけなので修正案をくれるならなんでもよいです). もちろんその前にチートで高得点を叩き出す, というのをやってもかまいません(対策次第得点は無効にします).

*** 「他の囚人とやりとりすることはできません」

[https://github.com/tarao/prisoners-switch/pull/4:title=チート例]

コード化された問題を見たら, まずグローバル変数にどの囚人が部屋に入ったか記録していって, 全員が入っていたら勝利宣言する, というのをやってみるのではないでしょうか. なにか対策されているかもしれないけど, されてなかったらそれで解けるのは明らかなので, ひとまず手癖で試してみたくなります.

一方, 対策する側としてはこれは厄介な問題です. 素直に考えると各囚人のコードを別々のサンドボックスで実行しないといけない? どうやって? となります.

あんまり大掛かりなことをしたくはなかったので, 今回は発想を転換して, 「他の囚人とやりとりすることはできません」を直接実現するのではなく, 「やりとりしようと思えばできるが, 話した相手は別の並行世界の囚人かもしれない」としました.  ゲームを複数回同時に回して, 同じ番号の囚人が複数人生まれるようにしました. 囚人のインスタンスの初期化順序もばらばらにして, どの囚人がどの囚人と同じ回のゲームにいるのかわからなくしました.

実はこれだけでは「全ゲームで勝利宣言できるだけの入室があったら宣言する」ということができてしまいます. たとえば10ゲーム走っているなら, 同じ番号でインスタンスが別の囚人が10人とも部屋に入ったかどうか確かめればよいのです(上の例は実際そうやっているものです).

そのため, 最終的な対策としては「絶対に勝利できないゲーム」もいくつか混ぜておきます(もちろんそれらのゲームへの勝利は正解判定の条件からは除外します). これらのゲームにはそれぞれ少なくとも1人, 絶対に部屋に呼ばれない囚人がいます. 全ゲームの全囚人が部屋に入ったことを確かめようとすると永久に勝利宣言できなくなります. 「絶対に勝利できないゲーム」の存在はある意味出題側のチートという感じですね.

なんかたぶん一般論として「他人とやりとりすることはできない」と「あらゆる並行世界の他人とやりとりしてしまう」は, ある意味では同値(?)とか双対(?)みたいなことが形式的に言えたりするんじゃないかと妄想しましたが, 妄想しただけで調べたり考えたりは何もしていません. ご存知の方がいたらこっそり教えてください.

*** 不正な結果の出力その他

[https://github.com/tarao/prisoners-switch/pull/6:title=チート例]

コードを自由に書けるということは, 偽のチェック結果を出力してすぐ終了したり, もしくはリフレクションやポインタ演算による不正なメモリアクセスなどでチェックを実行する部分のコードを不正に操作することで無理矢理に正解判定に持っていくことができてしまいそうです.

今回はたまたま言語にはGoを選んだので, Goではこのようなことをするには特定のパッケージ(<code>fmt</code>とか<code>os</code>とか<code>reflect</code>とか<code>unsafe</code>とか)を<code>import</code>する必要があるはずです. そこで, ゲームのルール定義以外のパッケージの<code>import</code>を一切禁止することにしてみました.

<code>go list -f '{{range $imp := .Imports}}{{printf "%s\n" $imp}}{{end}}'</code>とすると<code>import</code>されたパッケージ一覧が取得できるので, [https://github.com/tarao/prisoners-switch/blob/145f4023108965d76bde903c6798df8ed4296480/verifier/check_imports#L12-L14:title=余計なものがあったらダメ]ということにしています.

*** 自動チェック方法そのものの改竄

[https://github.com/tarao/prisoners-switch/pull/5:title=チート例]

そもそも不正な<code>import</code>がないかどうかのチェックとか, 判定結果を出力している部分とか, 自動チェック機構そのものもリポジトリに含まれているので, それらを改竄されてしまったらどうしようもありません.

このために, 変更のあった部分に自動チェック機構のコードが含まれていないかチェックしています. これは単に[https://developer.github.com/v3/pulls/#list-pull-requests-files:title=GitHubのAPIで変更のあったファイルのリストを取得]すればよいですね. この部分はTravis CIで実行される<code>verifier/run</code>の中でやっても意味がない(改竄されてしまう)ので, GASでやります.

ただし, Travis CIのwebhookのペイロードは捏造されている可能性があるので注意が必要です. たとえば, ビルドIDやPull RequestのURLなどは正しく正解しているものを指しておいて, コミットステータスの対象のSHA1だけ別のものにすることで, 本来は正解していないものを正解のステータスにしようとしているかもしれません.

このようなことを避けるために, GAS側ではトリガとなるビルドIDのみ利用して, それ以外のPull Requestの情報, 実行結果のログ(正解判定やスコアの情報), 変更のあったファイル一覧などはすべて改めてAPIで取得します. こうすれば, ペイロードが捏造されても単に特定のビルドのチェックが余計に走るだけで, 結果を改竄することはできません.

<a href="https://docs.travis-ci.com/user/notifications#verifying-webhook-requests">Travis CIのwebhookには改竄防止のために<code>Signature</code>ヘッダというのがついてきます</a>が, GASで標準で使えるライブラリだけでこれを検証するのはちょっと面倒だったので使いませんでした. あとは, Travis CIではforked pull requestに対して<code>secure</code>にした環境変数は(セキュリティ上の理由で)使えませんが, webhookを<code>secure</code>にしておく(ログとかには一切表示しない)というのだけでもできるともうちょっと楽になりそうです(しかしpull requestに対しては使えないようでした).

** おわりに

今回は特定の問題に対してPull Requestで解答できるようにしてみましたが, もうちょっと工夫すれば大枠のしくみは使いまわして, 低コストでこういう問題をいくつも作ることができるかもしれません. 他の人の解答も見えてしまうので, コンテストとか大学の課題みたいなものには使えませんが, 遊びとしてはこれくらいでもいいような気がします.

** 追記

24時間経たないうちに最初の正解者が出ました. [https://github.com/tomohisaota:title=@tomohisaota]さんによる解答で, 効率(ステップ数の少なさ)もかなりよいです.

[https://github.com/tarao/prisoners-switch/pull/11:embed]

さらにすぐ後にスイッチ1つの正解も出してくれました. 拍手!

[https://github.com/tarao/prisoners-switch/pull/12:embed]

これまでに投稿された解答は以下で誰でも一覧できるようにしました.

[https://docs.google.com/spreadsheets/d/1qhZFYmxQyp2Z6GC2FmpmGmbKVggI83FLxMUtcw_hC9U/edit#gid=2009893292:embed]
