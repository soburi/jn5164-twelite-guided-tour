Protocol Stack
--------------

### JN516xで使えるプロトコル

JN516x では IEEE802.15.4, 6LoWPAN/JenNetIP, ZigBeeの3種類の
プロトコル実装がSDKとして提供されている。

: IEEE802.15.4

  IEEE802.15.4 はJN516xで利用できるプロトコルスタックで最もプリミティブなプロトコルスタックである。
  6LoWPANやZigBeeからも下位のデータリンク層として利用されているが、
  単純な通信であれば、直接MAC層を利用してデータ送受信を実装することも可能である。

: 6LoWPAN/JenNetIP

  6LoWPAN は IEEE802.15.4 の上でIPv6の通信を行うための規格である。
  IPv6の(割と大きな)パケットの情報を削って、低電力デバイス向けのネットワークに
  データを流せるようにする。一般のIPv6ネットワークとはゲートウェイを介して接続する。
  JenNetIPはNXPが開発した6LoWPANを拡張したプロトコルである。JN516xのSDKとしては、
  JenNetIPの一部として6LoWPANが含まれる形となる。本稿では扱わない。

: ZigBee

  ZigBee は ZigBee Allianceによって制定されたIEEE802.15.4 の上の通信プロトコル。
  Zigbeeについては商用ライセンス等の制約が多いため、本稿では扱わない。


### IEEE802.15.4 Protocol Stackの要点

IEEE802.15.4のプロトコルスタックを使って、単純なデータ送信を行う方法について概略を示す。
サンプルコードの
[JN-AN-1178: IEEE802.15.4 Wireless UART for JN516x](http://www.nxp.com/documents/application_note/JN-AN-1178.zip)
がわかりやすいので、これを繙いていく。

[^1]: <http://www.nxp.com/documents/application_note/JN-AN-1178.zip>


#### 初期化
まず、 _u32AppQApiInit()_ で、イベントキューを初期化する。

```
    (void)u32AppQApiInit(NULL, NULL, NULL);
```

QueueAPIを使わない場合*u32AppApiInit()*で 直接コールバックを処理しても構わない。
staticな受信バッファのアドレスを返すバッファ取得関数を作成し、
MLMEとMCPSの2種類に対して実処理を行うコールバック関数とバッファ取得関数を指定する。




### 基本操作

QueueAPIを使う場合は受信イベントはキューイングされるので、
メインループの中でdequeueして処理する。
サンプルでは *AN1178\_154\_WUART\_Coord/**Source/**AN1178\_154\_WUART\_Coord.c* と
 *AN1178\_154\_WUART\_EndD/**Source/**AN1178\_154\_WUART\_EndD.c*
 にある*vProcessEventQueues()* で実装している処理がそれである。
dequeueは *psAppQApiReadMcpsInd()*, *psAppQApiReadMlmeInd()* の関数で
行われ、受信したメッセージを取得できる。
それぞれで取得できる構造体の*u8Type*のフィールドで
メッセージ種別、メッセージの詳細は*uParam*フィールドに格納されるので、 振り分けて処理をする。

メインループでの処理は

```
while(true)
{
      MAC_MlmeDcfmInd_s *psMlmeInd = psAppQApiReadMlmeInd();
      switch(ps->MlmeInd->u8Type)
      {
      case MAC_MLME_IND_ASSOCIATE:
      /* somethings to do */
      }
      vAppQApiReturnMlmeIndBuffer(psMlmeInd);
}
```

のように、Queueから取得したメッセージをdispatchするような処理になる。

送信は *vAppApiMlmeRequest()*, *vAppApiMcpsRequest()* の関数でMLME,MCPSの
メッセージの送信を行う。
MLMEは通信制御に関するもの、MCPSは実際のデータのやり取りに使う。
 送信処理の一例として、MLME-SCANでEnergy Scanを実行するコードを示す。
 *MAC\_MlmeReqRsp\_s* 構造体の *uParam* 共用体が要求種別ごとのパラメータとなる。

```
/* Structures used to hold data for MLME request and response */
MAC_MlmeReqRsp_s   sReqRsp;
MAC_MlmeSyncCfm_s  sMlmeSyncCfm;
/* Start energy detect scan */
sReqRsp.u8Type = MAC_MLME_REQ_SCAN;
sReqRsp.u8ParamLength = sizeof(MAC_MlmeReqStart_s);
sReqRsp.uParam.sReqScan.u8ScanType = MAC_MLME_SCAN_TYPE_ENERGY_DETECT;
sReqRsp.uParam.sReqScan.u32ScanChannels = 0x00000800UL;
sReqRsp.uParam.sReqScan.u8ScanDuration = 3;

vAppApiMlmeRequest(&sReqRsp, &sMlmeSyncCfm);
```

IEEE802.15.4 のメッセージ1つの送信は以下のような手順になる。

1. request (送信側が要求を送信を行う)
2. indicate (受信側が要求を受信する)
3. response (受信側が応答を送信する)
4. confirm (送信側が要求に対する応答を受信する)


### 接続

IEEE802.15.4のネットワークには少なくとも一台のCoordinatorが存在し、
これに対して、他のEndDeviceのノードが接続して通信が開始される。
Coordinatorは MLME-STARTを要求し、ネットワーク(PAN=Private Area Network)を開始する。
これに先立って、MLME-SCAN要求でenergy scanを実行して、電波の混雑の少ないチャンネル
を確定する。 *AN1178\_154\_WUART\_Coord/**Source/**AN1178\_154\_WUART\_Coord.c* の
以下の関数でそれぞれの手順を行っている。

1. *vStartEnergyScan()*  (MLME-SCANの要求)
2. *vHandleEnergyScanResponse()* (MLME-SCANの結果を処理)
3. *vStartCoordinator()* (MLME-STARTを要求)



EndDevice側は、MLME-SCAN要求でactive scanを実行し、接続可能なPANを探す。
PANが見つかったら、そこに対してMLME-ASSOCIATEを要求する。

実装は
*AN1178\_154\_WUART\_EndD/**Source/**AN1178\_154\_WUART\_EndD.c* の
それぞれの関数で行っている。

1. *vStartActiveScan()*
2. *vHandleActiveScanResponse()*
3. *vStartAssociate()*

MLME-ASSOCIATE要求はCoordinatorが受理し、ASSOCIATEを許可する場合は、
その応答でEndDeviceにアドレスを割り振って接続が完了する。Coordinator側は
 *vHandleNodeAssociation()*、 EndDevice側は *vHandleAssociateResponse()*
でそれぞれ、MLME-ASSOCIATEの通知、その通知への応答を処理している。


開始のシーケンスを纏めると以下のようになる。

|  Coordinator                         | EndDevice                        |
|---------------------------------|------------------------------|
|  request SCAN(energy)       |                                             |
|  confirm SCAN(energy)       |                                             |
|  request START                     |                                            |
|  confirm START                    |                                             |
|                                                  |  request SCAN(active)    |
|                                                  |  confirm SCAN(active)    |
|                                                  | ← request ASSOCIATE   |
|indicate ASSOCIATE ←       |                                              |
|response ASSOCIATE →     |                                              |
|                                                  |→ confirm ASSOCIATE   |



### データ送信

データ送信はMCPS-DATA要求で行う。
これは単純に、*vAppApiMcpsRequest()* でデータを送信すればよい。
802.15.4ではヘッダ含めて最大で127バイトである。



### 802.15.4についての参考資料

802.15.4のメッセージの内容については
[ラピスセミコンダクタ株式会社](http://www.lapis-semi.com/)[^1]の
[MK72660-01のデータシート](http://www.lapis-semi.com/jp/data/datasheet-file_db/telecom/FJDK72660_01-01.pdf)[^2]が
わかりやすい。802.15.4の仕様に基づいたデータの定義なので、意味自体は他社の実装においても差異は無い。


[^1]: <http://www.lapis-semi.com/>
[^2]: <http://www.lapis-semi.com/jp/data/datasheet-file_db/telecom/FJDK72660_01-01.pdf>
