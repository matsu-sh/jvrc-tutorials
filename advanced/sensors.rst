センサの接続
============


ここではJVRCモデルに搭載された各種センサのデータを取得できるようにします。

プロジェクトを開く
------------------

「メニュー」の「プロジェクトの読み込み」から JVRC モデルファイル用のプロジェク
トファイルを読み込みます。プロジェクトファイル名は「モデルファイルのインス
トール」でダウンロードしたリポジトリの「model/robot/samples/sample1.cnoid」です。


JVRC モデルのセンサ
-------------------

JVRC モデルに搭載されているセンサは、テキストエディタでリポジトリの「model/robot/main.wrl」ファイルを開くと確認することができます。
モデルファイルを開くと次のように記述されており、加速度センサ gsensor とジャイロセンサ gyrometer が搭載されていることがわかります。 ::

   DEF JVRC Humanoid {
     humanoidBody [
       DEF PELVIS Joint {
         jointType "free"
         translation 0 0 0.854
         children [
           DEF PELVIS_S Segment {
   	  mass 10
   	  centerOfMass -0.01 0 0.034
   	  momentsOfInertia [0.08958333333333333 0 0 0 0.08958333333333333 0 0 0 0.11249999999999999]
   	  children [
               DEF gsensor AccelerationSensor {
                 sensorId 0
               }
               DEF gyrometer Gyro {
                 sensorId 0
               }
   	    Inline { url "pelvis.wrl" }
   	  ]
   	}
   
他にも、力センサ rfsensor, lfsensorが搭載されていることが分かります。 ::

   				DEF rfsensor ForceSensor {
   				  sensorId 0
   				}

   				DEF lfsensor ForceSensor {
   				  sensorId 1
   				}

カメラ rcamera, lcamera、距離センサ ranger も確認することができます。 ::

   			    DEF NECK_P Joint {
   			      jointType "rotate"
   			      jointAxis "Y"
   			      jointId 17
   			      ulimit [1.0471975511965976] #+60
   			      llimit [-0.8726646259971648] #-50
   			      uvlimit [ 5.75958]
   			      lvlimit [-5.75958]
   			      rotorInertia 0.0596
   			      children [
   			        DEF NECK_P_S Segment {
   				  mass 2
   				  centerOfMass 0.01 0 0.11
   				  momentsOfInertia [0.00968 0 0 0 0.00968 0 0 0 0.00968]
   				  children [
   				    Inline { url "head.wrl" }
   				  ]
   				}
   				DEF rcamera VisionSensor {
   				  translation 0.1 -0.03 0.09
   				  rotation      0.4472 -0.4472 -0.7746 1.8235
   				  frontClipDistance 0.05
   				  width 640
   				  height 480
   				  type "COLOR"
   				  sensorId 0
   				  fieldOfView 1.0
   				} 
   				DEF lcamera VisionSensor {
   				  translation 0.1 0.03 0.09
   				  rotation      0.4472 -0.4472 -0.7746 1.8235
   				  frontClipDistance 0.05
   				  width 640
   				  height 480
   				  type "COLOR"
   				  sensorId 1
   				  fieldOfView 1.0
   				} 
   				DEF ranger RangeSensor {
   				  translation 0.1 0.0 0.0
   				  rotation      0.4472 -0.4472 -0.7746 1.8235
   				  sensorId 0
   				  scanAngle 1.5707963267948966
   				  scanStep 0.011344640137963142
   				  scanRate 100
   				  minDistance 0.1
   				  maxDistance 30.0
   				} 
   			      ]
   			    } # NECK_P

各センサの仕様について解説します。

加速度センサの値はTimedDoubleSeq型になります。

ジャイロセンサの値はTimedDoubleSeq型になります。

力センサの値はTimedDoubleSeq型になります。

カメラの値はTimedLongSeq型になります。width x heightの各ピクセルの色情報が1ピクセル当たり4バイト(long型)としてTimedLongSeqのdata部分に格納されます。
今回のカメラの場合、width = 640, height = 480と定義されているので、640x480のデータとなります。

距離センサの値はTimedDoubleSeq型になります。
シーケンスに計測方向に向かって右からスキャンした距離データが格納されています。
距離の値は何かに干渉が発生する限り出力されますが、干渉がない場合は0になります。


コントローラのソースコード
--------------------------

コントローラのヘッダのソースコードは以下になります。Choreonoidに含まれるサンプルのSR1WalkControllerRTC.hを基にしています。 ::

   /**
      Sample Robot motion controller for the JVRC robot model.
      This program was ported from the "SR1WalkControllerRTC.h" sample of Choreonoid.
   */
   
   #ifndef RobotSensorsControllerRTC_H
   #define RobotSensorsControllerRTC_H
   
   #include <rtm/idl/BasicDataTypeSkel.h>
   #include <rtm/Manager.h>
   #include <rtm/DataFlowComponentBase.h>
   #include <rtm/CorbaPort.h>
   #include <rtm/DataInPort.h>
   #include <rtm/DataOutPort.h>
   #include <cnoid/MultiValueSeq>
   
   class RobotSensorsControllerRTC : public RTC::DataFlowComponentBase
   {
   public:
       RobotSensorsControllerRTC(RTC::Manager* manager);
       ~RobotSensorsControllerRTC();
   
       virtual RTC::ReturnCode_t onInitialize();
       virtual RTC::ReturnCode_t onActivated(RTC::UniqueId ec_id);
       virtual RTC::ReturnCode_t onDeactivated(RTC::UniqueId ec_id);
       virtual RTC::ReturnCode_t onExecute(RTC::UniqueId ec_id);
   
   protected:
       // DataInPort declaration
       RTC::TimedDoubleSeq m_angle;
       RTC::InPort<RTC::TimedDoubleSeq> m_angleIn;
       RTC::TimedDoubleSeq m_gsensor;
       RTC::InPort<RTC::TimedDoubleSeq> m_gsensorIn;
       RTC::TimedDoubleSeq m_gyrometer;
       RTC::InPort<RTC::TimedDoubleSeq> m_gyrometerIn;
   };
   
   extern "C"
   {
       DLL_EXPORT void RobotSensorsControllerRTCInit(RTC::Manager* manager);
   };
   
   #endif

コントローラのソースコードは以下になります。Choreonoidに含まれるサンプルのSR1WalkControllerRTC.cppを基にしています。 ::

   /**
      Sample Robot motion controller for the JVRC robot model.
      This program was ported from the "SR1WalkControllerRTC.cpp" sample of
      Choreonoid.
   */
   
   #include "RobotSensorsControllerRTC.h"
   #include <cnoid/BodyMotion>
   #include <cnoid/ExecutablePath>
   #include <cnoid/FileUtil>
   #include <iostream>
   
   using namespace std;
   using namespace cnoid;
   
   namespace {
   
   const char* samplepd_spec[] =
   {
       "implementation_id", "RobotSensorsControllerRTC",
       "type_name",         "RobotSensorsControllerRTC",
       "description",       "Robot Controller component",
       "version",           "0.1",
       "vendor",            "AIST",
       "category",          "Generic",
       "activity_type",     "DataFlowComponent",
       "max_instance",      "10",
       "language",          "C++",
       "lang_type",         "compile",
       ""
   };
   }
   
   
   RobotSensorsControllerRTC::RobotSensorsControllerRTC(RTC::Manager* manager)
       : RTC::DataFlowComponentBase(manager),
         m_angleIn("q", m_angle),
         m_gsensorIn("gsensor", m_gsensor),
         m_gyrometerIn("gyrometer", m_gyrometer)
   {
   
   }
   
   RobotSensorsControllerRTC::~RobotSensorsControllerRTC()
   {
   
   }
   
   
   RTC::ReturnCode_t RobotSensorsControllerRTC::onInitialize()
   {
       // Set InPort buffers
       addInPort("q", m_angleIn);
       addInPort("gsensor", m_gsensorIn);
       addInPort("gyrometer", m_gyrometerIn);
   
       return RTC::RTC_OK;
   }
   
   RTC::ReturnCode_t RobotSensorsControllerRTC::onActivated(RTC::UniqueId ec_id)
   {
       return RTC::RTC_OK;
   }
   
   
   RTC::ReturnCode_t RobotSensorsControllerRTC::onDeactivated(RTC::UniqueId ec_id)
   {
       return RTC::RTC_OK;
   }
   
   RTC::ReturnCode_t RobotSensorsControllerRTC::onExecute(RTC::UniqueId ec_id)
   {
       if(m_angleIn.isNew()){
           m_angleIn.read();
       }
   
       for(size_t i=0; i < m_angle.data.length(); ++i){
               cout << "m_angle.data[" << i << "] is " << m_angle.data[i] << std::endl;
       }
   
       if(m_gsensorIn.isNew()){
           m_gsensorIn.read();
       }
   
       for(size_t i=0; i < m_gsensor.data.length(); ++i){
               cout << "m_gsensorIn.data[" << i << "] is " << m_gsensor.data[i] << std::endl;
       }
   
       if(m_gyrometerIn.isNew()){
           m_gyrometerIn.read();
       }
   
       for(size_t i=0; i < m_gyrometer.data.length(); ++i){
               cout << "m_gyrometerIn.data[" << i << "] is " << m_gyrometer.data[i] << std::endl;
       }
   
       return RTC::RTC_OK;
   }
   
   
   extern "C"
   {
       DLL_EXPORT void RobotSensorsControllerRTCInit(RTC::Manager* manager)
       {
           coil::Properties profile(samplepd_spec);
           manager->registerFactory(profile,
                                    RTC::Create<RobotSensorsControllerRTC>,
                                    RTC::Delete<RobotSensorsControllerRTC>);
       }
   };


それぞれで行っている処理については、「RTコンポーネントのコントローラの接続」とほとんど同じで、センサが増えただけです。

これらのソースコードは「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/RobotSensorsControllerRTC.cpp」と「model/robot/RTC/RobotSensorsControllerRTC.h」に保存されています。

コントローラのビルド
--------------------

「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/」ディレクトリに移動し、makeコマンドを実行します。

「model/robot/RTC/」ディレクトリに「RobotControllerSensorsRTC.so」というファイルが作成されるはずです。

その後、次のコマンドを実行します。 ::

   sudo make install DESTDIR=/usr

コントローラの設定
------------------

アイテムビューで「BodyRTC」を選択し、プロパティビューの「コントローラのモジュール名」を「RobotSensorsControllerRTC」とします。これは「コントローラのビルド」で作成したモジュールのパスと対応しています。


シミュレーションを実行する
--------------------------

シミュレーションツールバーの「シミュレーション開始ボタン」を押します。
シミュレーションを実行するとchoreonoidを実行している端末にセンサの値が出力されています。
「RTコンポーネントのコントローラの接続」のときとは違い、関節角度(m_angle.data)だけではなく、 加速度センサ(m_gsensor)、ジャイロセンサ(m_gyrometer)の値が表示されるはずです。

.. image:: images/output2.png


サンプルプロジェクトについて
----------------------------

このサンプルのプロジェクトファイルは「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/samples/sample3.cnoid」に保存されています。

.. toctree::
   :maxdepth: 2

