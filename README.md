SphereURG
======================
SphereURGはコマンド式サーボモータとレーザー式距離センサを用いた回転式3次元距離センサです     
SphereURGは複数のRTC群によって構成されているセンサです   
<img src="http://farm9.staticflickr.com/8209/8274513348_44e0532a69_b.jpg" width="400px" />
<img src="http://farm9.staticflickr.com/8220/8273446855_cc911277e7_b.jpg" width="400px" />

使用するコンポーネントは以下のとおりです  

* [UrgManager](https://github.com/HiroakiMatsuda/UrgManager)  
2次元距離センサRTC。北陽電機（株）のレーザースキャナであるURGシリーズと通信します  

* [RsServoManager](https://github.com/HiroakiMatsuda/RsServoManager)  
コマンド式サーボモータRTC。双葉電子工業（株）のRSシリーズを制御します  

* SphereURGController  
サーボモータと距離センサに指令を送るRTC。各RTCの計測や移動タイミングを調整します  
また、外部RTCからのコマンドでセンサの測域範囲を変更することもできます    

* SphereURG  
サーボモータと距離センサのデータを統合して出力します  
このRTCからの出力が回転式3次元距離センサSphereURGの出力になります  

* SphereURGViewer  
SphereURGから出力されるデータを可視化するビューアーRTCです  

* [MultiTypeConsolIn][consolein]  
SphereURGControllerに指令を送るRTCとして使用しています  
これはTimedLongSeq型のデータ列を送れるものならどのようなRTCでも構いません  

[urg]: https://github.com/HiroakiMatsuda/UrgManager　
[rs]: http://www.futaba.co.jp/robot/command_type_servos/index.html  
[consolein]: https://github.com/HiroakiMatsuda/MultiTypeConsoleIn  



本RTC群を用いたデモンストレーションをYoutubeでご覧になれます  
[回転式3次元距離センサ SphereURG][video]  
[video]: http://www.youtube.com/watch?v=QIoGwwRsPKI   


動作確認環境
------
Python:  
2.6.6  

OS:  
Windows 7 64bit / 32bit  
Ubuntu 10.04 LTS / 12.04 LTS 32bit  

対応RTC:
RsServoManager

ファイル構成
------  
SphereURGController  
│―SphereURGController.py  
│―ini   
│　　│―sucontrol.ini    
│  
│―rtc.conf  

* SphereURGController.py  
RTC本体です  
* sucontrol.ini  
使用するサーボのIDや、移動角度、移動時間などを設定します    
* rtc.conf  
ポートの設定や動作周期を設定できます  
  
SphereUrg  
│―SphereUrg.py    
│  
│―rtc.conf  

* SphereURG.py  
RTC本体です  
* rtc.conf  
ポートの設定や動作周期を設定できます  

SphereURGViewer  
│―SphereURGViewer.py    
│―suviewer.py  
│―ini   
│　　│―view.ini  
│      
│―rtc.conf  

* SphereURGViewer.py  
RTC本体です  
* suviewer.py  
matplotlibを用いてデータを3次元プロットするモジュールです  
* rtc.conf  
ポートの設定や動作周期を設定できます  

注:本RTCにおいてユーザーが操作すると想定しているファイルのみ説明しています  
またその他のRTCについては、それぞれのDLページのREADMEを御覧ください  

ファイルの設定はiniファイルを通して行えるので、簡単に設定を変えられます  
iniファイルの設定はInitialize時に読み込むので、設定を変更した場合はRTCを再起動してください
SphereURGControllerはURGやサーボモータへ指令を送る都合でDeactivateへの遷移指令を出しても即座には反応しません  
RTSystemEditorからエラーが出る場合がありますが、しばらく待てば正常に状態遷移します  

RTCの構成
------  
<img src="http://farm9.staticflickr.com/8503/8272097561_27d3d37c05_c.jpg" width="500px" />    
RsServoManagerなどの対応するRTCを接続することでGUIを通していサーボモータを制御できます  

SphereURGController   

* urg port :OutPort  
データ型; TimedLongSeq  
[Command]
0:計測終了  
コマンド例　[0]
1:計測開始
コマンド例　[1]  

* motion port :OutPort  
データ型; TimedLongSeq  
[Flag, Id, Position, Time]  
 ・  `Flag` :  モーションの同期用フラグ
0:同期なし
移動指令は全て同期なしで指令  
 ・  `Id` :  サーボID  
指令を送るサーボモータのIDを指定します      
 ・  `Position` :  移動位置  
サーボモータへ角度 [0.1 deg]を指定します     
 ・  `Time` :  移動時間  
指定位置までの移動時間 0~16383 [msec]の間で指定します  

* on_off port :OutPort  
データ型; TimedLongSeq  
[Flag, Id, Torque mode]  
 ・  `Flag` :  モーションの同期用フラグ  
0:同期なし  
1:同期あり  
Flagを1にすると設定したサーボモータ全てに同時に指令を送ります  
個別のトルク保持指令ボタンを押した場合は同期なしモード  
全体のトルク保持指令ボタンを押した場合は同期モード   
 ・  `Id` :  サーボID  
指令を送るサーボモータのIDを指定します    
 ・  `Toruque mode` :  トルクモード指定  
0:トルクをオフにします  
1:トルクをオンにします  

* command port :InPort  
データ型; TimedLongSeq  
[Command, CW\_angle, CCW\_angle, Move\_time]  
 ・  `command` :  計測の開始と終了  
2:計測範囲指定    
 ・  `CW\_angle` :  CW（時計回り）方向の計測角度 [0.1 deg]
 ・  `CCW\_angle` :  CCW（反時計回り）方向の計測角度 [0.1deg]  
 ・  `Move\_time` :  1 [deg]移動するのにかかる時間 [ms]  
コマンド例　[2, 300, -300, 30]  
　　　　　　　[2, 300, 200, 30]  

SphereURG   

* data port :OutPort  
データ型; TimedLongSeq  
UrgManager:urg.ini [DATA] intens = OFFのとき  
[Timestamp, Angle, 1, Length\_dist, DIst1, Dist2, ..., DistX]  
UrgManager:urg.ini [DATA] intens = ONのとき  
[Timestamp, Angle, 2, Length\_dist, Length\_intens, DIst1, Dist2, ..., DistX, Intens1, Intens2, ... ,IntensX]  
 ・  `Timestamp` :  URG内部のタイムスタンプ  
アップカウントのタイマで、最大16777216 [ms]までカウント後、0にリセットされます     
 ・  `Angle` :  サーボの現在角度 [0.1deg]      
 ・  `Length_dist` :  距離データの長さ（データの個数）   
 ・  `Length_intens` :  反射強度データの長さ（データの個数）  
 ・  `Dist1~X` : 距離データ [mm]  
 ・  `Intens1~X` : 反射強度データ   

* urg port :InPort  
データ型; TimedLongSeq  
[Command, CW\_angle, CCW\_angle, Move\_time]  
 ・  `command` :  計測の開始と終了  
2:計測範囲指定    
 ・  `CW_angle` :  CW（時計回り）方向の計測角度 [0.1 deg]
 ・  `CCW_angle` :  CCW（反時計回り）方向の計測角度 [0.1deg]  
 ・  `Move_time` :  1 [deg]移動するのにかかる時間 [ms]  
コマンド例　[2, 300, -300, 30]  
　　　　　　　[2, 300, 200, 30]  

* servo port :InPort  
データ型; TimedLongSeq  
[Angle]  
 ・  `Angle` :  サーボモータの現在角度 [0.1deg]  

SphereURGViewer   

* data port :InPort  
UrgManager:urg.ini [DATA] intens = OFFのとき  
[Timestamp, Angle, 1, Length\_dist, DIst1, Dist2, ..., DistX]  
UrgManager:urg.ini [DATA] intens = ONのとき  
[Timestamp, Angle, 2, Length\_dist, Length\_intens, DIst1, Dist2, ..., DistX, Intens1, Intens2, ... ,IntensX]   
 ・  `Timestamp` :  URG内部のタイムスタンプ  
アップカウントのタイマで、最大16777216 [ms]までカウント後、0にリセットされます     
 ・  `Angle` :  サーボの現在角度 [0.1deg]      
 ・  `Length_dist` :  距離データの長さ（データの個数）   
 ・  `Length_intens` :  反射強度データの長さ（データの個数）  
 ・  `Dist1~X` : 距離データ [mm]  
 ・  `Intens1~X` : 反射強度データ   

使い方
------
###1. SphereURGControllerを設定する###
sucontrol.iniをテキストエディタなどで開き編集します  

SERVO  
  ・ ```id = X```  
使用するサーボモータのIDを設定します  

  ・ ```inittial_pos =  X```  
サーボモータの初期値（計測開始位置）[0.1deg]をしていします      
  ・ ```offset =  X```  
サーボモータの角度にオフセット値 [0.1deg]を加えます  
サーボモータとセンサの取付け角度のズレを補正するために使用します  
  ・ ```cw_angle =  X```   
CW（時計回り）方向の計測角度 [0.1 deg]  　 
  ・ ```ccw_angle =  X```    
CCW（反時計回り）方向の計測角度 [0.1deg]    
  ・ ```move_time =  X```  
 1 [deg]移動するのにかかる時間 [ms]  
  ・ ```margin_time =  X```  
サーボモータの移動時間待ち時間にマージンを加えます  
制御対象によってはサーボモータの移動が規定の時間内に完了しない可能性があるためこの値で調整します  


###2. SphereURGViewerを設定する###
view.iniをテキストエディタなどで開き編集します  

VIEW  
  ・ ```x_min = X```  
表示するデータのX軸最小値を設定します    
  ・ ```x_max = X```  
表示するデータのX軸最大値を設定します  
  ・ ```y_min = X```  
表示するデータのY軸最小値を設定します    
  ・ ```y_max = X```  
表示するデータのY軸最大値を設定します  
  ・ ```Z_min = X```  
表示するデータのZ軸最小値を設定します    
  ・ ```Z_max = X```  
表示するデータのZ軸最大値を設定します  
  ・ ```mode =  XXX```  
描画モードを設定します        
HOLD: 過去のプロット点を保持しながらプロットします  
NORMAL: 過去のプロット点は保持せず、描画ごとに更新します  
  ・ ```color = XXX```  
表示するプロット点の色を指定します　　
カラーコードで指定でき黒なら#000000のように指定します        
  ・ ```title = XXX```  
表示するデータにタイトルを付けます  

URG  
この項目は座標変換に使用される値です。使用するURGに合わせて設定してください  
  ・ ```angle_min =  X```  
URGの最小スキャン角度を指定します [deg]  
  ・ ```angle_max =  X```  
URGの最大スキャン角度を指定します [deg]  
  ・ ```step =  X```  
位置スッテップあたりの角度を指定します [deg]  

###2. サーボモータ制御用RTCと接続する###
1. RsMotion.py起動する  
ダブルクリックするなどして起動してください  
2. サーボモータと接続する    
RsServoManagerなどのサーボモータ制御用RTCと接続します  

###3. その他のRTCを設定する###
* [UrgManager](https://github.com/HiroakiMatsuda/UrgManager)      
* [RsServoManager](https://github.com/HiroakiMatsuda/RsServoManager)  
* [MultiTypeConsolIn][consolein]  
以上３つのコンポーネントをリンク先のREADME.mdを参考に設定してください  
これらのRTCの設定に矛盾があると正常に動作しないのでご注意ください  
初期設定はRS405CBとTop-URGを用いて動作するよう設定しています  
  
###4. RTC郡を接続する###
RTCの構成項目にある図を参考に各、RTCを接続します  
このとき、SphereURGViewerのみ状態に関係なく、Initialize時から起動します  
各コンポーネントを接続したら、以下の順でコンポーネントをActivateします  
1. UrgManager
2. RsServoManager  
3. SphereURGController
4. SphereURG  
5. SphereURGViewer  
6. MutliTypeConsoleIn  

同時にActivateしてもエラーが起きないと思いますが、場合によってはエラーが起きる可能性があるため、できる限りハードウェアよりのコンポーネントから起動してください  

この時点でSphereURGViewerに点群が表示されていれば起動に成功しています  
もし表示されない場合は、RTSystemEditorなどでコンポーネントの状態を確認してください  
エラーで各コンポーネントが落ちている場合は、一度Reset状態にしてから再度Activateしてください  
それでもダメな場合は、配線の接続やiniファイルの設定を見なおしてください  

###5. スキャン範囲を変更する###
SphereURGは追跡対象を見つけたときに、その周辺を集中的にスキャンするフォーカススキャンという動作が可能です  
この動作は、SphereURGControllerに接続されているRTCから指令を送ることで実行されます  
本サンプルではMultiTypeConsoleInから手動でコマンドを入力します  

MultiTypeConsoleIn console.iniは以下のように設定されているとします    
DATA  
  ・ ```mode =  console```  
  ・ ```num =  4```  
  ・ ```type =  TimedLongSeq```    

次に、MultiTypeConsoleInにコンソールに[2, 300, -300, 30]と入力します  
これでSphereURGを-300 [0.1deg]から300 [0.1deg]まで30 [ms / deg]で動かせます  
いろいろ値を変えて試して見てください  

<img src="http://farm9.staticflickr.com/8343/8273446817_2327890dd9_b.jpg" width="400px" />
      
以上が本RTCの使い方となります  

ライセンス
----------
Copyright &copy; 2012 Hiroaki Matsuda  
Licensed under the [Apache License, Version 2.0][Apache]  
Distributed under the [MIT License][mit].  
Dual licensed under the [MIT license][MIT] and [GPL license][GPL].  
 
[Apache]: http://www.apache.org/licenses/LICENSE-2.0
[MIT]: http://www.opensource.org/licenses/mit-license.php
[GPL]: http://www.gnu.org/licenses/gpl.html