Peripherals API
----------------

Peripherals API は JN516xの比較的多彩なハード機能を使うための
APIである。いくつかの主要な機能について、例を交えて説明する。


### Digital I/O

最も基本的なハード制御機能で、ピンを入出力端子として使用する。
ピンから出力される電圧のHIGH/LOWを制御する。
また、ピンに入力されている電圧がHIGHかLOWかを検知する。

JN516xのDIO設定関数は、2つの引数をそれぞれ、
Onにするピン、Offにするピンのビット列として受け付ける。
共通ルールとして、いずれのビット列も0であるピンについては、
**現状維持**の動作を行う。
共に1が立っているピンについては、それぞれの関数の仕様にある
優先動作に従う。
*vAHI_DioSetDirection()* で入出力の設定、
*vAHI_DioSetPullup()* で、内蔵プルアップ抵抗のOn/Offを設定する。

```
/* 0～3番ピンを出力に、4～7番ピンを入力にする。 */
vAHI_DioSetDirection(0xF0, 0xF);

/* 0,1番ピンをプルアップ、2,3番ピンのプルアップを解除 */
vAHI_DioSetPullup(0x3, 0xB);
```


*vAHI_DioSetOutput()* と *u32AHI_DioReadInput* はシンプルな
書き込み、読み出しを行う。

```
/* 0,1番ピンをHIGH、2番ピンをLOWにする。3番ピンは[現在のまま] */
vAHI_DioSetOutput(0x3, 0x4);

/* 4番ピンを読み取る */
bool pin4isHigh = ( ( u32AHI_DioReadInput() & 0x10 ) >> 4) ? true : false;
```

入力ピンとして使用する場合は、信号の立ち上がり**もしくは**立ち下がり
で割り込み処理を行うこともできる。両方検知するためには、2つのピンを使って
立ち上がりと立下りを検知する必要がある。

```
/* 4,5番ピンの割り込みを有効にする */
vAHI_DioInterruptEnable(0x10, 0x0);

/* 4番ピンの立ち上がり時、5番ピンの立下り時に割り込みを発生させる */
vAHI_DioInterruptEdge(0x10, 0x20);
```

ピンへの入力を契機にスリープ状態からの復帰を行える。
使い方は割り込み検知の時と関数名以外は同じ。

```
/* 6番ピンの変化でスリープから復帰する */
vAHI_DioIWakeEnable(0x70, 0x0);

/* 6番ピンの立ち上がり時にスリープから復帰する */
vAHI_DioWakeEdge(0x70, 0x00);
```


#### references

##### [JN-UG-3087: JN516x Integrated Peripherals API User 5. Digital Inputs/Outputs (DIOs)](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G9.1004281)[^1]
##### [JN-UG-3087: JN516x Integrated Peripherals API User 21.  DIO and DO Functions](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G26.1020675)[^2]

[^1]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G9.1004281>
[^2]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G26.1020675>

### Tick Timer

Tick Timerはクロックを元にした高精度のハードウェアタイマを提供する。
typicalな使用方法として、システム起動からの経過時間の取得に利用できる。
タイマ満了時のコールバックでタイマ満了回数をカウントして、
タイマ周期を乗算することでシステム起動時からの経過時間を知ることが
できる。また、スピンループによるウェイト処理もこれを利用して実現できる。

最大周期を設定してタイマーを開始する。

```
	/* 明示的に停止してカウンタを0に設定 */
	vAHI_TickTimerConfigure(E_AHI_TICK_TIMER_DISABLE);
	vAHI_TickTimerWrite(0);
	/* タイマ周期の最大値(約16秒)をセット */
	vAHI_TickTimerInterval(0x0fffffff);
	/* コールバック通知を設定 */
	vAHI_TickTimerIntEnable(true);
	vAHI_TickTimerRegisterCallback(ticktimer_callback);
	/* タイマの繰り返し実行を開始 */
	vAHI_TickTimerConfigure(E_AHI_TICK_TIMER_RESTART);
```

現在のタイマー値とタイマ満了回数から経過時間が計算できるので、
タイマ満了時のコールバック *ticktimer_callback()*でタイマ満了回数をカウントする。
*micros()*はシステムの動作クロックを係数として起動からの経過時間を算出する。

```
#define clock2usec(a) ( (a) / clockCyclesPerMicrosecond() )
#define MICROSECONDS_PER_TICKTIMER_OVERFLOW (clock2usec(0x0FFFFFFF))
uint32_t ticktimer_overflow_count = 0;

void ticktimer_callback(uint32 u32Device, uint32 u32ItemBitmap)
{
	ticktimer_overflow_count++;
}

uint32_t micros() {
	unsigned long m;

	m = MICROSECONDS_PER_TICKTIMER_OVERFLOW * ticktimer_overflow_count +
		clock2usec(u32AHI_TickTimerRead());
	return m;
}
```

#### references

##### [JN-UG-3087: JN516x Integrated Peripherals API User 5. Digital Inputs/Outputs (DIOs)](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G9.1004281)[^1]
##### [JN-UG-3087: JN516x Integrated Peripherals API User 21.  DIO and DO Functions](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G26.1020675)[^2]

[^1]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G9.1004281>
[^2]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G26.1020675>


### Sleep

*vAHI_Sleep()* の呼び出しで、省電力モードに移行する。
メモリのOn/Off, 32kHzオシレータのOn/Offの組み合わせ、もしくはDeepSleepが設定できる。
スリープ状態からはI/Oの駆動か、Wake Timerによってのみ復帰可能となるたが、
Wake Timerは32kHzのクロックで動作しているため、Wake Timerで自律的に復帰する場合には
32KHzオシレータはOnにする必要がある。
32kHzオシレータを停止した状態では、Digital I/Oの割り込みを使って復帰する必要がある。

メモリOn状態でのスリープ状態からの復帰は *AppWarmStart()*, DeepSleepを含むメモリOff状態からの
復帰では *AppColdStart()* が呼ばれる。

復帰状態の詳細は *u16AHI_PowerStatus()* で判別することができる。

```
void AppColdStart()
{
	/* Wake Timerの満了をチェック */
	uint8_t which_timer_expired = u8AHI_WakeTimerFiredStatus();

	/* Peripheral API を初期化 */
	uint32_t api_version = vAHI_Init();

	/* 6番ピンの変化でSleepから復帰する */
	vAHI_DioIWakeEnable(0x70, 0x0);
	/* 6番ピンの立ち上がり時にSleepから復帰する */
	vAHI_DioWakeEdge(0x70, 0x00);

	/* do somethings ... */

	/* オシレータOFF, RAM保持でsleep (あまりやらない設定?) */
	vAHI_Sleep(E_AHI_SLEEP_OSCOFF_RAMON);
}

void AppWarmStart()
{
	/* どのピンの入力でWakeUpしたのかを判定 */
	uint32_t whats_happen = u32AHI_DioWakeStatus();

	/* Peripheral API を初期化 */
	uint32_t api_version = vAHI_Init();

	/* WakeTimerを有効化 */
	vAHI_WakeTimerEnable(E_AHI_WAKE_TIMER_0, true);

	/* do somethings ... */

	/* Wake Timerをセット. 一定時間後に復帰  (60sec * clock_per_sec) */
	vAHI_WakeTimerStartLarge(E_AHI_WAKE_TIMER_0, ( 60 * 32000.f);
	/* オシレータON, RAM保持なしでsleep */
	vAHI_Sleep(E_AHI_SLEEP_OSCON_RAMOFF);
}
```


#### references

##### [JN-UG-3087: JN516x Integrated Peripherals API User 19. System Controller Functions vAHI_Sleep)](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G24.1011260)[^3]
##### [JN-UG-3087: JN516x Integrated Peripherals API User 19. System Controller Functions u16AHI_PowerStatus)](http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G24.1029145)[^4]
[^3]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G24.1011260>
[^4]: <http://www.nxp.com/documents/user_manual/JN-UG-3087.pdf#G24.1029146>
