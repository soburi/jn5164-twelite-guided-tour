パッケージング、配布
--------------------


### パッケージ定義ファイル

作成したパッケージを配布するには、 パッケージを構成するツールを記載した
定義ファイルを作成し、それをweb上で公開する。
パッケージの定義ファイルの仕様については、
[Arduino IDE 1.6.x package_index.json format specification](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.6.x---package_index.json-format-specification)[^1]
にまとめられている。

[^1]: <https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.6.x---package_index.json-format-specification>

Arduinoからは公開したパッケージ定義ファイルのURLを指定し、パッケージをインストールすることができる。

パッケージ定義ファイルはjsonの形式で書かれており、ファイル名は　
*package\_[YOURNAME]\_[PACKAGENAME]\_index.json* とする。


定義ファイルの構造は以下の例のようになる。

```
{
   "packages":
   [
     {
       "name": "JN516x Boards",
        "maintainer": "soburi",
        "websiteURL": "https://github.com/soburi/JN516x-arduino-package",
        "email": "soburi@gmail.com",
        "platforms": [ ... ] ,
        "tools": [ ... ]
     }
   ]
}
```

3rd-partyの定義ファイルは*packages*の配列に複数の要素を入れないことが推奨されている。
パッケージ自体の情報として、名称、メンテナ、URL, e-mailがあって、
構成要素は*platforms*, *tools*の配列に格納される。
 構成は*platforms* が *tools* に依存する形になるので、先に*tools*を定義する。
*tools*の各要素は以下のような構造を持つ。

```
{
  "name": "ba-elf-ba2-gcc",
  "version": "4.1.2-r8352-build0",
  "systems":
   [
     {
       "host": "i686-mingw32",
       "archiveFileName": "ba-elf-ba2-gcc_i686-mingw32_4.1.2-r8352-build0.tar.bz2",
       "url": "https://github.com/soburi/JN516x-arduino-package/blob/repository-0.0.20150723/ba-elf-ba2-gcc_i686-mingw32_4.1.2-r8352-build0.tar.bz2?raw=true",
       "checksum": "SHA-256:15e6c8be6d2d53c5f6d13195bd19b34fe234717a4e014167f7c40b99b104a996",
       "size": "30317284"
     },
    {
      ...
    }
   ]
}
```

*name*要素はツールの格納先のフォルダ名に使われる。
*boards.txt*からは、*{runtime.tools.[TOOLNAME].path}*とすることで、
フォルダ名を取得できる。
*version*は同じツールが複数版あるときの識別子として使われる。

*systems*の各要素はホスト環境ごとの具体的なファイル情報になる。
*host*はArduinoの動作環境を指定する。現行で配布されている環境は
以下の通り。

- Linux 64-bit (x86_64-pc-linux-gnu),
- Linux 32-bit (i686-pc-linux-gnu),
- Windows (i686-mingw32),
- Mac 64-bit(x86_64-apple-darwin)

これらの環境向けのそれぞれの配布ファイルごとに
ファイル情報を指定する。

*url*はツールの取得元URLになる。
*archiveFileName*はダウンロードするファイル名を指定する。
(通常はhttpでダウンロードするときに付与されるものだが、WebServerによっては
付与されない場合があるため)
*size*は文字通りファイルサイズ。
*checksum*はファイルの正当性確認のためのハッシュ値を付与する。
*[ALGORITHM]:[CHECKSUM]*のフォーマットで、指定する。


*platform*の要素の定義は、

```
{
  "name": "JN516x Boards",
  "architecture": "jn516x",
  "category": "Contributed",
  "version": "0.1.20150429",
  "url": "https://github.com/soburi/JN516x-arduino-package/archive/release-0.1.20150429.tar.gz",
  "archiveFileName": "release-0.1.20150429.tar.gz",
  "checksum": "SHA-256:6c8fe3cdd8684a5604485718f41e252e91401cbc40812d7f3a851e6e8ef6479f",
  "size": "35846",
  "boards": [
    {
       "name": "TOCOS TWE Lite"
    }
  ],
  "toolsDependencies": [
    {
      "packager": "jn516x",
      "name": "ba-elf-ba2-gcc",
      "version": "4.1.2-r8352-build0"
    },
    {
      "packager": "jn516x",
      "name": "jenprog",
      "version": "1.1.0-20150712"
    }
  ]
},
```

*name*, *version*, *url*, *archiveFileName*, *checksum*, *size*については*tools*の要素と
同じ意味である。
*architecture*はプラットフォームのアーキテクチャ名(CPUアーキテクチャ)を指定する。
*category*はBoardsManagerに表示される分類となる。
*boards*もBoardsManagerの表示に使われる。サポートしている具体的なボード名(製品名)
を入れる。
*toolDependencies*はこのplatformのバージョンが依存するツール類を定義する。
これは先ほど定義したtoolsの内容を参照し、同時にインストールするツールを指定することになる。
*packager*は*architecture*と同じものを指定する。(これについては明示的な仕様の記載がない)
*name*と*version*の組でインストールするツールを指定する。


### パッケージのインストール

Arduino以外のパッケージを追加するには、*[環境設定]*から*[Additional Boards Manager URLs]*の
テキストボックスにパッケージ定義のURLを指定する。
複数指定する場合はURLをカンマで区切って並べる。
設定を変更したのち、ArduinoのUIのメニューから _[ツール]->[ボード]->[Boards Manager...]_
を選択し、Boards Managerを立ち上げると、
指定のURLで定義された3rd party製のパッケージが表示され、インストールができる。

本稿で作成している JN516x 向けのPlatform Packageは以下のURLを指定することでインストールできる。
<http://soburi.github.io/JN516x-arduino-package/package_soburi_jn516x_index.json>
