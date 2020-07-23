# LowWeightCipher(Serpenters)

パイコネ変換：https://ja.wikipedia.org/wiki/%E3%83%91%E3%82%A4%E3%81%93%E3%81%AD%E5%A4%89%E6%8F%9B


20200723

置換パターンを秘密鍵のハッシュ値をシードに暗号学的に安全な疑似乱数を使った乱数で２つの置換パターンを生成し、それを動的置換パターンとして置き換えることにしました。
BBS疑似乱数は遅いし、でかい数を使わないと出来ないので、符号かPKPを使った疑似乱数生成器を自作しないといけなくなりました。
暗号学的に安全な疑似乱数生成器の定義と構成法は、ゴールドライヒのfoundation of Cryptographyに乗っているので調べてみます。
今はscanfを使った体験版ですが、ここ数日中に移行します。

時間が経つに連れ、どんどん脆弱性が明らかになっていく感じがしますｗ。
置換パターンを生成するプログラム(p.c)を公開しました。

ほとんど寝てなくて開発できない。
サブキーを入れるタイミングとサブキーの生成処理を変更した。
生成方法はかなりいい加減です。
なので本当にこれでいいのかは謎。
次回までに調べておきます。

すみません、訂正です。
パイコネじゃありません。
もし平文をカオスで暗号化するなら、カオスはストリーム暗号にしか使えないはずです。
なぜならカオスは非可逆的だからです。
暗号化に直接カオスを利用するような方法があるかも知れないですが、今のところは使ってないです。
カオスを使っているとしたら、マスターキーからサブキーを生成するときに、６４ビットブロックの鍵をシフトしながら自分自身にXORしているところでしょうか？
自分自身にXORすることをパイコネと読んでいるつもりでした。
しかしハッシュ関数にパイコネ変換が適用できても、ブロック暗号にこの方法が直接適用できるわけではなさそうです。

今の所、動的に平文を入れ替えながらSboxで換字するという方法は復号に成功してます。
しかし、置換パターンを動的にするのはSboxの動的生成と比べて本質的ではないかも知れない。
この方法は後日、攻撃方法を適用するに当たって弱点にしかならないことが解るかも知れないです。

解読できる可能性は、置換パターンが固定して公開してあることから想像がつきます。

XOR処理は入力と出力の差分が分ってしまえばサブキーの生成方法が分っているので、秘密鍵を特定することができるはずです。
一つの改良案としては、公開されている置換パターンを、秘密鍵に応じて動的に生成するようなやり方を試すことです。
攻撃を不可能にするためには何が必要かを考えなければなりません。


20200722

間違いに気づいたけど、難なくクリア。
まさか今日中に解決するとは思わなかった。
何も調べないで作った割には良く出来てると思う。
乱数をシーケンシャルに使うと駄目なことが判明。
ということは昨日考えていたようにサブキーがカオスというのは使えないことに。
ということはこの暗号の特徴はこのトランプのシャッフルみたいに動的置換をやっても復号できるというのが特徴なのかも。
同期を取った32バイトの乱数を使って、パッチを当てるようにサブキーとして使う分にはうまく行く。
ちゃんとラウンド内でローテーションできてるし、強いて言えば遅いのが欠点。
サブキーをシフトとかやって処理を軽くすることもできるけど、ひねりが足りないのでつまらない。
どんな落とし穴があるかわからないので、まだ気をつけなければならないが、サブキーを対して考えもせずに導入できたのは嬉しい。
でもただサブキーが使えるようになっただけじゃ楽しくないので、共用体を活用してスピードアップも計ってみた。
測定すると、sage-9.0のソースで11秒。
まあまあの性能。

よくある経験だけどデータが0とかで偶然うまく行っているような場合も有って、今日はそれにハマった。
どこまで正しいか解らなくなった。
全部表示させて確認とか。
部分計算を独立させて動かすと何の問題もないのに、処理している間にデータが消えて全部ゼロになるというバグなんて想像してなかった。
まあ、実装ミスだろうということでバグ取り計画に記録。

全然設計に時間をかけていないので、どの程度駄目なのかは未確定だ。
このままだとAESの128ビット暗号のようにはならず、SHACAL-2 -NESSIEのように256ビットブロックサイズの暗号になりそうだ。
アイデア通りに処理ブロックを32バイトで固定して、そのすべてのバイト全体を繰り返し置換している。
一つのブロックが10回位シャッフルされるのに対して、置換群の位数は1700以上あるのでそんなに簡単には周期が出てこない。
一つのブロックを処理している間は確実に異なる置き換えがなされるはずだ。（間違っているかも知れない）

でも大丈夫だと思うのは、1ビットでも計算が合わないと、出てくるデータがぐちゃぐちゃになっていることからすぐ解る。


出来上がったと思ったらとんでもない勘違いをしていたが、漸くサブキー付きの簡単なSPNのイメージになった。
back2theEggの頃からか違いしていて、ラウンド数を更新するときに必要な関数を一つ飛ばしていたので完全な間違いをしていたことが発覚。
道理で調子が良すぎると思った。
自分で気づくのはなかなか難しいけど、気づかないよりはマシだ。
置換というのは非可換軍なので、一回暗号化してしまうと逆回転させないといけない。
そこで復号キーとして、トータルで何回ラウンドしたかがわかれば逆回転して復号できると思った。
で、ちょっとしたカウンタなんかをつけてみたり。

何だかもう一夜にして出来上がってしまった単純なSPN構造。
でも自分のやりたいことじゃなくて、かなり妥協している。
サブキーは乱数じゃうまく行かないので、単に鍵の配列を入れ替えているだけだし、どうして乱数にするとうまく複合できないのかがわからない。
同じ操作を繰り返しているだけだから、本来は復号できるはずなのだ。
それにラウンド内で配列操作すると計算量が増えるので遅くなる。
これにはなんとか乱数化したような処理を付け足す予定。
つまり同じものが再生できてないということなのだろうが、まだそこまでは確認していない。
とにかく暫定版というべきものが出来たはず。
換字と置換しかしてない。

20200721

処理単位を64バイトから32バイトに変更しました。
速度は余り変わりがありません。

ハッシュの延長でやっていた秘密鍵暗号を独立したリポジトリに移しました。
今後はこちらで開発を勧めます。
ハッシュ関数と秘密鍵暗号を並列で取り組むのでどちらかに偏るかも知れません。
本当なら計算コストが低い（処理が軽い）暗号を作りたいところですが、安全性を考えるとどうしても処理が増えてしまい、簡単な処理ではうまく行きません。
簡単な処理で強い暗号を作るというのはなかなか難しいものです。

微妙に実装が変わっています。
ほとんど変わってませんが、サブキーの生成がより安全になった感じです。

今後1,2週間をかけて、Feistel型とSPN型の可変式実装を行う予定です。
そのためにはきちんと設計し直す必要が有ります。
今のままでも自分のイメージ通りには動いていますが、それを押し通すよりマナーに従ったほうが得だという算段です。

実装完了後、差分攻撃と線形解読法の実装の準備を始めます。
攻撃方法を理解する方が難しいでしょう。
攻撃法の実装には約1か月の時間がかかると思います。

現状がどうなっているのかよく把握できてないので、漠然としたイメージしかないのですが内部がランダム置換とSboxを使っていて、特に左右を分けることなくバイト単位で混ぜ合わせているという感じです。
バッファサイズが2048バイトに対して、処理ブロックが64バイト、そのうち32バイトをF関数に入れるFeistel構造と、64バイト全体に置換をかけるSPN構造の二つの方法を同時に実装します。
今のままだと処理単位が512ビットと多すぎるので、32バイト（あるいは16バイト）に修正する可能性があります。
それはAESの256ビット鍵がどのように鍵より小さなブロックサイズに使われているか調べることでわかるでしょう。
秘密鍵のサイズは256ビット固定です。
これ以上小さな秘密鍵は使わないと思うので。

サブキーのサイズが32バイトとして128ビットブロック暗号にするためには、サブキーの半分だけ使用することで対応できます。
また、サブキーというのは逆算できないといけないものなのか、それとも復元できない方がいいのかはまだ調べてません。
もし復元できないとダメな場合は、サブキー生成にカオスは使えないことになります。
パイコネっていうだけで、カオス決定なのでその点は間違いないでしょう。
今のままだと、単に秘密鍵から乱数を生成して、それをサブキーにしている感じなので、ブロック暗号というよりストリーム暗号になってる感じです。
安全性はまだ不確定ですが、実質ストリーム暗号っぽいですね。
そうなるともうストリーム暗号としては遅すぎるのでなんとかブロック暗号の体裁を意識して作り直すつもりです。

カオスを作るような置換は漸化式であらわすことが出来ます。
置換群には古い歴史と、奥深さがある領域ですが、日本で置換群を専門にやっている人はまずいないといっていいでしょう。
研究者の層の厚さが英語文化圏と日本の差です。
日本ではやっていないような研究も海外ではありますが、日本でしか行われていない研究はまずないはずです。
で、その漸化式ですが、単なる置換ではいいカオスは作れません。
二つの置換群x,yを用意し、xの逆元をx^-1であらわします。
この時、

y_(i+1)=xy_(i)x^-1

という漸化式であらわすことのできる置換ｙだけが乱数を作ることが出来ます。
なぜこの場合だといい乱数が作れるのかはまだ分かっていません。（未解決問題）
