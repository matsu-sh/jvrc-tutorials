RTコンポーネントにおけるトルク指令の入力
========================================


ここでは、トルク指令を出力するようにRTコンポーネントを拡張し、ロボットが直立を
維持できるようにします。

プロジェクトを開く
------------------

「メニュー」の「プロジェクトの読み込み」から JVRC モデルファイル用のプロジェク
トファイルを読み込みます。プロジェクトファイル名は「モデルファイルのインス
トール」でダウンロードしたリポジトリの「model/robot/samples/sample2.cnoid」です。

コントローラのソースコード
--------------------------

コントローラのヘッダのソースコードは以下になります。Choreonoidに含まれるサンプルのSR1WalkControllerRTC.hを基にしています。 ::

   /**
      Sample Robot motion controller for the JVRC robot model.
      This program was ported from the "RobotTorqueController.h" sample of Choreonoid.
   */
   
   #ifndef RobotTorqueControllerRTC_H
   #define RobotTorqueControllerRTC_H
   
   #include <rtm/idl/BasicDataTypeSkel.h>
   #include <rtm/Manager.h>
   #include <rtm/DataFlowComponentBase.h>
   #include <rtm/CorbaPort.h>
   #include <rtm/DataInPort.h>
   #include <rtm/DataOutPort.h>
   #include <cnoid/MultiValueSeq>
   #include <vector>
   
   class RobotTorqueControllerRTC : public RTC::DataFlowComponentBase
   {
   public:
       RobotTorqueControllerRTC(RTC::Manager* manager);
       ~RobotTorqueControllerRTC();
   
       virtual RTC::ReturnCode_t onInitialize();
       virtual RTC::ReturnCode_t onActivated(RTC::UniqueId ec_id);
       virtual RTC::ReturnCode_t onDeactivated(RTC::UniqueId ec_id);
       virtual RTC::ReturnCode_t onExecute(RTC::UniqueId ec_id);
   
   protected:
       // DataInPort declaration
       RTC::TimedDoubleSeq m_angle;
       RTC::InPort<RTC::TimedDoubleSeq> m_angleIn;
   
       // DataOutPort declaration
       RTC::TimedDoubleSeq m_torque;
       RTC::OutPort<RTC::TimedDoubleSeq> m_torqueOut;
   
   private:
       cnoid::MultiValueSeqPtr qseq;
       std::vector<double> q0;
       cnoid::MultiValueSeq::Frame oldFrame;
       int currentFrame;
       double timeStep_;
   };
   
   extern "C"
   {
       DLL_EXPORT void RobotTorqueControllerRTCInit(RTC::Manager* manager);
   };
   
   #endif

今回はトルクの出力をしなければならないので、出力ポートのための設定が増加しています。
`RTC::OutPort<RTC::TimedDoubleSeq>` はRTCの出力ポートを表す型であり、出力ポートを操作するにはこれを利用します。

コントローラのソースコードは以下になります。Choreonoidに含まれるサンプルのSR1WalkControllerRTC.cppを基にしています。 ::

   /**
      Sample Robot motion controller for the JVRC robot model.
      This program was ported from the "SR1WalkControllerRTC.cpp" sample of
      Choreonoid.
   */
   
   #include "RobotTorqueControllerRTC.h"
   #include <cnoid/BodyMotion>
   #include <cnoid/ExecutablePath>
   #include <cnoid/FileUtil>
   
   using namespace std;
   using namespace cnoid;
   
   namespace {
   
   static const double pgain[] = {
       8000.0, 8000.0, 8000.0, 8000.0, 8000.0, 8000.0,
       3000.0, 3000.0, 3000.0, 3000.0, 3000.0, 3000.0,
       8000.0, 8000.0, 8000.0,
       8000.0, 8000.0, 8000.0,
       3000.0, 3000.0, 3000.0, 3000.0, 3000.0,
       3000.0, 3000.0, 3000.0, 3000.0, 3000.0,
       8000.0, 8000.0, 8000.0,
       3000.0, 3000.0, 3000.0, 3000.0, 3000.0,
       3000.0, 3000.0, 3000.0, 3000.0, 3000.0,
       };
   
   static const double dgain[] = {
       100.0, 100.0, 100.0, 100.0, 100.0, 100.0,
       100.0, 100.0, 100.0, 100.0, 100.0, 100.0,
       100.0, 100.0, 100.0,
       100.0, 100.0, 100.0,
       100.0, 100.0, 100.0, 100.0, 100.0,
       100.0, 100.0, 100.0, 100.0, 100.0,
       100.0, 100.0, 100.0,
       100.0, 100.0, 100.0, 100.0, 100.0,
       100.0, 100.0, 100.0, 100.0, 100.0,
       };
   
   
   const char* samplepd_spec[] =
   {
       "implementation_id", "RobotTorqueControllerRTC",
       "type_name",         "RobotTorqueControllerRTC",
       "description",       "Robot TorqueController component",
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
   
   
   RobotTorqueControllerRTC::RobotTorqueControllerRTC(RTC::Manager* manager)
       : RTC::DataFlowComponentBase(manager),
         m_angleIn("q", m_angle),
         m_torqueOut("u", m_torque)
   {
   }
   
   RobotTorqueControllerRTC::~RobotTorqueControllerRTC()
   {
   
   }
   
   
   RTC::ReturnCode_t RobotTorqueControllerRTC::onInitialize()
   {
       // Set InPort buffers
       addInPort("q", m_angleIn);
   
       // Set OutPort buffer
       addOutPort("u", m_torqueOut);
   
       return RTC::RTC_OK;
   }
   
   RTC::ReturnCode_t RobotTorqueControllerRTC::onActivated(RTC::UniqueId ec_id)
   {
       if(!qseq){
           string filename = getNativePathString(
               boost::filesystem::path(shareDirectory())
               / "motion" / "RobotPattern.yaml");
   
           BodyMotion motion;
   
           if(!motion.loadStandardYAMLformat(filename)){
               cout << motion.seqMessage() << endl;
               return RTC::RTC_ERROR;
           }
           qseq = motion.jointPosSeq();
           if(qseq->numFrames() == 0){
               cout << "Empty motion data." << endl;
               return RTC::RTC_ERROR;
           }
           q0.resize(qseq->numParts());
           timeStep_ = qseq->getTimeStep();
       }
   
       m_torque.data.length(qseq->numParts());
   
       if(m_angleIn.isNew()){
           m_angleIn.read();
       }
       for(int i=0; i < qseq->numParts(); ++i){
           q0[i] = m_angle.data[i];
       }
       oldFrame = qseq->frame(0);
       currentFrame = 0;
   
       return RTC::RTC_OK;
   }
   
   
   RTC::ReturnCode_t RobotTorqueControllerRTC::onDeactivated(RTC::UniqueId ec_id)
   {
       return RTC::RTC_OK;
   }
   
   RTC::ReturnCode_t RobotTorqueControllerRTC::onExecute(RTC::UniqueId ec_id)
   {
       if(currentFrame > qseq->numFrames()){
               return RTC::RTC_OK;
       }
   
       if(m_angleIn.isNew()){
               m_angleIn.read();
       }
   
       MultiValueSeq::Frame frame = qseq->frame(currentFrame++);
   
       for(int i=0; i < frame.size(); i++){
               double q_ref = frame[i];
               double q = m_angle.data[i];
               double dq_ref = (q_ref - oldFrame[i]) / timeStep_;
               double dq = (q - q0[i]) / timeStep_;
               m_torque.data[i] = (q_ref - q) * pgain[i] + (dq_ref - dq) * dgain[i];
               q0[i] = q;
   
               cout << "q_ref = " << frame[i] << " ";
               cout << "q = " << q << " ";
               cout << "dq_ref = " << dq_ref << " ";
               cout << "dq = " << dq << " ";
               cout << "torque = " << m_torque.data[i] << endl;
       }
       oldFrame = frame;
   
       m_torqueOut.write();
   
       return RTC::RTC_OK;
   }
   
   
   extern "C"
   {
       DLL_EXPORT void RobotTorqueControllerRTCInit(RTC::Manager* manager)
       {
           coil::Properties profile(samplepd_spec);
           manager->registerFactory(profile,
                                    RTC::Create<RobotTorqueControllerRTC>,
                                    RTC::Delete<RobotTorqueControllerRTC>);
       }
   };

これらのソースコードは 「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/RobotTorqueControllerRTC.cpp」と 「model/robot/RTC/RobotTorqueControllerRTC.h」に保存されています。

コントローラの設定
------------------

アイテムビューで「BodyRTC」を選択し、プロパティビューの「コントローラのモジュール名」を「RobotTorqueControllerRTC」とします。これは「コントローラのビルド」で作成したモジュールのパスと対応しています。
さらに、プロパティビューの「自動ポート接続」を true にします。

.. image:: images/property_torque.png

ポーズ列の追加
--------------

まずアイテムビューで「JVRC」を選択します。
次に、「メニュー」の「ファイル」「新規」より「ポーズ列」を選択し「SampleMotion」という名前で追加します。

.. image:: images/motion.png

次に、「表示」の「ビューの表示」から「ポーズロール」を選択します。次の画面が表示されるはずです。

.. image:: images/pose_role.png

基準の姿勢を作るため、アイテムビューで「JVRC」を選択し、ツールバーにある「選択ボディを初期姿勢に」のボタンを押します。

.. image:: images/pose_toolbar.png

ポーズロールにおいて、1.0 を選択して「挿入」を押します。
同様に 2.0, 3.0, 4.0 を選択して「挿入」を押します。

ポーズロールは次のようになるはずです。

.. image:: images/pose_role2.png

ポーズロールで作成したのはキーフレームと呼びます。これより、プログラムで使用するモーションを生成させます。
ツールバーから「ボディモーションの生成」ボタンを押します。

.. image:: images/motion_toolbar.png

モーションはツールバーのボタンで手動で生成しなくても、キーフレームの更新時に自動生成することができます。
これを有効にするにはツールバーの「自動更新モード」のボタンをオンにしてください。

.. image:: images/motion_toolbar2.png

SampleMotion の子供に motion があるので、これを選択し名前を付けて保存ボタンを押します。

.. image:: images/item_motion.png

「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/」ディレクトリに「RobotMotion.yaml」というファイルで保存します。

コントローラのビルド
--------------------

「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/」ディレクトリに移動し、make コマンドを実行します。

「model/robot/RTC/」ディレクトリに「RobotTorqueControllerRTC.so」というファイルが作成されるはずです。

その後、次のコマンドを実行します。 ::

   sudo make install DESTDIR=/usr


シミュレーションを実行する
--------------------------

シミュレーションツールバーの「シミュレーション開始ボタン」を押します。
シミュレーションを実行するとロボットが崩れ落ちず、立ったままの状態になったはずです。

.. image:: images/simulation_no_controller.png


サンプルプロジェクトについて
----------------------------

このサンプルのプロジェクトファイルは「モデルファイルのインストール」でダウン ロードしたリポジトリの「model/robot/samples/sample3.cnoid」に保存されています。

.. toctree::
   :maxdepth: 2

