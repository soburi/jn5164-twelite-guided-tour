### Serial

Arduinoの頻出機能として、ログ出力に使う*Serial*がある。
Arduinoの機能としては比較的規模の大きいものであるが、
避けて通れないものの一つなので、読み解いていく。


#### クラス階層

*Serial*はC++のクラスの継承を使って実装されているので、
まず、この構成を確認する。

*Serial* のインスタンスは*UARTClass*型のグローバル変数として
*variants/arduino\_due\_x/variant.cpp* で宣言されている。

この *UARTClass* は

_Print < Stream < HardwareSerial < UARTClass_

という継承関係を持っている。 それぞれの階層の役割として、

: Print
    型ごとに適当な表示を行うための一連の *print()*, *println()* 関数群、
    *print()* が利用する最終段の出力関数の *write()* が定義されている。
    この *write()* は 仮想関数となっており、このクラスを継承したクラスで
    実装される。(言い換えれば、このクラスを継承したクラスで *print()* を動作
    させるには、 *write()* のみ実装すればよい)

: Stream
    *read()*, *available()*, *peek()*, *flush()* のinputに関する機能を定義している。
    これらはいずれも仮想関数で、ハードの実装にあわせて継承先で実装する。

: HardwareSerial
    *Serial*としての *begin()*, *end()* を定義している。演算子*bool()* の定義もやっているが、
    実質的に使っていない。

: UARTClass
    実装クラス。継承元で定義されている仮想関数の実装と割り込みの処理を行っている

という役割になっている。それぞれの階層で定義している仮想関数をUARTのハード実装にあわせて
*UARTClass*で実装する。

#### UARTClass の改造

SAM向けの*UARTClass*は当然SAMのハード構成に依存したものになっているが、
バッファリングの処理が多いので、実際にハードを触る箇所は多くない。
必要なところをJN516x向けに置き換えていく。

#### 初期化処理

 *UARTClass::init()*で初期化処理を実装する。
UARTのパラメータ設定はそこそこ数があって、以下の関数を呼んで初期化を行う。

: bAHI\_UartEnable()
    UARTを有効化する。送受信のバッファを指定する。
: vAHI\_UartSetRTSCTS()
    RTS,CTSの有効無効を設定する。ArduinoとしてはRTS,CTSは使っていないので、無効にする。
: vAHI\_UartSetBaudRate()
    標準的なBaudRate値を設定する。
: vAHI\_UartSetControl()
    シリアルポートの設定、パリティ、データ長、ストップビットを設定する。
: vAHI\_UartSetInterrupt()
    割り込み発生の契機を設定する。データ受信時と送信FIFO空の時の割り込みを設定する。
: vAHI\_Uart0RegisterCallback()
    UART0 の割り込みハンドラを設定

ArduinoのAPIとして初期化パラメータが渡されるのは、BaudRate、パリティ、データ長、ストップビットの
４項目の設定についてである。 *vAHI_UartSetBaudRate()*, *vAHI_UartSetControl()* にパラメータを
マッピングして初期化を行う。

#### 送信処理

*Serial.write()*で送信処理を行っている。
割り込み処理中でなければ、1バイトの送信を行い、
割り込み処理中はバッファリングする。

1バイトの送信は *vAHI_UartWriteData()* で行う。
送信処理中の判定は *u8AHI_UartReadInterruptStatus()* の
戻り値の *E\_AHI\_UART\_INT\_TX* ビットで判別できる。

JN516xの機能としては、*u16AHI_UartBlockWriteData()* でDMAを使った効率的な
転送ができるが、母体からの変更が大きくなるので、割愛する。

#### 受信処理

*UARTClass*の実装では、データ受信の割り込み発生時に受信バッファにデータを格納する。
実際のデータの読み出しは*Serial.read()*の要求で行う。
データ受信の割り込みはのパラメータの *E\_AHI\_UART\_INT\_RXDATA* ビットを設定して通知される。

割り込みハンドラは  *variant/arduino\_due\_x/variant.cpp* で実装されているので、
*variant/twelite/variant.cpp* で実装する。

```
void UART0_Handler(uint32_t u32DeviceId, uint32_t u32ItemBitmap)
{
  Serial.IrqHandler(u32ItemBitmap);
}
```

ハンドラの処理本体は *UARTClass::IrqHandler()* で実装している。

```
void UARTClass::IrqHandler(const uint32_t status)
{
  // Did we receive data?
  if ((status & E_AHI_UART_INT_RXDATA) == E_AHI_UART_INT_RXDATA)
    _rx_buffer->store_char(u8AHI_UartReadData(_dwId));

  ...
}
```



### coresのいろいろ

coresにある、ハードウェア依存の実装が必要なものを以下に示す。

: UARTClass
    UART(Serial)の機能実装

: WInterrupts
    digital I/O の割り込み処理 *attachInterrupt()* を実装
: wiring
    時計機能のdelay(), millis(), micros(). delayMicroseconds() を実装
: wiring\_analog
    アナログ入出力機能(ADC, PWM) analogRead(), analogWrite() を実装
: wiring\_digital
    デジタル入出力 digitalRead(), digitalWrite(), pinMode()を実装
: wiring\_pulse
    パルス検出 pulseIn()を実装
