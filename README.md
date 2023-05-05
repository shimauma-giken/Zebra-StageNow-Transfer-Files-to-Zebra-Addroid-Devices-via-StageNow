<!-- 罫線を表示 -->
<style type="text/css"> 
    table { border-collapse: collapse; } 
    table, th, td { border: 1px solid black; } 
</style>

# 外部FTP/HTTPファイルサーバからZebra Anddoid端末にStageNowを用いてファイルを転送する方法

Apk以外にもEHSやDatawedgeなどの設定ファイルを転送するケースが結構あるので、覚えておくと便利。実際の設定を用いて説明。

---

### 1. FTP Server側の設定

転送元にファイルを用意する。

    zebra@raspberrypi0:~/stagenow/file $ ps -ef |grep ftp
    root     20920     1  0 14:54 ?        00:00:00 /usr/sbin/vsftpd /etc/vsftpd.conf
    zebra    21192 20999  0 15:49 pts/0    00:00:00 grep --color=auto ftp
    zebra@raspberrypi0:~/stagenow/file $ pwd
    /home/zebra/stagenow/file
    zebra@raspberrypi0:~/stagenow/file $ echo 'Hello Stagenow!' > test.txt
    zebra@raspberrypi0:~/stagenow/file $ cat /home/zebra/stagenow/file/test.txt ★ 転送元ファイル
    Hello Stagenow!
    zebra@raspberrypi0:~/stagenow/file $

<br>


### 2. 転送先デバイスの情報を確認
---

転送先にディレクトりを用意する。

    PS C:\Users\mogetan> adb devices
    List of devices attached
    22260524700499  device

    PS C:\Users\mogetan> adb shell
    ET45S:/ $ cd /sdcard/Download/ ★ 転送先ディレクトリ
    ET45S:/sdcard/Download $ ls -al
    total 0
    ET45S:/sdcard/Download $ echo "No file exists"
    No file exists
    ET45S:/sdcard/Download $

▲ Download下にtest.txtを送信するシナリオを想定

<br>

### 3. StageNow側の設定
---

転送先・元情報を設定する。


![](./StageNow%20FileMgr-External-FTP-01.png)
▲ Xpert Mode > FileMgr を選択。

<br>

![](./StageNow%20FileMgr-External-FTP-02.png)
▲ URIはフルパスで記載すること。

<br>

#### # 参考情報、XML

    <wap-provisioningdoc>
    <characteristic version="10.1" type="FileMgr">
        <parm name="FileAction" value="1" />
        <characteristic type="file-details">
        <parm name="TargetAccessMethod" value="2" />
        <parm name="TargetPathAndFileName" value="/sdcard/Download/myfile.txt" /> ★
        <parm name="IfDuplicate" value="1" />
        <parm name="SourceAccessMethod" value="1" />
        <parm name="SourceURI" value="&amp;#1272&amp;#127bgrch5q5J4ZQyTVHVyqYLxJDBvkL1QyZ0dxy9Gc4/fIuemZTrwD0T+D2dvsndyx2gMx0uMXoxgJxl/a0HttVXH//fcqSTqPd10vAVUpzOImZpUTRK85o+4UvNSPou3sRyQtCWXnld8DoyZlPmqkdgosYhIyuKHD0EQsY24eioqMMbRMfHwh+VoB+9c0sKdXiMt9YZoX199W+h3SNFJ5RMtp2r5URPIxy5s+M07DJDLOroacOq97RbecldY0MgSejgJLIjCz39y98lMqlUbg+oalMjYumEOs+X2UWZOUkrT1xnWoGUcls4NLQCgL+lQ4fOwdhHY0x1jyauJjldiX+1Q==" />
        </characteristic>
    </characteristic>
    </wap-provisioningdoc>

▲ Source URIは認証情報が含まれているので暗号化されている。

<br>
<br>


## 4. Staging Barcodeをスキャン
---

StageNow の"公開"にて任意のBarcodeを作成し、Android > StageNowで作成されたバーコードをスキャンする。正常終了のメッセージを確認する。
<br>

![](./StageNow%20FileMgr-External-FTP-03.png)

<br>
<br>
<br>

## A. Trouble Shooting - うまくいかないときは。
---


    XXXXX:/sdcard/Download $ cd /storage/emulated/0/stagenow/logs
    XXXXX:/storage/emulated/0/stagenow/logs $ ls -al
    -rw------- 1 u0_a255 u0_a255   0 2023-05-04 15:47 StageNowClient.log
    -rw------- 1 u0_a255 u0_a255 332 2023-05-04 15:39 StageNowClient.log.1
    -rw------- 1 u0_a255 u0_a255   0 2023-05-04 15:35 StageNowClient.log.1.lck

    ET45S:/storage/emulated/0/stagenow/logs $ cat StageNowClient.log.1
    {"dateAndTime":"2023\/05\/04 午後3:35:38","errors":"[{\"key\":\"FileMgr\",\"value\":\"ERROR, FileAction-ファイルが見つかりません\",\"solution\":\"\"}]"}{"dateAndTime":"2023\/05\/04 午後3:35:38","errors":"[{\"key\":\"FileMgr\",\"value\":\"ERROR, FileAction-ファイルが見つかりません\",\"solution\":\"\"}]"}ET45S:/storage/emulated/0/stagenow/logs $

1. StageNow Logを確認するのが基本となる。
1. XMLファイルも併せて確認する。

<br>

### B. 詳細は本家バイブルを確認すること
---

- FTP以外のプロトコル利用時は下記リンクのXMLを参照のこと。  
    https://techdocs.zebra.com/stagenow/3-0/csp/file/#source-uri

- Download File from Remote FTP Server Using StageNow Tool  
https://supportcommunity.zebra.com/s/article/Download-File-from-Remote-FTP-Server-using-StageNow-Tool?language=en_US

    
<br>

### C. 付録：Stagenow を用いて自動インポート/ 設定サンプル一覧
---

| Configirations    | Values    |
|-|-|
| StgNow profile    | FileMgr-DataWedge-Profile-02-ftp-Auth
| Target path & file| /enterprise/device/settings/datawedge/autoimport/dwprofile_chrome.db
| Source URI        | ftp-p://zebra:zebra@192.168.4.55:21/home/zebra/stagenow/file/dwprofile_chrome.db
| Result            | Success

<br>

| Configirations    | Values    |
|-|-|
| StgNow profile    | FileMgr-DataWedge-Profile-04-http-NoAuth
| Target path & file| /sdcard/Download/myfile.csv
| Source URI        | http://192.168.4.55/backup/csv01/sample.csv 
| Result            | Success









