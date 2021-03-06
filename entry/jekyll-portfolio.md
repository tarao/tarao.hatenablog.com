---
Title: ポートフォリオをYAMLなどからJekyllで生成するようにした
Category:
- ruby
Date: 2020-01-17T11:00:00+09:00
URL: https://tarao.hatenablog.com/entry/jekyll-portfolio
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/26006613498615297
---

自分の過去の登壇・執筆情報の管理が面倒になってきたのと, Twitter等に貼ってあった自己紹介のページがあまりに得体が知れない感じになってしまっていたので, ポートフォリオというかプロフィールというか, そういうものを用意することにした.  静的ページでいいけど, 1次情報はYAMLかなにか, 管理しやすいものになっていると助かるので, まぁ[https://jekyllrb.com/:title=Jekyll]でいいでしょ, という感じでミニマルに作りはじめた.  ブログ記事とかGitHubのリポジトリとかもかいつまんで載ってるといいかもね, とやっていったらそこそこのボリュームになってしまった.  でも最近は歳を取ったのか趣味でコードを書くモチベーションが低くて全体的に意識が低い.

====
** できたもの

[https://tarao.orezdnu.org:embed]

:ソースコード:https://github.com/tarao/tarao.github.io/

** Jekyll

まず一番の目的としては, 文献リストを表示したい.  その際には, とくに論文や雑誌記事に関しては, 論文における正しい引用として認められるようなきちんとした書式になっていてほしい.  このニーズで一番信頼がおけるのはBibTeXだと思うものの, BibTeX形式でリストを管理したくはない.  たとえば連名の場合に著者名を<code>名前1 and 名前2 and 名前3</code>みたいに書きたくない.  そりゃあ出力はいい具合に読みやすい形式になっていてほしいけど, データを管理する上ではふつうに構造化データであってほしい.  リストを書かせろ.  データ管理に文芸的もなにもない.  そういうわけでYAMLで書きたい.  YAMLじゃなくてもいいけどJSONで書きたくはない.

YAMLで書いたものからいい具合に静的ページを生成するとなると, まぁJekyllだよね.  あんまりイマドキ感はない.  おっさんチョイス感がある.  いやJekyllをdisりたいわけではない.  適度に枯れていて, やりたいことがあったときに頑張ってフロンティアを開拓しなくて済みそうでよい.  ただそういう選び方の方法論がおっさんくさい.  意識が低いので仕方ない.

そういうわけでJekyllを使ったことはなかったのでざっとドキュメントを眺めて, YAMLから生成するのがサポートされているし, Liquidというテンプレートエンジンが乗ってるのでやりたいことはなんでもできそうだし, プラグイン書けるようなので凝ったことをしたくなっても困ることはなさそう, という辺りをおさえて作りはじめた.

** デザイン

[f:id:tarao:20200117000731p:image:w480:right]

Jekyllのテーマはいっぱい公開されていて, よさげなやつに乗っかれると楽だなぁと思った(なにせ意識が低い)けど, ブログ書く用のものが多い.  ポートフォリオ用のもあるけど乗っかろうとするとなんか見せ方が思ってたんと違う, となりそう.  とくに肝心の文献リストみたいなところは自分でなんとかするしかなさそう.

ちょっと探してすぐ諦めて(意識が低いと諦めも早い), 自分でデザインを起こすことにした.  と言っても意識が低いのでてきとーに出来合いのカラーテーマ的なものをパクりたい.  ふだんエディタで使っている[https://github.com/bbatsov/zenburn-emacs:title=ZenburnのEmacs移植版]の配色をパクることにした.

** 文献リスト

基本的には, タイトル, 著者, その他の情報を羅列するだけなんだけど, せっかくHTMLで整形されるのでタイトルや雑誌, イベント名などはリンクにしたい.  これはLiquidは単にテンプレートエンジンなのでやればできる.  問題は著者名で, 3人以上だったら

>||
A, B, and C
||<

みたいにするけど2人だったら

>||
A and B
||<

だし, 日本語だったら全員<code>,</code>区切りとか, いろいろある.  当然そういうことをやってくれる機能がJekyllにあったりはしないので<a href="https://github.com/tarao/tarao.github.io/blob/92a1af9887e08480500626ecd88b781b0fea9e9a/_plugins/filters/authors.rb"><code>authors</code>フィルタを自作</a>した.

という具合にいろいろやっていくと, ふと「これはつきつめていくとBibTeXの再発明になるのでは?」という気もしてくるけどぐっとこらえる. だからと言ってBibTeXを使うのは絶対に嫌だ.  この部分は一度やってしまえばあとはYAMLしか触らないから面倒なのは今だけだと自分に言い聞かせた.  出来上がってみると必要最小限でコンパクトに実現できている気はする.

** oEmbed

文献リストには登壇情報も載せたい.  リンク先は当然SlideShareやSpeaker Deck((どうして両方使っているかというと, もともとはSlideShareだけ使っていたのが, ふだんの方法で作ったPDFを新たにアップロードすると日本語が欠落してしまって, サポートに問い合わせても「PowerPointを使えば大丈夫なはずだよ」というトンチンカンな答えしか返ってこないので見限って乗り換えたから.))になる.  そうすると, そもそもスライドは埋め込めるよね, という気がしてきたのでやってみた.  やってみたというか, 自分でやるほどのモチベーションはないので, [https://gist.github.com/vanto/1455726:title=oEmbedの埋め込みをやってくれるプラグイン]を見つけてきた.

そのままだとビルドする度に毎回リクエストが飛んでめちゃくちゃ時間がかかるようになるのを回避するためにキャッシュしたり, 埋め込むべきものがないときはエラーで落ちるけど埋め込めるかどうか判断はしたくなくてURLがあったらとにかくoEmbedでの埋め込み表示を試すようにしたかったので, その辺りは手直しした.

なんかでもやってみるとスライドは小さすぎてあんまりべんりではないね...... SlideShareは最大化ボタンをいきなり押せるから多少べんりではある.  Speaker Deckに乗り換えたとこなのに.

** GitHubリポジトリ

文献リストやプロフィールなど, 一番の目的が達成されると欲が出てくる.  主なリポジトリとか出てると親切そう.  というかポートフォリオなんだから, ソフトウェアエンジニアとしての作品であるところのソフトウェアも並んでないとまずい気がする.

GitHubのリポジトリだったらどうせなら自動で一覧をとってきてノーメンテでいきたい.  「主」かどうかは★の数とかでいいとして, 個人リポジトリ以外にOrganizationになってるやつとかもある.  でもそういうのまで出すとなると, プルリク送っただけなのかコミット権を持ってるのかくらいは区別したい.

APIをいろいろ叩けばいいんだろうけど, あんまりいろいろ叩きたくない.  なにせ意識が低い.  GraphQLで取ってきたらいい具合にならないだろうか.  なってほしい.  いろいろ試してみると, 結論としては<code>contributionsCollection</code>で引っぱってくるとよさそうということがわかった.  これは最大1年間のコントリビューション(コミットとかプルリクとかissueとか)を取ってくるやつなので, 期間を変えて何回かリクエストすれば触ったことのあるリポジトリはぜんぶ取れる.

ちなみに, リポジトリ系のクエリ(<code>repositories</code>とか<code>repositoriesContributedTo</code>とか<code>topRepositories</code>とか)はOrganizationのやつは取れないとか最近のしか取れないとか, さまざまな理由でダメそうだった.

** ブログ記事

だいたい材料は揃ったけどもう一声.  何者なのかわかりやすいという意味では, このブログの記事の一覧があるとよさそう.  ぜんぶ出すのは多すぎるので主なものだけ.  最近の記事とかでもいいけど, どうでもいいものを見せられても困るだろうから主なものの方がいい.  「主」かどうかはブクマ数でいいだろう.  はてなブックマークの検索APIを使えばすぐできる.  プラグインをできる限り汎用的にしておきたかったのでRSSで取ってくることにした.

実は, ここだけでなく[https://developer.hatenastaff.com/:title=Haten Developer Blog]に記事を書いていることもたまにあるので, できることならそれも出したい.  こっちはそんなにたくさん書かないのでまぁぜんぶ出てもよいし, 書くとなったらどうでもいいことは書かないので新着記事でもよい.  このブログにはたくさんの人が記事を書いていて, 自分が書いたものは<a href="https://developer.hatenastaff.com/archive/author/tarao"><code>/archive/author/tarao</code></a>みたいなパスで一覧できる.

しかし, このページにはこの一覧を返すRSSやAPIの類はない.  はてなブログのソースコードも見たので間違いない.  そればかりか「このページのRSS欲しいよね」という話をid:hitode909:detailくんに300年くらい前からしていて「プルリクお待ちしております」と言われて「ですよねー」となったまま長い年月が過ぎた.  ついに, やるときがきたのか...?

そう思って一瞬だけやる気になったものの, やりたいことのシンプルさに対して思ったより考えることが多い.  キャッシュどうするとかページング必要かとか.  テスト書かないとさすがに怒られるよな, というか書かないと自分が不安だけどでも書きたくないでござる, とか.  その前に, はてなブログのローカル開発環境を整えるのめんどくさい.  なんか雑に既存の実装の真似をして「こういうイメージです」って渡したらブログチームで仕上げてくれないかなー, 無理だなー, 自分がそれされたらぜったい嫌だしなー.  となって, すべてのやる気を失った.

諦めればいいか, とおもったその時, 悪魔が囁いた.

>>
ひとまずスクレイピングでよくね?
<<

書いてみたら15分で[https://github.com/tarao/tarao.github.io/commit/f9a9c8ada899072b6dcbc172b45e904a0f7977d6:title=書けた]わ.  わっはっは.  うーん, 意識が低い.

><dl><dt>(追記)</dt>
<dd>
<p>...という話をブログチームの人にしたところ, なんと実装してくれた! 最高! べんり!</p>

[https://staff.hatenablog.com/entry/2020/02/06/174700:embed]
</dd></dl><

** GitHub Actions

プラグインをちょいちょい書いたりしたけど, そういうののテストをしっかり書いてCIを回す気はいっさいない.  なぜなら意識が低いから!  というかちゃんとできてるかどうかは生成されたペライチのページを見ればわかるし, 見た目の確認は必ずするはず.

とはいえ, デプロイ手順は簡単にしたい.  というか自動化したい.  でないと, たまにしか更新しないだろうから, どうしたらいいのか絶対に忘れる.  ついでにGitHub Actions使ってみるチャンス((社内で既にけっこう使われてるけど自分でやってみてなかった.))なのでやってみる.  Jekyllのビルドをやるやつ絶対あるやろ.  前処理のスクリプトでYAMLを生成してからビルドとかではなくぜんぶプラグインとして実装したのはこのためだったのだ!  探したらいい感じのがあった:

[https://github.com/helaili/jekyll-action:embed]

でもなんかこれを指定してもうまくいかない.  Bundlerのバージョンがどうとかで怒られる.  <a href="https://github.com/helaili/jekyll-action/blob/b61690ab6e2cfa44ebd039ae32f0d735b14eb967/Dockerfile#L14"><code>Dockerfile</code>を見ると<code>BUNDLER_VERSION</code>を固定してる</a>けどこれがいけないんじゃないの? とおもってforkしてその行を消したら動いた.  この対応でいいのかはわからん!  わかろうという気もあまりない.  なぜなら意識が低いから.  (正しい対応のしかたを知っている人がいたら教えてください.  というかプルリクください.)

本来ならfork版を使ってないでちゃんと調べて本家にプルリクするところだけど, どのみち外部提供のアクションに自分のトークン渡すの怖いからコード読んだ上でforkするつもりだったので, 「まぁどうせforkするんだし動けばいいや」となってしまった.

GitHub Actionsの感想としては, アクションをリポジトリから引っぱってこれて, ワークフローの定義自体は非常に簡潔にできてよいですね.  GitHubの★数や新しいリポジトリ, 新着ブログ記事があったら自動でアップデートしたいから, push時だけでなく定期実行もしたい, みたいなのもちょろっと書くだけで済んで簡単.  いちばん最初にチェックアウトしてくるアクションを書き忘れてハマったのはナイショだ!

** 感想

もともとは登壇情報などはプライベートGistにリストで書いていて, 書き足すのも見るのもなんかすごく不便だったので, 最高の状態になった.  ただあらためて全体を眺めてみると, けっきょくこいつは何者なのかというのがわからない感じがある.  なんか独特っぽいけど, 「ああ, この人はこれ系の人かぁ」というのが定まらない感じがある.  一言で言うとセルフブランディングに失敗している.  なんかしばらく前からそんな気はしていたけど, いよいよ可視化されてしまった......
