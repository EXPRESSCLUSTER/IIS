# IIS のクラスタ化 (Windows Server 2019)

## 評価環境
- Windows Server 2019
  - IIS 10
- CLUSTERPRO X 4.3

```
+------------------------------+
| server0                      |
|  ActiveDirectory, DNS Server |
|  Domain name: local          |
+------------------------------+

+----------------------+
| server1.local        |   +--------------------+
|  Windows Server 2019 |   | M: ドライブ        |
|  IIS 10              +---+  M:\web\index.html |
|  CLUSTERPRO X 4.3    |   |                    |
+----------------------+   +--------------------+
                               |
                               | Mirroring
                               V
+----------------------+   +--------------------+
| server2.local        |   | M: ドライブ        |
|  Windows Server 2019 +---+  M:\web\index.html |
|  IIS 10              |   |                    |
|  CLUSTERPRO X 4.3    |   +--------------------+
+----------------------+
```


## サイト単位での制御および監視を行う
### 事前準備
1. 以下のリソースを持つクラスタを構築してください。IP アドレス、データパーティションは任意の値を設定してください。
   - フローティング IP リソース
     - IP アドレス: 192.168.1.11
   - ミラーディスクリソース
     - データパーティション: M ドライブ
1. server1, server2 に IIS をインストールしてください。
   ```powershell
   Install-WindowsFeature Web-Server -IncludeManagementTools
   ```
### CLUSTERPRO からサイトを制御する
1. フェイルオーバグループを server1 に移動してください。
1. ミラーディスク (e.g. M: ドライブ) 上に、任意のディレクトリ (e.g. M:\web) を作成し、テスト用に index.html を保存してください。
   - index.html のサンプル
     ```html
     <html>
       <body>
         Hello, test1.
       </body>
     </html>
     ```
1. server1 にて、[インターネット インフォメーション サービス マネージャー] を起動し、左ペインの [サイト] を右クリックして、[Web サイトの追加] を選択してください。
1. [Web サイトの追加] の画面にて、以下のパラメータを設定してください。
   - サイト名: test1
   - 物理パス: M:\web
   - IP アドレス: 192.168.1.11
1. DNS サーバに以下の A レコードを追加してください。
   - ホスト名: test1
     - FQDN 名に test1.local と表示されることを確認してください。
   - IP アドレス: 192.168.1.11
1. Web ブラウザまたは curl コマンドを実行し、test1.local, 192.168.1.11 でアクセスできることを確認してください。
1. [インターネット インフォメーション サービス マネージャー] にて、サイト test1 を停止してください。
1. フェイルオーバグループを server2 に移動してください。
1. server2 にて、 [インターネット インフォメーション サービス マネージャー] を起動し、左ペインの [サイト] を右クリックして、[Web サイトの追加] を選択してください。
1. [Web サイトの追加] の画面にて、以下のパラメータを設定してください。
   - サイト名: test1
   - 物理パス: M:\web
   - IP アドレス: 192.168.1.11
1. Web ブラウザまたは curl コマンドを実行し、test1.local, 192.168.1.11 でアクセスできることを確認してください。
1. server2 にて、 [インターネット インフォメーション サービス マネージャー] を起動し、左ペインの [サイト] を右クリックして、[Web サイトの追加] を選択してください。
1. Cluster WebUI を起動し、[設定モード] に切り替えてください。
1. フェイルオーバグループにスクリプトリソースを追加してください。また、start.bat, stop.bat で以下を実行するように設定してください。
   - start.bat
     ```bat
     C:\Windows\System32\inetsrv\appcmd.exe start site /site.name:test1
     ```
   - stop.bat
     ```bat
     C:\Windows\System32\inetsrv\appcmd.exe stop site /site.name:test1
     ```
1. 設定反映後、フェイルオーバグループを起動し、test1.local, 192.168.1.11 でアクセスできることを確認してください。

