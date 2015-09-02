### Examples > 01. Basic > blink

Arduinoに同梱されているサンプルプログラムの中で一番基本的なプログラムのblink、
__いわゆるLチカ__
である。まずは、これが動くように最低限の構成を作っていく。
ソースの実行部分は以下のようになっている。

```
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(1000);
  digitalWrite(13, LOW);
  delay(1000);
}
```

呼び出さなくてはならない関数は*setup()* と *loop()* の2つ、
呼び出されてちゃんと動かなければならない関数は
*pinMode()*, *digitalWrite()*, *delay()*の3つである。
これらを実装していく。

#### 起動まわり

Arduinoのエントリーポイントは *setup()* と *loop()* の2か所がある。
この２つの関数は *cores/arduino/main.cpp* で実装されている *main()* (!) の中で
呼ばれている。

基本形としては、新規に *cores/arduino/start.cpp* を起こして、
ストレートに

```
extern "C" void AppColdStart()
{
    main();
}
```

で良い。システムとしての初期化処理は、後述の *init()* で処理されるが、
言語レベルの初期化やデバッグ向けの初期化などは、ここに置く。

#### 初期化処理

 *main()* で一番初めに呼び出される関数 *init()* は *variant/arduino\_due\_x/variant.cpp* で実装されている。
*variant* 配下の構成はボード固有であるが、ここは、元の構成からあまり変更ないよう、
*variant/twelite/variant.cpp* で実装する。

*init()* では、主にペリフェラル回りの初期化処理を行う。前章で触れたように、
*vAHI_Init()* の呼び出し、割り込みハンドラ等の初期化、TickTimerの起動を
行う。

#### pinMode(), digitalWrite()

*pinMode()*, *digitalWrite()* はDigital I/O の機能に対応する。
これらの関数は *cores/arduino/wiring_digital.c* に実装されている。

ピンの入力・出力の方向を決める関数なので、ほぼ*vAHI_DioSetDirection()*と同等の機能だが、
パラメータ種別にに *INPUT_PULLUP* があってプルアップ抵抗の有効無効も一緒に指定されるので、
*vAHI_DioSetPullup()* でプルアップ抵抗の指定も一緒に行う。

```
void pinMode(uint32_t pin, uint32_t mode)
{
	if (mode == INPUT) {
		vAHI_DioSetDirection((1UL<<pin), 0);
		vAHI_DioSetPullup(0, (1UL<<pin));
	} else if (mode == INPUT_PULLUP) {
		vAHI_DioSetDirection(0, (1UL<<pin));
		vAHI_DioSetPullup((1UL<<pin), 0 );

	} else {
		vAHI_DioSetDirection(0, (1UL<<pin) );
	}
}
```

*digitalWrite()* はもっと単純で、*vAHI_DioSetOuptut()*にマッピングされる。

```
void digitalWrite(uint32_t pin, uint32_t val)
{
	if (val == LOW) {
		vAHI_DioSetOutput(0, (1UL<<pin));
	} else {
		vAHI_DioSetOutput((1UL<<pin), 0);
	}
}
```

#### delay(), millis(), micros()

点滅の時間待ち合わせ処理の *delay()* を実装する。
Arduinoの *delay()* はスピンループでの待ち合わせを行っている。
TickTimerで現在のTick数がわかるので、これで起動からの
経過時間取得の *millis()*, *micros()* を実装して、
その時間で待ち合わせのスピンループを回せばよい。
*micros()* は前章記載の実装を使い、*delay()* は

```
void delay(unsigned long ms)
{
	uint32_t start = (uint32_t)micros();

	while (ms > 0) {
		yield();
		if (((uint32_t)micros() - start) >= 1000) {
			ms--;
			start += 1000;
		}
	}
}
```

のような感じで。
