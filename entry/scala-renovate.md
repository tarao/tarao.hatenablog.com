---
Title: Scalaの依存ライブラリ更新はRenovateでもけっこうイケる
Category:
- scala
Date: 2020-12-20T12:00:00+09:00
URL: https://tarao.hatenablog.com/entry/scala-renovate
EditURL: https://blog.hatena.ne.jp/tarao/tarao.hatenablog.com/atom/entry/26006613666506407
---

この記事は, [https://qiita.com/advent-calendar/2020/hatena:title=はてなエンジニア Advent Calendar 2020]の20日目です. 昨日はid:Pasta-K:detailによる[https://blog.pastak.net/entry/2020/12/19/140000:title=ウェブブラウザにバグ報告をするときにやること]でした. 明日の担当はid:motemen:detailです.

Scalaで, 依存ライブラリのバージョンを上げるpull requestを自動的に作ってもらうソリューションとしては[https://github.com/scala-steward-org/scala-steward:title=scala-steward]があります. ScalaのことはScalaでやるのが確実そうなので安心感がありますね. ただ, scala-stewardを自分でホストするのは面倒だし, かと言って[https://xuwei-k.hatenablog.com/entry/2020/11/21/035306:title=GitHub Actionsで動くようにするのもそれなりに面倒]なようです.

Scalaに特化しない方法としては[https://github.com/renovatebot/renovate:title=Renovate]があります. これはいろんな言語のライブラリ等のバージョンをぜんぶ面倒見てくれてべんりです. GitHub Appsになっているのでポチポチ設定するだけで導入できます. 考えてみれば, Scalaのプロジェクトだってどうせ<code>Dockerfile</code>なども置いたりしているだろうし, Scalaのライブラリのバージョンだけ上げられればいいということの方が稀だと思います. だったらぜんぶRenovateに任せてしまえる方がいい. 不安要素としてはScalaに対するケアがどれくらい行き届いているか.

この記事では, Scalaで書かれたプロダクトで実際にRenovateを使ってみて行き当たった不安要素と, その解決を見ていきます.

====
** バージョンを変数で定義している場合

ScalaおよびJavaのライブラリは, 細かく分割されている中からいくつか使い, それらは同じバージョンを指定して揃えたいということがよくあります. そうすると同じバージョンを何回も書きたくないのでいったん変数(定数)として定義してから使うことになるでしょう. こういう感じに.

>|scala|
val CirceVersion = "0.13.0"
||<

>|scala|
    libraryDependencies ++= Seq(
      "io.circe" %% "circe-core" % CirceVersion,
      "io.circe" %% "circe-generic" % CirceVersion,
      "io.circe" %% "circe-generic-extras" % CirceVersion,
      "io.circe" %% "circe-parser" % CirceVersion,
    ),
||<

このパターン自体は, Renovateのsbt対応が入ったときにサポートされていましたが, <code>scalaVersion</code>の指定で変数を参照している場合は考慮されないという問題がありました.

最初そこで諦めて「<code>scalaVersion</code>を変数に書いてる場合に対応してないからRenovateダメだわ」と, 同僚であり社内でRenovate推し活動をしていたid:ikesyo:detailに愚痴ったところ, なんと彼が対応するpull requestを出してくれました!

[https://github.com/renovatebot/renovate/pull/4205:embed]

** リッチな差分の表示

そこまでしてもらったら使わないのも申し訳ないのでチームでRenovateを使いはじめました. 使ってみてわかったのは, Scalaのライブラリの場合だと, リリースノートの情報やソースコードの差分へのリンクをpull requestに表示してくれないため, バージョンを上げても大丈夫かどうかの判断材料を探しに行くのがかなり手間ということでした.

たしかに, GitHub等から直接インストールするわけではないので, バージョンの情報からソースコードの差分やリリースノートに辿り着くのは難しいよな...... いや待てよ, POMにSCMのURLとかあるからそこから復元すればいけるでしょ. と考えて[https://gist.github.com/tarao/506d8e66fbc7059db156a51eb4e5cbcb:title=ちょちょいとスクリプトを書いてみた]らイケそうでした. ここまできたら, あとはこれをTypeScriptで書いたらいいだけ. ということで自らpull requestを出しました.

[https://github.com/renovatebot/renovate/pull/6756:embed]

こういう感じで出ます.

[f:id:tarao:20201217202324p:image:w600]

あとでscala-stewardのソースコードを見たらだいたい同じようなことをしてそうでした. POMの情報の取得は[https://get-coursier.io/:title=Coursier]に任せていたりしてちょっと<s>ずるい</s>羨しいなという気はしましたが.

** 実際に使ってみてどうか

べんりに使えています. 細かい不満としては以下くらいでしょうか.

- GitHub上にソースコードがある場合でもリリースタグの命名規則が揺れていてうまく差分を取れないものがある
-- <a href="https://github.com/renovatebot/renovate/pull/6871"><code>release-<var>なんとか</var></code>というパターンは対応した</a>けどキリがないので他は諦めた
-- scala-stewardでも同じ問題があるはず?
- ライブラリのサイトのトップページ以外の情報がない場合がある
-- とくにJavaのライブラリ
-- scala-stewardでも同じ問題があるはず
- <code>project/build.properties</code>は更新してくれない気がする
-- プルリクチャンス!!!

あるいはFAQとしては以下がありそうですが、大丈夫そうです.

- Q. <code>project/plugins.sbt</code>も更新してくれるの?
-- A. してくれます
- Q. <code>.sbt</code>を分割してても大丈夫?
-- A. [https://github.com/renovatebot/renovate/blob/7c573e96950616e94a8d2ccf0988f1c478ad9f0f/lib/manager/sbt/index.ts#L6:title=大丈夫そう]です

Scala界隈で激しく使うともっと細かい不満がいろいろ出てくる可能性はありますが, そういう場合はpull requestを出して改善していきましょう!

あと言っておきたいこととしては, みんなライブラリを公開するときはPOMの<code>&lt;scm&gt;</code>とか<code>&lt;url&gt;</code>のところはちゃんと埋めようね, そしてリリースタグは<code>v<var>なんとか</var></code>で打ち[https://semver.org/:title=Semantic Versioning]に従おうね, ということくらいです.
