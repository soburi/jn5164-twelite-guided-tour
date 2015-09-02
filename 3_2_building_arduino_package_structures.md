Arduino coreの実装
---------------------

Arduino coreはArduinoの基本API実装である。

### 開発用のフォルダ

Arduinoのインストール先ディレクトリの *hardware/[提供元]/[プラットフォーム名]*
のフォルダを作成することで、そのディレクトリがプラットフォームの検索対象となる。
本稿ではこのディレクトリを使う設定で作業する。
後述のBoardsManagerからインストールする場合は、ユーザーディレクトリの配下に格納される。
Platform Packageとしてパッケージングする場合も、このディレクトリの構成を
tar.gzとして固めて配布すればよい。
Platform Packageとは異なる設定となる箇所はディレクトリに別途*platform.local.txt*を作成し、
設定をオーバーライドする。

### 構成要素

基本となるAVRのcoreはarduinoのディレクトリの *hardware/arduino/avr* にある。

別のアーキテクチャ向けにcoreを実装する場合は、この構成を踏襲して実装することになる。


: bootloaders
    文字通りbootloaerのコード。本稿では扱わない
: cores
    API実装のソースコードを格納する
: firmwares
    ISP用のファームウェア。本稿では扱わない
: libraries
    coreに含まれない標準ライブラリや、ボード専用の拡張ライブラリを格納する
: variants
    coreのアーキテクチャに複数のボードの実装がある場合、ボード依存部分のソースコードが置かれる。
: platform.txt
    コンパイル時に実行するコマンドなど、platform依存の動作を定義するためのファイル。
: boards.txt
    同じプラットフォームを使う複数のボードがある場合に、その差分を定義する。


  今回のJN516x向けの実装ではbootloaders, firmwaresは不要なので作成しない。

既存のSAMコア(Arduino Due)のソースコードを~~パクッて~~forkして、JN516x向けに改造して、
cores部分を実装する。その作業の中で、必要に応じてボード依存部分はvariantsに切り分けていく。

librariesは基本APIに依存するものもあるので、coresの部分が出来上がってからの
作業になる。 本稿ではArduino 1.6.4 の SAMコア向けのソースコードをベースに実装していく。



platform.txt, boards.txt
-------------------------
具体的なAPIを実装するのに先立って、パッケージの設定ファイルの
platform.txtとboards.txtを作成する。
この2つのファイルでコンパイル時に実行するコマンドを決めている。

これらは[Arduino IDE 1.5 3rd party Hardware specification](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-3rd-party-Hardware-specification) [^7]
で仕様が定義されている。
[^7]: <https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-3rd-party-Hardware-specification>
platform.txtの主な役割はコンパイルを行うコマンドを定義することだ。

まず、識別名として、 *name*と*version*を設定する。
*name*はArduinoのIDEのメニューから見える名前になる。
*version*は現状使用されていない。

設定の中核になるのは、*recipe.*で始まるパラメータの設定である。
これらがコンパイルやリンクが行われるときのコマンド定義となるからだ。


列挙すると以下のようなコマンドの定義がある。

: recipe.c.o.pattern
    .cをコンパイルするコマンド
: recipe.cpp.o.pattern
    .cppをコンパイルするコマンド
: recipe.S.o.pattern
    .Sをアセンブルするコマンド
: recipe.ar.pattern
    .oからスタティックライブラリを生成するコマンド
: recipe.c.combine.pattern
    elf形式にリンクするコマンド
: recipe.objcopy.bin.pattern
    elfをバイナリに変換するコマンド

platform.txtで定義している変数は、ほとんどがこれらのコマンドに収斂する。

コンパイラはPlatform Packageの定義で構成が変わる。
Platform Packageとしてインストールされたツールのパスは、
*{runtime.tools.[TOOLNAME].path}* で参照することができる。

それぞれのコマンドの設定は、チップベンダー提供のサンプルなどを参照して設定する。
Arduino自身は特にこの辺のコマンドに対する制約などは持っていない。
それぞれのボードに対して適当なバイナリが生成できればよい。



コンパイルのコマンド以外には、ファイルのアップロードツールなどのコマンドも
ここで定義している。アップロードに使うツールであれば、
*tools.[TOOLNAME].upload.pattern*
のパラメータでコマンドラインを指定する。

boards.txtはボードの種別ごとに差し替える必要のあるパラメータを定義する。
boards.txtで新規のボードを定義するには
*[BOARDNAME].name* のパラメータを定義する。ここで*BOARDNAME*は任意の
識別文字列である。

*[BOARDNAME].build.*のパラメータはコンパイル時にコマンドラインからdefine値
として渡される値となる。これは、platform.txtからは*build.…*と、*BOARDNAME*
を抜いた名前で参照できる。必要に応じて任意のパラメータを追加することもできる。
*[BOARDNAME].upload.tool* でアップロードツールを指定できる。
ここで指定できる値は、Platform Packageに含まれるツールの識別子のみとなる。
ツールはplatform.txtから*tools.…*で参照されるので、こちらの方とも
設定を整合する必要がある。

この辺りの仕様については、Arduinoのドキュメントに詳細に記載されており、
不明瞭な点は少ない。わからない点があれば冒頭に示した仕様を確認すれば
特に問題なく設定できるはずだ。
