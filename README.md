# docker-mirakurun-chinachu-encoder

[公式のリポジトリ](https://github.com/Chinachu/docker-mirakurun-chinachu) をベースに, より軽量に, 更に録画後エンコード機能をデフォルトで内蔵

> mirakurun 82.6MB -> 68.7MB

> chinachu 641MB -> 467MB


## Contents
### Mirakurun
* Alpine Linux 3.5
* [Mirakurun](https://github.com/kanreisa/Mirakurun)
  * branch: master

### Chinachu
* Alpine Linux 3.5
* [Chinachu](https://github.com/kanreisa/Chinachu)
  * branch: gamma

### Encoder
* Preconfigured `ffmpeg`-based encoding (configurable)


## Test Environment
* OS: Ubuntu 16.04.2
* Tuner: PT3
* Card Reader: SCR3310-NTTCom
* Docker Engine: 17.03.0-ce
* Docker Compose: 1.8.1


## Prerequisites
* Docker環境
  * `Docker Engine 1.10+`
  * `Docker Compose 1.6+`
* Tuner Driver
  * 以下PT3の例  
```shell
sudo sh -c "echo \"blacklist earth-pt3\" >> /etc/modprobe.d/blacklist.conf"
git clone https://github.com/m-tsudo/pt3.git
cd pt3
make
sudo make install
sudo $SHELL ./dkms.install
# sudo reboot
```
* SELinux無効化
* [OPT] ホストでpcscdを動かしている場合は停止する  
```shell
sudo systemctl stop pcscd.socket
sudo systemctl disable pcscd.socket
```


## Usage
### 起動/停止
`runtime/` で実行する
```shell
docker-compose up -d
docker-compose down
```

### 初期設定
全てホストから設定する

#### 1. コンテナ設定
`runtime/docker-compose.yml` を調整する
* 録画ファイルの保存先  
`chinachu > volumes > recorded`
* chinachu WebUIのポートバインド  
`chinachu > ports`
* デーモン化  
`* > restart`

#### 2. チューナ設定
```shell
docker exec -it mirakurun mirakurun config tuners
```
以下PT3の例
```yml
- name: PT3-S0
  types:
    - BS
    - CS
  command: recpt1 --device /dev/pt3video0 <channel> - -
  decoder: arib-b25-stream-test
  isDisabled: false

- name: PT3-S1
  types:
    - BS
    - CS
  command: recpt1 --device /dev/pt3video1 <channel> - -
  decoder: arib-b25-stream-test
  isDisabled: false

- name: PT3-T0
  types:
    - GR
  command: recpt1 --device /dev/pt3video2 <channel> - -
  decoder: arib-b25-stream-test
  isDisabled: false

- name: PT3-T1
  types:
    - GR
  command: recpt1 --device /dev/pt3video3 <channel> - -
  decoder: arib-b25-stream-test
  isDisabled: false
```

#### 3. チャンネル設定
自動チャンネルスキャンを行う
```shell
docker exec mirakurun curl -X PUT 'http://localhost:40772/api/config/channels/scan'
```
ただし地上波のみなので (v2.2.0現在), BS・CSを利用する場合は以下のコマンドから手動で設定する
```shell
docker exec -it mirakurun mirakurun config channels
```

#### 4. chinachu設定
適宜WebUI (http://localhost:20772) から行う
* WUI (w/ Basic認証), ユーザ名/パスワード (デフォルトは認証なし)
* 録画ファイル名フォーマット  
エンコード後もこのファイル名を引き継ぐため, 拡張子は `*.mp4` を推奨
* etc.

#### 5. エンコード設定
デフォルトでは録画終了時に `v:H.264`, `a:copy`, `-tune animation` でエンコードを行い, 元ファイルを `recorded/dump` に退避させつつ置き換える. 必要に応じて `runtime/chinachu/recorded_command` で設定する.

#### 6. 再起動
一通り初期設定が完了したらコンテナを再起動する
```shell
docker-compose down && docker-compose up -d
```


### Others
* 各種設定ファイルは `runtime/` 下に永続化される
* アップデートはイメージ単位で行うことを想定 (`chinachu updater` は利用できない)
* エンコード元のTSファイルは `recorded/dump` 下に蓄積される. デフォルトでは `storageLowSpaceCommand` によって古くなると削除され, `storageLowSpaceAction` の管理からは外れている.


### Bugs / Future Works
See [Issues](https://github.com/amaya382/docker-mirakurun-chinachu-encoder/issues)


### Acknowledgements
本リポジトリは [Chinachu/docker-mirakurun-chinachu](https://github.com/Chinachu/docker-mirakurun-chinachu) を元に作られています
