
はじめに
=====


本書について
-------------

本書では、JN5164(TWE-Lite)で動作するプログラム開発に関するノウハウの紹介と、
開発の実践としてJN516xのArduino対応のためにArduinoのPlatform Packageの作成
方法のガイドを提供する。

JN516xについて
-----------------

NXP JN516xはNXP Semiconductors N.V[^1]  (以下NXP)から販売されている無線機能付きの
OpenRISC[^2]アーキテクチャを採用した32bitマイコンである。
無線機能はIEEE802.15.4に対応し、IoTやセンサーネットワークなどの用途に
適用が可能である。省電力で動作可能でコイン電池での駆動も可能である。

現在、製品シリーズとしては JN5161, JN5164, JN5168, JN5169 がラインナップ
されている。[^3]

[^1]: <http://www.nxp.com/>
[^2]: <http://opencores.org/or1k/Main_Page>
[^3]: <http://www.nxp.com/techzones/wireless-connectivity/jn51xx/jn516x.html>

TWE-Liteについて
-------------------

東京コスモス電機株式会社[^4] (以下TOCOS)から発売されているJN5164のOEM品(?)がTWE-Lite[^5]である。
国内の技適を取得しており、国内で **合法的** に問題なく使うことができる。
無線モジュール形態での提供に加えて、DIP28サイズのブレイクアウトボードに
実装したもの[^6]が、秋葉原などの電子部品店を通して供給されており、ホビーユースとしての
入手性、扱いやすさに優れたパッケージとなっている。
ソフト観点ではJN5164そのものなので、技術情報はJN516xの情報が利用可能である。

[^4]: <http://tocos-wireless.com>
[^5]: <http://tocos-wireless.com/jp/products/TWE-001Lite.html>
[^6]: <http://tocos-wireless.com/jp/products/TWE-Lite-DIP/index.html>

Arduinoについて
-------------------
Arduino[^7] は最近の電子工作やMake界隈でよく使われるAVR マイコン[^8] を搭載した
マイコンボードとその開発環境である。
C言語を簡易化したような言語で、プログラミング初心者でも簡単にデジタルな
モノづくりができる。上級者でも組み込み開発特有の面倒な手続きがいらないので、
プロトタイピングツールとしてもよく使われる。
最近では、オリジナルのAVRマイコン以外にもARM[^9]アーキテクチャのものも発売されて、
マイコンボードそのものというよりも、IDEを中心としたエコシステムという
色彩を強めている。最新版のIDEでは、サードパーティ製のボードを使うための
BoardsManagerの機能が整備され、ESP8266[^10] などの新しいアーキテクチャをArduinoに
対応させる動きも活発である。

[^7]: <https://www.arduino.cc/>
[^8]: <http://www.atmel.com/products/microcontrollers/avr/>
[^9]: <http://www.arm.com/products/processors/instruction-set-architectures/index.php>
[^10]: <http://espressif.com/en/products/esp8266/>

本稿で使用する環境、機材
--------------------------
本稿では、開発ターゲットのマイコンとしてJN5164(TWE-Lite DIP) を使用し、
そのソフトウェア開発環境にはNXPから提供されているJN516xの開発環境を用いる。
Arduino向けのパッケージを作成することから、Arduino IDE 1.6.4版を使用する。
開発環境はWindows上に構築する。


リソースなど
----------------

本書で扱ったリソース類は、[jn5164-twelite-guided-tour-resources](https://github.com/soburi/jn5164-twelite-guided-tour-resources)[^11]
に格納している。

また、JN516x向けのArduino Platform Packageは[JN516x-arduino-package](https://github.com/soburi/JN516x-arduino-package)[^12]
にある。

[^11]: <https://github.com/soburi/jn5164-twelite-guided-tour-resources>
[^12]: <https://github.com/soburi/JN516x-arduino-package>
