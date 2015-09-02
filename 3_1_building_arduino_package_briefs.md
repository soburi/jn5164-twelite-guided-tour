Arduino Platform Packageの作成
===========================================

Arduino Boards Manager
--------------------------

Arduinoのversion1.6.2からBoardManagerの機能が追加された。
これは、Arduinoや、その派生ボード、互換ボードの開発環境を
Platform Packageとしてネットワーク経由でインストールできる仕組みである。
1.6.4からは、サードパーティー製の開発環境をインストールするための
設定も実装されており、「野良Arduino」もパッケージの提供さえあれば、
arduinoの開発環境から使うことができる。
Arduino自身も[Arduino Due](https://www.arduino.cc/en/Main/arduinoBoardDue) [^1]や
[Arduino Zero](https://www.arduino.cc/en/Main/ArduinoBoardZero) [^2]などのAVR以外のアーキテクチャに
ついてはBoards Managerで配布を行っている。

[^1]:  <https://www.arduino.cc/en/Main/arduinoBoardDue>
[^2]:  <https://www.arduino.cc/en/Main/ArduinoBoardZero>


Platform Package の構成要素
-------------------------------------------

Platform PackageはArduino上でターゲットのボード用のプログラムを
ビルドするのに必要な要素をすべて含む。すなわち、

- コンパイラ
- アップロードツール
- Arduino core のソースコード

が含まれる。これらはパッケージ配布の定義ファイルでそれぞれダウンロードする
アーカイブファイルのURLを指定する。

: コンパイラ
    JN516x向けのコンパイラとして、JN-SW-4041に含まれるgcc相当のソースコードが
		[TOCOSのwebサイト](http://tocos-wireless.com/jp/tech/Linux/ba-jn5148-toolchain-src.zip) [^1]
		から取得できる。これをビルドして格納する。
		(多少古い版のgccだけど我慢。)

[^3]: <http://tocos-wireless.com/jp/tech/Linux/ba-jn5148-toolchain-src.zip>


: アップロードツール
    TOCOSから、アップロードツールとして[TOCOS版jenprog](http://tocos-wireless.com/jp/tech/misc/jenprog/) [^2]
    が配布されているが、リセット処理の作りが雑でArduinoの動作とマッチしない。
    本稿ではオリジナルの[pscholl版jenprog](https://github.com/pscholl/jenprog) [^3]
    を[改造したもの](https://github.com/soburi/jenprog) [^4]使用する。
    FT232を使用した、リセット端子がプログラムに閉じて処理できていないので、
    Arduino環境に組み込んだ時にもともとの動作仕様を維持できないため。

[^4]: <http://tocos-wireless.com/jp/tech/misc/jenprog/>
[^5]: <https://github.com/pscholl/jenprog>
[^6]: <https://github.com/soburi/jenprog>

: Arduino core
     Arduino の API実装となるArduino coreは本稿でのメインテーマである。
     次のセクションで具体的な実装方法を取り上げる。

### その他

Arduino coreがSDKに依存するが、ライセンスの問題でこれを含めることができない。
ユーザーが別途インストールしたものを参照する形式とする。
