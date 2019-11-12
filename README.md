# モバイルOS自作
　このWikiはAndroidをベースに俺専用にカスタマイズするのが目的です．自分のための備忘録兼日記として不定期更新する予定です．ここに記す内容は筆者である都留がSecHack365でモバイルOSを開発する中で具体的にどの様な手順を踏んだのかをコードや画像を交えて書き残していくつもりです．
予定や最終的にどの様なモノを作るのかについては下記の課題を参照してください．
-  SECHACK365_TRAINEE2019-85 Androidハック

## まえがき
***
　さて皆さんは，Androidアプリを作った経験はありますか？スマホが普及した現代ならば情報の学生ならば殆どの方が作った経験はあると思われます．では，Androidのソースコードをリポジトリからクローンしてビルドしたことはありますか？....殆どの方はいないですよね．何を今更Androidのビルドをしなくちゃならないんだ．とか，Androidのソースってオープンソースなん？とか思ってる方も沢山いらっしゃります(私が話をしたSecHack365のトレーニーの方はほぼほぼこれにあたりました)．至って当たり前のことですが，実はこれが盲点であり，Androidの世界シェアがダントツ1位であることや新規モバイルOSが開発されにくいことの裏付けとなっています．自分自身の手でモバイルOSを作ることで既存のAndroidやIOSには搭載していない機能を組み込むことで新しいアイデアを形にすることを目標として進めていきます．

## 目標
***
　はじめのAndroidをAOSPからクローンしてビルド
- Nexus 5 (Hammerhead)
- Android 6.0.0 r1 (marshmallow)

## 目次
***
1. 環境構築
2. ディレクトリ構造
3. 実機書き込み

## 1. 環境構築
***
　さて，AndroidをカスタマイズするにあたってAndroidをビルド・エミュレートする環境が必要です．しかし，ソースの量が膨大なAndroidはビルドするだけでもかなり高スペックなPCが必要です．そこでビルド用サーバを用意してそこで生成したイメージをローカルPCでエミュする流れで進めていきます．

環境
- SecHack365貸出ビルド用サーバ (ビルド用サーバ)
  - 
- Surface book 2 (ローカルPC)
  - 
- Nexus 5 (ターゲット端末)
  - 

### 1.1. ビルド用サーバ
***
 #### 必要なパッケージのインストール ####  

`$ sudo apt update`  
`$ sudo apt -y install curl make zip unzip \` 
`git python default-jre openjdk-8-jdk \`
`bison g++-multilib gcc-multilib libxml2-utils`

 #### ソースコードのダウンロードに必要なツールrepoをインストールする ####  

`$ mkdir ~/bin`  
`$ PATH=~/bin:$PATH`
`$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo`  
`$ chmod a+x ~/bin/repo`

 #### プロジェクト用のフォルダを作成する #### 

`$ mkdir AOSP`
`$ cd AOSP`

 #### manifestを作成し，リポジトリをクローンする #### 

`$ repo init -u https://android.googlesource.com/platform/manifest -b android-8.1.0_r1`
`$ repo sync -j12`

-b android-8.1.0_r1 でチェックアウトするブランチを選択しています．
一覧は https://source.android.com/setup/start/build-numbers.html#source-code-tags-and-builds
repo sync でリポジトリをクローンすることができます．
repo sync -j12 のようにすることで並列実行できます(並列させる分CPUパワーとネットワーク帯域を多く使うので注意)．

 #### ビルド前準備 #### 

`$ export LC_ALL=C #LC_ALL=Cでないとビルドがこける`  
`$ source build/envsetup.sh #環境設定`
`$ lunch aosp_x86-eng #ターゲットの選択`

 #### ビルドする #### 

`$ make -j12`

repo sync 同様， make -j12 のようにすることで並列実行できます．
### 1.2. ローカルPC
***
#### kvmの設定 #### 

`$ sudo apt install -y qemu-kvm libvirt0 libvirt-bin virt-manager bridge-utils`  
`$ sudo adduser $USER kvm`

 #### エミュレータの環境設定 ####  
emulatorコマンド利用するために，下記を実行してパスを通す．

`$ source build/envsetup.sh`  
`$ lunch aosp_x86-eng`

 #### エミュレータの実行 #### 

`$ emulator`

out/ にビルド成果物が入っています．
emulator コマンドは out/target/product/generic_x86/以下のイメージファイルを探して実行してくれるみたいです．

 
### 1.3. ターゲット端末
***
#### OS書き込み準備 ####

1. Androidを立ち上げて 設定→デベロッパーモード→USBデバッグON

2. ブートローダーを起動  
`$ adb reboot bootloader`

3. oemロック解除  
`$ fastboot oem unlock`  
fastboot コマンドで使用できるコマンドが増える．
fastboot flash コマンドなどメーカーが使用を制限していた場合，アンロックすると使用可能になったりする．(アンロック前にコマンドを実行すると「FAILED」となるようなもの)初期搭載のブートローダーは通常，メーカー署名入りのAndroidシステムのみ起動できるようになっています．アンロックをすることで未署名のカスタムROMが起動できるようになります．

4. root 化  
CF-Auto-Root-hammerhead-hammerhead-nexus5 をダウンロードします．
ホストOSのWindows上で展開したCF-Auto-Root-hammerhead-hammerhead-nexus5内のroot-windows.batをダブルクリックする．

5. 