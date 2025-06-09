# Lab 2 Open5GS&UERANSIM

## Outline
- [1. 實驗內容](#1實驗內容)
- [2. 相關知識](#2-相關知識)
- [3. 相關建置環境/設備規格](#3-相關建置環境設備規格)
- [4. 實驗架構](#4-實驗架構)
- [5. 實驗流程](#5實驗流程)
    - [5-1 架設 Open5GS](#5-1-架設-open5gs)
        - [問題與討論(一)](#問題與討論一)
    - [5-2 安裝與架設UERANSIM](#5-2-安裝與架設ueransim)
        - [問題與討論(二)](#問題與討論二)
    - [5-3 UERANSIM 外網測試](#5-3-ueransim-外網測試)
        - [問題與討論(三)](#問題與討論三)
------ 

## 1.實驗內容
- 建立一個 5G 專網，架設 Open5GS 核心網路、gNB、UE
- 學習如何模擬手機連線
- 使用 Iperf3 進行 Downlink ICMP 和 Uplink ICMP 連線測試
------

## 2. 相關知識
- Open5GS:基於 3GPP 規範的 5G 核心網路 (5GC) 的開源軟體。 為建立和測試 5G 網路提供功能齊全且靈活的平台。 Open5GS 的關鍵元件包括：存取和行動管理功能(AMF)、會話管理功能(SMF)、使用者平面功能 (UPF)、網路儲存庫功能 (NRF)、服務能力暴露功能（SCEF）、訂閱者資料管理 (SDM)、統一資料管理（UDM）等。
 
- UERANSIM：OpenAirInterface軟體聯盟開發的一款用於 5G 用戶設備（UE）和無線存取網路（RAN）的開源模擬工具。模擬 5G UE 在各種網路部署場景中的行為。

- MongoDB是一個流行的開源 NoSQL 資料庫管理系統。它使用文件導向的資料模型，而不是傳統的關係型資料庫表結構。MongoDB的主要特點和概念包括：

    - 文檔導向的資料模型：MongoDB 儲存資料的基本單元是文檔，這些文檔採用類似 JSON 的格式，包含各種類型的資料。這種資料模型使得 MongoDB 非常適合處理複雜的資料結構和半結構化資料。

    - 靈活的模式：與傳統的關聯式資料庫不同，MongoDB 不要求文件具有相同的結構或模式，這使得在開發過程中可以更靈活地進行模式設計和資料建模。這種靈活性使得 MongoDB 更適用於快速迭代和麵對變化的應用程式。

    - 分散式儲存：MongoDB 支援水平擴展，可以在多台伺服器之間分散數據，以應對大規模資料儲存的需求。
------

## 3. 相關建置環境/設備規格
### [Ubuntu安裝](https://github.com/TTU-CWBT/Ubuntu-install-tutorial)
:warning:這次的實驗需要安裝 Ubuntu Desktop，請在安裝完 Ubuntu 後在終端機輸入安裝桌面的指令，這個指令會需要花一些時間，請耐心等待。
``` shell=1
sudo apt install ubuntu-desktop
```

| 軟體名稱 | 軟體版本  |
|:-------- |:---------:|
| Ubuntu   | 22.04 LTS |
| Open5gs  |  v2.7.1   |
| UERANSIM |  v3.2.6   |

------

## 4. 實驗架構
- 目前架構都以 VM 進行
- 核心網路：Open5GS
- 模擬 Radio 訊號：UERANSIM

------

## 5.實驗流程
### 5-0 前情提要
由於以下實驗會有大量複製貼上指令的動作，所以建議在本地端的終端機使用SSH連線至虛擬機(VM)的方式進行，SSH 的連線方式如下。
- Connect to a remote server:
``` shell=1
ssh username@remote_host
或是
ssh username@remote_IP

example 
ssh open5gs@10.0.2.12
```
> 常見錯誤
> 如果發現找不到 IP，可以透過 `ip a` 查看 IP address
> 如果沒有 IP address，可以輸入 `sudo dhclient` 來代管網卡，然後再次輸入 `ip a` 指令查看 IP
> 如果連線不穩定，可以改用 Host Only 的 IP address

------

### 5-1. 架設 Open5GS
#### Ubuntu 基本套件安裝
- 請輸入以下指令至 terminal
``` shell=1
 sudo apt update && sudo apt upgrade
 sudo apt-get install vim
 sudo apt install net-tools
 sudo apt install traceroute
 sudo apt install tldr
 sudo apt install iperf3
```
#### 安裝 MongoDB
- 安裝過程中可能會造成網路中斷，重新連線即可，依序輸入
##### x86
``` shell=1 驗證mongodb x86
sudo apt install gnupg curl
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```
##### arm64
``` shell=1 驗證mongodb m1
sudo apt install gnupg curl
sudo apt install software-properties-common gnupg apt-transport-https ca-certificates -y
curl -fsSL https://pgp.mongodb.com/server-7.0.asc |  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse"
sudo apt update
sudo apt install mongodb-org -y
mongod --version
```
>常見錯誤
>![5-1-1 MongoDB常見錯誤](https://hackmd.io/_uploads/HyRx18bMle.png)
> M 系列的 Mac 可能會出現這個問題，可以參考以下網址
https://www.cherryservers.com/blog/install-mongodb-ubuntu-22-04

- 檢查 mongodb 版本
> mongod --version
![5-1-2 mongodb version](https://hackmd.io/_uploads/HkSUG_Zfel.png)

檢查 mongodb 是否正常運作
```  shell=1
sudo systemctl status mongod
```
>正常運作會長這樣
![5-1-3 mongodb status](https://hackmd.io/_uploads/Bk7NGdbMel.png)

#### 安裝 Open5GS
##### 下載 Open5GS & UERANSIM 相關套件

``` shell=1
sudo apt install cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev git
```
安裝完畢後請依序輸入以下指令
``` shell=1
sudo add-apt-repository ppa:open5gs/latest
```
> 按下Enter
``` shell=1
sudo apt update
sudo apt install open5gs
輸入y
```

#### 安裝 node.js
請依序輸入以下指令
``` shell=1
sudo apt install -y ca-certificates
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update
sudo apt install nodejs -y
```

#### 安裝 Open5GS-webUI
請輸入以下指令
``` shell=1
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```
>會出現以下畫面，需耐心等待

![5-1-4 install open5gs webui](https://hackmd.io/_uploads/HJobUuWMeg.png)
##### 註冊 UE 用戶
安裝完畢後，需要回到虛擬機中（此時不能依賴 SSH 連線，會失敗），在『**虛擬機**』 開啟瀏覽器並輸入`127.0.0.1:9999`，第一次開啟會需要稍等一下，若成功開啟後，就代表 Open5GS 有順利安裝成功，接著輸入預設密碼
> Username: admin
> Password: 1423

![5-1-5 login webui](https://hackmd.io/_uploads/Hk8mr0ZQgx.png)

> 登入成功後會看到以下介面

![5-1-6 註冊UE](https://hackmd.io/_uploads/HyuXBRWmxg.png)
> 然後點擊右下角紅色的『+』並開始編輯以下資料
> IMSI 輸入我們設定的:001010000000000
> Type 修改為 IPv4 

![5-1-7 ISMI](https://hackmd.io/_uploads/H1BVwqz7xg.png)

#### 驗證 Open5GS 安裝成功
如果 Open5GS 安裝成功後，每次重新開機後都會自動在背景執行，輸入`ps aux | grep open5gs` 可以確認 Open5GS 核心功能是否正常運作，若成功畫面會如下（請確認已以下這些功能是否正常，若不正常請勿執行下一步驟）
![5-1-8 驗證open5gs](https://hackmd.io/_uploads/SJZW_oZGxe.png)
> 核心功能如下
> AMF, SMF, UPF, NRF, UDM, UDR, PCF,AUSF, NSSF

確認核心功能都啟用後，便可以開始設定核心網路的 amf, smf, upf功能了
##### 1. 設定 amf.yaml
輸入`sudo vim /etc/open5gs/amf.yaml`編輯 amf，需要修改的如下
> mcc、mnc:'001', '01'
> Address:open5gs 的 hostonly IP

![5-1-9 amf](https://hackmd.io/_uploads/rkKp25f7lx.png)
完成後輸入 `sudo systemctl restart open5gs-amfd.service` 重新啟動 amf
##### 2. 設定 nrf.yaml
輸入 `sudo vim /etc/open5gs/nrf.yaml` 編輯 nrf，需要修改的如下
> PLMN ->mmc=001 mnc=01

![5-1-10 urf](https://hackmd.io/_uploads/SJ-109Gmxl.png)
完成後輸入 `sudo systemctl restart open5gs-urfd.service` 重新啟動 urf

##### 3. 設定 upf.yaml
輸入`sudo vim /etc/open5gs/upf.yaml`編輯 upf，需要修改的如下
> 將 gtpu 的 IP 設定為核網 host only 的 IP

![5-1-11 upf](https://hackmd.io/_uploads/HkrhR9MQxg.png)
完成後輸入 `sudo systemctl restart open5gs-upfd.service` 重新啟動upf

##### 4. 設定網路功能(路由、 NAT 轉換與防火牆功能)
輸入以下指令設定網路功能，以下指令每次重新開機都需要重新輸入設定，若沒有輸入以下設定，核心網路功能會受限或無法使用！
> :warning:在執行 `sudo iptables -t nat -A POSTROUTING -o enp0s1 -j MASQUERADE` 指令前，請特別確認 `enp0s1` 是否為 Open5GS VM 中對應 Shared Network 的網卡名稱。
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o enp0s1 -j MASQUERADE
sudo systemctl stop ufw
```
為了方便已將指令寫成腳本，學生們只要建立一個`.sh`的檔案，並將以下內容複製貼上並儲存，以後開機只要輸入 `sh filename.sh` ，就可以自動設定了
- 網路設定shell script
``` shellscript=1
#!/bin/bash
# Program:
# this is auto setting network
# History:
# 2025/05/16    Timothy_Su_TZD   First version
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o enp0s1 -j MASQUERADE
sudo systemctl stop ufw

exit 0
```

設定完成後基本上就完成了核心網路的所有設定，接下來就則要設定 UE & RAN

------

### 問題與討論（一）
1. 截圖 Open5GS Web UI註冊成功畫面
2. 解釋什麼是 IMSI 與 IPv4
3. 請解釋 AMF, NRF, UPF, SMF分別負責什麼工作？
4. 請解釋 AMF 中的 ngap 是什麼？

------

### 5-2. 安裝與架設UERANSIM
##### 前置作業
在安裝 UERANSIM 時會需要另一個獨立的虛擬機，不過現在不需要重新安裝一次 Ubuntu，只需要複製之前安裝的 Open5GS，並更改新虛擬機（UERANSIM）的網卡 MAC Address 即可，如下圖
![5-2-1 copy_vm](https://hackmd.io/_uploads/r1KF-sGQlx.png)
> 右鍵選擇虛擬機（UERANSIM）>> Edit >> Network >> Shared Network && Host Only >> MAC Address >> Click Random >> Save
![5-2-2 network randem](https://hackmd.io/_uploads/rJs6EsMQex.png)

接著開啟 UERANSIM (登入密碼與open5gs虛擬機登入密碼一樣），並且開啟 terminal（這裡建議使用 SSH 連線，因為也會有需要大量複製的需求），為了後續方便辨識建議使用者將 UERANSIM 的 username 改為 ueransim，更改方法可以參考[ubuntu下修改主机名、用户名以及用户密码](https://https://blog.csdn.net/qq_34160841/article/details/106886306)的文章，將UERANSIM虛擬機的username改成這樣。
![5-2-3 rename vm](https://hackmd.io/_uploads/ByAGwsMXxx.png)

##### 安裝UERANSIM
- 把 UERANSIM 從 GitHub 上面抓下來
``` shell=1
cd ~
git clone https://github.com/aligungr/UERANSIM
sudo apt update
```
- 安裝 UERANSIM 相關工具
``` shell=1
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo apt-get install build-essential
```
> 若出現 Cmake 版本過舊，請參考下列安裝較新版的 CMake
``` shell=1
sudo apt remove cmake
wget [https://cmake.org/files/v3.22/cmake-3.22.0.tar.gz]
tar xvf cmake-3.22.0.tar.gz
./configure
Make –j8
sudo make install
```

- Building UERANSIM
輸入以下指令，輸入 `make` 時跑比較久是正常的，切勿中斷指令執行！若都沒有錯誤，那就是安裝成功了！
``` shell=1
cd ~/UERANSIM
make
```
>make 執行畫面

![5-2-4 make](https://hackmd.io/_uploads/SJEhzhfXeg.png)

> 看到 `UERANSIM successfully built.` 就是成功了！
![5-2-5 make success](https://hackmd.io/_uploads/H1RWQ2GQxl.png)


##### 設定 UERANSIM
- gNB 設定
輸入`vim ~/UERANSIM/config/open5gs-gnb.yaml`編輯 gNB 設定，需要修改的如下
>mcc、mcc修改為’001’和’01’
>linkIp、 ngapIp、 gtpIp修改為 UERANSIM VM的 host only IP
>amfConfigs 的 address 修改為Open5GS VM的 host only IP
>![5-2-6 gnb setting](https://hackmd.io/_uploads/BJESnoGXee.png)

- UE 設定
輸入 `vim ~/UERANSIM/config/open5gs-ue.yaml` 編輯UE設定，需要修改的如下
> Supi:001010000000000
> mcc、mnc:001,01
> gnbSearchList:UERANSIM VM的 host onlyIP
> ![5-2-7 ue setting](https://hackmd.io/_uploads/B1nQpjzXgx.png)

設定完成後，就可以開啟 gNB 與 UE 的模擬了
- Open gNodeB
``` shell=1
cd ~/UERANSIM/build
sudo ./nr-gnb -c ../config/open5gs-gnb.yaml
```
> 成功開啟後會顯示 `NG Setup procedure is successful`
> ![5-2-8 open gnb](https://hackmd.io/_uploads/HyQBOhMQge.png)


- Open UE
``` shell=1
cd ~/UERANSIM/build
sudo ./nr-ue -c ../config/open5gs-ue.yaml 
```
> 成功開啟後會顯示 `Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.`
> ![5-2-9 openue](https://hackmd.io/_uploads/rk8fK2MQxx.png)
> 並且在 gNB 的畫面也會同時新的顯示連接成功的訊息

- 檢查 Open5GS 連線紀錄
- 我們可以在open5GS VM 中輸入`sudo cat /var/log/open5gs/amf.log`，便可以觀察到 gNB 和 UE 連線資訊。
 ![5-2-10 amf log](https://hackmd.io/_uploads/rJVE22MXlx.png)

------

### 問題與討論（二）
1. 請截圖 gNB 與 UE 順利連接至 Open5GS 之畫面
2. 請在 /var/log/open5gs/amf.log 找出 gNB 與 UE 成功連線的紀錄並截圖

------

### 5-3. UERANSIM 外網測試
:warning: **請維持 gNB 與 UE 與 Open5GS 間的正常連線**

如果UE有成功透過gNB再經由核心網路註冊成功，我們可以透過`ip a`的指令在UERANSIM VM都看到一張名為 `uesimtun0` 的網卡（如下圖）
![5-3-1 ursimtum_ip](https://hackmd.io/_uploads/SJC-O6f7lx.png)
這張網卡就是UE的網卡，如果我們把UE的指令關掉，這張網卡便會消失，如果想要讓UE可以連接外網，就會需要設定 ip route，請輸入以下指令。
``` shell=1
sudo ip route add default dev uesimtun0
```
如果要確認有沒有連接至外網，可以透過兩種方式確認，邏輯都是如果可以成功 ping 到 `google.com`，就是有成功。

1. ping
 
``` shell=1
ping -I uesimtun0 google.com 
ping -I uesimtun0 8.8.8.8
這兩個指令是一樣的，則一即可
這個指令會一直執行，可以透過Ctrl+c停止
```
![5-3-2 ping](https://hackmd.io/_uploads/ByXG1XQQxl.png)


2. traceroute
``` shell=1
traceroute google.com -i uesimtun0 -n
```
![5-3-3 traceroute](https://hackmd.io/_uploads/r1NSkQQQex.png)


> :warning:如果沒有成功 ping 或是 tracetoute 都一直停在『* * * 』就是失敗，請在Open5GS VM重新輸入以下指令   
> :warning:在執行 `sudo iptables -t nat -A POSTROUTING -o enp0s1 -j MASQUERADE` 指令前，請特別確認 `enp0s1` 是否為 Open5GS VM 中對應 Shared Network 的網卡名稱。   
>
>``` shell=1
> sudo sysctl -w net.ipv4.ip_forward=1
> sudo iptables -t nat -A POSTROUTING -o enp0s1 -j MASQUERADE
> sudo systemctl stop ufw
> ```

#### UL/DL ICMP 連線測試 
:warning: **請維持 gNB 與 UE 與 Open5GS 間的正常連線**
請在Open5GS VM輸入以下指令，並持續到UL/DL ICMP測試結束為止。
```
sudo iperf3 -s
```
![5-3-4 iperf3 -s](https://hackmd.io/_uploads/r1dVjNQmgx.png)
> 輸入後會顯示以上結果
##### TCP
- Uplink ICMP連線測試
在 UERANSIM VM 中輸入以下指令，並請確認 `10.45.0.1` 是否為 Open5GS VM 中 `ogstun` 網卡的 IP，以及 `10.45.0.4` 是否為 UERANSIM VM 中 `uesimtun0` 的 IP。
``` shell=1
sudo iperf3 -c 10.45.0.1 -B 10.45.0.4 -b 100M -t 10
```

- Downlink ICMP 連線測試
``` shell=1
sudo iperf3 -c 10.45.0.1 -B 10.45.0.2 -b 100M -t 10 –R 
```
![5-3-5 TCP conn](https://hackmd.io/_uploads/HkM1hNm7ex.png)
> 如果順利連線會顯示上圖，左邊是 Open5GS VM，右邊是 UERANSIM VM

##### UDP
- Uplink ICMP 連線測試
``` shell=1
sudo iperf3 -c 10.45.0.1 -u -B 10.45.0.2 -b 100M -t 10 
```
- Downlink ICMP 連線測試
``` shell=1
sudo iperf3 -c 10.45.0.1 -u -B 10.45.0.2 -b 100M -t 10 –R 
```
![5-3-6 UDP conn](https://hackmd.io/_uploads/H14PlHXQge.png)
> 如果順利連線會顯示上圖，左邊是 Open5GS VM，右邊是 UERANSIM VM

------

### 問題與討論（三）

1. 請問實測出 TCP 最大的 UL 與 DL 大約是多少？
2. 請問實測出 UDP 最大的 UL 與 DL 大約是多少？
3. 請問 TCP 與 UDP 哪個的吞吐量較大？為什麼？
4. 如果需要選擇可靠度較高的通訊協定，應該選擇 TCP 還是 UDP？為什麼？ 


























