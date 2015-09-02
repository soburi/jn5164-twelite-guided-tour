機種固有ライブラリの作成
----------------------------

### フォルダ構成

BoardsManagerパッケージの中に含める機種固有ライブラリは
*libraries*配下にフォルダを作成する。

### library.properties

まず、ライブラリの定義ファイルを作成する。

定義ファイルは*library.properties*というファイル名で、ライブラリの最上位のフォルダに置く。

このファイルの仕様は
[Arduino IDE 1.6.x package_index.json format specification](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5:-Library-specification#libraryproperties-file-format)[^1]
で定義されている。 これに従って設定を作成する。
ほとんどが説明用の記述なので、プログラム的な意味があるのは、

- name
    Arduinoのメニューに表示される表示名
- version
    バージョン。BoardsManagerでインストールするときのアップデート要否に使われる。
- architectures
    アーキテクチャを指定する。アーキテクチャはboards.txtで定義される値。
    Arduino の APIのみで構成されていてアーキテクチャ依存のないライブラリは"*"を指定する。

の３つ。ここ以外は大きな影響はないが適切な情報を設定する。

[^1]: <https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5:-Library-specification#libraryproperties-file-format>


### ヘッダファイル

Arduinoのメニューからライブラリをインポートすると、
_そのライブラリに含まれるすべての.hファイルをアルファベット順にインクルード_ する。
わかりやすさからは、ライブラリ名と同名のヘッダファイルが一つ読み込まれるのが良いかと思う。
メインのヘッダ以外は拡張子を.h以外(.incなど)にするなどして、インポート時に読み込まれる
ヘッダファイルを限定するようにするとよい。


### 作例: Powerライブラリ

JN516xのスリープ機能を使って省電力動作を行うPowerライブラリを作成する。

まず、ライブラリのフォルダとして *libraries/Power* を作成する。
その中に、 *libraries/Power/library.properties* の定義ファイルを置く。
中身を一通り埋めると以下の例のようになる。

```
name=JN516x Power Management
version=0.1
author=soburi
maintainer=soburi <soburi@gmail.com>
sentence=Power Management functions for JN516x.
paragraph=The library provide power management function, such as sleep and doze.
url=http://github.com/soburi/JN51xx-arduino-package
architectures=jn516x
```

ライブラリのヘッダ、*libraries/Power/Power.h* を作成する。
今回は単純にスリープモードの指定と、復帰時間を指定するようにする。
Arduinoのライブラリの一般的な実装として、グローバル変数で一つのインスタンスを作成し、
このインスタンスをAPIとして使う。ここでは、この指針に従って、アクセス用のインスタンス *Power*
を宣言する。(この指針は、組み込み開発でメモリのフラグメントを避けるために動的なメモリ確保を
  行わないためのもの)

```
class PowerManager
{
public:
	void sleep(uint32_t sleepmode, uint32_t ms=0);
};

extern PowerManager Power;
```

スリープ機能のメインは *vAHI\_Sleep()* で、この関数を呼び出すことでスリープ状態に移行する。
スリープからの復帰の契機は、Digital I/Oの外部からの割り込みかWakeTimerの満了で復帰する。
WakeTimerで復帰する場合は _vAHI\_Sleep()_を実行する前にWakeTimerを仕掛けておく。
スリープ割り込み処理の入れ違いも避けたいので、先に割り込みを無効化するのが望ましい。
_vAHI\_Sleep()_ の実行から実際にスリープ状態に遷移するには多少のラグがあるので、
実行したら、無限ループでプログラムを停止させる。
また、割り込みとの入れ違いも避けたいので、_vAHI\_Sleep()_の実行前に
割り込みの停止を行う。

必要に応じて、Non-Volatile Memory(NVM)にカウンターを保持することで、タイマを継続して
カウントする実装を行うことができる。
ただしWakeTimerは32kHzのクロックで動作しているので、16MHzのクロックで動いているTickTimerと
比較すると精度が悪くなる。
ここでは、タイマ満了予定時点でのtick数とカウンターをNVMに設定して、
復帰時にタイマ満了していなかったら、タイマの残存時間を戻す方法で実装する。

```
#define MSEC2WTCOUNT(ms)  ( (ms)  * 320000.f )

void PowerManager::sleep(uint32_t sleepmode, uint32_t ms)
{
  if(ms != 0)
  {
    // タイマ満了予想時刻の計算
    uint64_t estimated = clockCyclesPerMicrosecond( millis() + ms) * 1000 );
    uint32_t e_count = estimated / 0x0FFFFFFF;
    uint32_t e_tick = estimated % 0x0FFFFFFF;

    // NVMに値保持
    vAHI_WriteNVData(2, e_count);
    vAHI_WriteNVData(3, e_tick);

    // WakeTimerを起動
    vAHI_WakeTimerEnable(E_AHI_WAKE_TIMER_0, true);
    vAHI_WakeTimerStartLarge(E_AHI_WAKE_TIMER_0, MSEC2WTCOUNT(ms) );
  }
  // 割り込みを発生する機能の停止
  vAHI_TickTimerConfigure(E_AHI_TICK_TIMER_DISABLE);

  // Sleepに遷移
	vAHI_Sleep((teAHI_SleepMode)sleepmode);

  // Sleepの遷移までwaitする
	while(true) ;
}
```

復帰時の初期化処理でタイマ値の復元を行う。
WakeTimerの満了によらない復帰(Digital I/Oの割り込みなど)だった場合は、
予想時刻よりも早く復帰しているので、差を反映してタイマ値を設定する。

```
#define WTCOUNT2TTCOUNT(cnt) ( (cnt) / 320000.f  * 16000000.f)
void init()
{
  uint32_t e_count = vAHI_ReadNVData(2);
  uint32_t e_tick = vAHI_ReadNVData(3) );
  if( (u8AHI_WakeTimerFiredStatus() & E_AHI_WAKE_TIMER_MASK_0) == E_AHI_WAKE_TIMER_MASK_0)
  {
    //タイマが満了していなかったら、その時間を満了予想時間から繰り上げる
    uint64_t wt_remains = u64AHI_WakeTimerReadLarge(E_AHI_WAKE_TIMER_0);
    e_count -= (WTCOUNT2TTCOUNT(wt_remains) / 0x0FFFFFFF);
    e_tick -= (WTCOUNT2TTCOUNT(wt_remains) % 0x0FFFFFFF);
  }
  ticktimer_overflow_count = e_count;
  vAHI_TickTimerWrite( e_tick );
}
```

#### Powerライブラリの設計について

スリープ機能はシステムの起動に深く根差した機能なので、
ライブラリとして外出しできる部分だけでは完結せず、復帰理由の取得や、WakeTimerからの時間の
取得・復元などcoresのinit()の実装に沁み出す部分がある。
このパッケージの設計指針としては、ライブラリを使わない限りは
オリジナルのArduinoとはAPI互換とするようにしており、このライブラリを
使わない通常のArduinoと挙動は変わらないように実装している。
