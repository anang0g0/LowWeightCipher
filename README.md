# LowWeightCipher(Serpenters)

パイコネ変換：https://ja.wikipedia.org/wiki/%E3%83%91%E3%82%A4%E3%81%93%E3%81%AD%E5%A4%89%E6%8F%9B

20200721

ハッシュの延長でやっていた秘密鍵暗号を独立したリポジトリに移しました。
今後はこちらで開発を勧めます。
ハッシュ関数と秘密鍵暗号を並列で取り組むのでどちらかに偏るかも知れません。
本当なら計算コストが低い（処理が軽い）暗号を作りたいところですが、安全性を考えるとどうしても処理が増えてしまい、簡単な処理ではうまく行きません。
簡単な処理で強い暗号を作るというのはなかなか難しいものです。

微妙に実装が変わっています。
ほとんど変わってませんが、サブキーの生成がより安全になった感じです。

今後1,2週間をかけて、Feistel型とSPN型の可変式実装（二つで一つ）を行う予定です。
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