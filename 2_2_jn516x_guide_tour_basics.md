基本的な処理の流れ
------------------

JN516x のプログラムの基本的な流れとして、
エントリーポイント、初期化処理、割り込みハンドラの使い方を理解する。


### エントリーポイント

JN516xにはプログラムのエントリーポイントが2種類存在する。
スリープ機能を使用したときの復帰で使い分けが行われる。

##### AppColdStart()

  電源投入時は *AppColdStart()* の呼び出しが行われる。
  RAMの保持を行わないスリープ状態からの復帰においても *AppColdStart()* が呼ばれる。
  いずれの状態からの起動かはAPIから判別することができる。

##### AppWarmStart()

  スリープ後、RAM保持状態からの復帰では *AppWarmStart()* が呼ばれる。


省電力機能を使用しない場合、*AppWarmStart()* は
*AppWarmStart()* の単純なラッパーとして実装し、
実処理は *AppColdStart()* に実装すればよい。

#### references

##### JN-AN-1003
Boot Loader Operation
##### JN-UG-3024
IEEE802.15.4 Stack User Guide

### 初期化処理

Peripheral類の使用を開始する前に *vAHI_Init()* を呼び出して、初期化を行う必要がある。
この処理は起動毎、復帰時にそれぞれ必要である。
ドキュメントの記載としては、上記のとおり、起動時、復帰時のAPI呼び出し前に実行する記載と
なっているが、復帰時のWake Timerの経過時間取得など、この規定に従えない処理も存在する。

```
void AppColdStart() {
	/* Peripheral API を初期化 */
	uint32_t api_version = vAHI_Init();
	...
```

#### references

##### [JN-UG-3087 JN516x Integrated Peripherals API User Guide 18. General Functions](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G23.1010636)[^1]
[^1]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G23.1010636>

### 割り込みハンドラ

JN516xの割り込み処理は
*void (\*callback)(uint32\_t device\_id, uint32\_t item\_bitmap)*
の型を持つのコールバック関数を割り込みハンドラとして登録することで行う。
*device\_id* は登録する割り込み種別毎に渡される値で、同一の関数で複数の割り込み種別を処理していなければ
無視して構わない。*item\_bitmap* には割り込みの発生要因が設定されて通知される。

コールバックの登録については、*vAHI_[機能名]RegisterCallback* の関数名で統一されている。
これらの関数で動作させるそれぞれの機能について登録処理を行う。
機能によって、割り込み通知の有効化の指定が必要となる。
多くのものは、機能有効化の *...Enable()* のような関数のパラメータとして割り込み通知の有効化を
指定するが、*vAHI\_TickTimerIntEnable()* のように独立した関数で指定するものもあり、
統一されていない。それぞれのAPI仕様を参照のこと。

```
	/* システム制御機能のコールバック通知を受け取る */
	vAHI_SysCtrlRegisterCallback(sysctrl_callback);
```

#### references

##### [JN-UG-3087 JN516x Integrated Peripherals API User Guide A](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G40.1006067)[^2]

[^2]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G40.1006067>



### イベントループとQueue API

必要に応じて、イベントループを作成する。
JN516xではこの目的でQueue APIが用意されているので、これを利用してもよい。
この場合はQueue APIの内部で割り込みハンドラの登録が行われるので、
割り込みハンドラの登録処理を行わない。

```
void AppColdStart() {
	uint32_t api_version = vAHI_Init();
	/* Queueの初期化 */
	uint32 success = u32AppQApiInit(NULL, NULL, NULL);

	/* Event loop */
	while(true) {
		AppQApiHwInd_s *psAHI_Ind = psAppQApiReadHwInd();
		if(psAHI_Ind) {
			/* イベント処理 */
		}
	}
}
```

#### references

##### [JN-AN-3024 IEEE 802.15.4 Stack User Guide A.Application Queue API](http://www.nxp.com/documents/user_manual/JN-UG-3024.pdf#G15.1006178)[^3]
[^3]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G15.1006178>

### DBG Functions

デバッグ用に便利な*printf*形式のUART出力機能が*dbg.h*に定義されている。
*AppColdStart()* ないし、*AppWarmStart()* で以下のように初期化することで
UARTにメッセージを出力できる。

```
#ifdef DBG_ENABLE
	DBG_vUartInit(E_AHI_UART_0, E_AHI_UART_RATE_9600);
#endif
	for(int i=0; i<1000; i++) ; /* 安定化のための待ち... */
	DBG_vPrintf(TRUE, "DBG Printf enabled");
```

#### references

##### [JN-UG-3075 JenOS User Guide 6. Debug (DBG) Module](http://www.nxp.com/documents/user_manual/JN-UG-3075.pdf#G10.1004281)[^4]
[^4]: <http://www.nxp.com/documents/user_manual/JN-UG-3075.pdf#G10.1004281>
