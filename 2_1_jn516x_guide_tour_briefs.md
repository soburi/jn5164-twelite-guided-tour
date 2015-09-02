JN516x ガイドツアー
===================

ベンダー提供情報
-----------
JN516xの開発環境やマニュアル類は[NXP社のサイト](http://www.arm.com/products/processors/instruction-set-architectures/index.php)[^1]
から参照することができる。
NXP社に買収される前の開発元のJennic社のサポートページ[^2] から古い版数の情報をたどることができる。

[^1]: <http://www.jp.nxp.com/techzones/wireless-connectivity/jn51xx.html>
[^2]: <http://www.jennic.com/support/index.php>

### 開発環境

JN516xの開発環境はコンパイラなどのツールチェーンと、無線プロトコルに
対応したSDKの部分から構成されている。

後にArduinoのマルチプラットフォーム対応を行うため、
本稿では最新版ではなくコンパイラのソースが入手できる版を使用する。
以下の2つのツールをインストールする。

- [JN-SW-4041: SDK-Toolchain](http://www.jennic.com/download_file.php?supportFile=JN-SW-4041-SDK-Toolchain-v1.1.exe)[^3]
コンパイラなどのツールチェーン、焼き込みツールやデバッグ用ツールなどのパッケージ

- [JN-SW-4065: JN516x-JenNet-IP-SDK](http://www.nxp.com/documents/other/JN-SW-4065.zip)[^4]
JenNetIP(6LoWPAN)のプロトコルスタック。実質的にIEEE802.15.4のプロトコルを包含する。


[^3]: <http://www.jennic.com/download_file.php?supportFile=JN-SW-4041-SDK-Toolchain-v1.1.exe>
[^4]: <http://www.nxp.com/documents/other/JN-SW-4065.zip>

### リファレンス文書の構成

特に参照頻度の高い文書として、ハードウェア機能のリファレンスである

- [JN-UG-3087: JN516x Integrated Peripherals API User Guide](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf)[^5]

通信機能のAPIリファレンスとなる

- [JN-UG-3024: IEEE 802.15.4 Stack User Guide](http://www.nxp.com/documents/user_manual/JN-UG-3024.pdf)[^6]

- [JN-UG-3080: JenNet-IP WPAN Stack User Guide](http://www.nxp.com/documents/user_manual/JN-UG-3080.pdf)[^7]

起動処理の詳細、ISPモードの通信プロトコル、バイナリフォーマットを記載した

- [JN-AN-1003: JN51xx Boot Loader Operation](http://www.nxp.com/documents/application_note/JN-AN-1003.pdf)[^8]

が挙げられる。

[^5]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf>
[^6]: <http://www.nxp.com/documents/user_manual/JN-UG-3024.pdf>
[^7]: <http://www.nxp.com/documents/user_manual/JN-UG-3080.pdf>
[^8]: <http://www.nxp.com/documents/application_note/JN-AN-1003.pdf>

### サンプルコードなど

サンプルコードの中ではWireless UARTの実装サンプル

- [JN-AN-1178: IEEE802.15.4 Wireless UART for JN516x](http://www.nxp.com/documents/application_note/JN-AN-1178.zip) [^9]

がプログラムの基本構成、IEEE802.15.4やUARTの使い方について比較的見通しが良いサンプルとなっている。

プロトコルスタック別に雛形のコードも用意されているが、

- [IEEE802.15.4 Application Template for JN516x](http://www.nxp.com/documents/other/JN-AN-1174.zip)[^10]
- [JenNet-IP Application Template](http://www.nxp.com/documents/application_note/JN-AN-1190.zip)[^11]

如何せんソース規模が大きかったり、実動作する部分がなかったりで若干扱いづらい感じがある。

[^9]: <http://www.nxp.com/documents/application_note/JN-AN-1178.zip>
[^10]: <http://www.nxp.com/documents/other/JN-AN-1174.zip>
[^11]: <http://www.nxp.com/documents/application_note/JN-AN-1190.zip>
