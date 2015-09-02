あとがき
--------

東京コスモス電機さんのTWE-Liteは、機能の充実したファームウェアが同梱されていて、
基本、あまりモジュール自体は触らずに無線化アダプタとして簡単に使えるようなモノに
なっています。

が、これをArduinoに繋げて使うと、「何でそこに32bitのマイコンが大した仕事しないで
遊んでるのに8bitのマイコンに金出さにゃならんのだ！」という微妙な状況が発生します。
TWE-Liteの開発環境は、当然ながら組み込み開発の文脈を背負ったもので、これで
モノ作るのはArduinoのぬる湯に浸かった後ではかなりダルイ感じになります。
であればと、という流れで本稿の「ArduinoでTWE-Liteを使えるようにする」という、
「ラーメンを作るために小麦を植える」「楽するための面倒は厭わない」正しい
プログラマ思想に基づいた開発を行う次第となりました。
モノとしては、ちょうどESP8266が安くてArduinoからそのまま使えるモジュールとして
安定供給されだしたので、かなり間の悪い感じはしますが、
あちらにはない省電力っぷりとか、まだ使えるところはあるかと思います。
もともと802.15.4はインターネット語であるTCP/IPと別の思想で動いているので、
この辺が楽に動かせるとIoTのTの側の世界観を遊べるんじゃないかと。
Arduinoなんでそんな難しいこともできないんで、F\*\*k'n IoTなモノを作るネタに
出来ると良いかと思っております。


__Best wish for your happy hacking!__