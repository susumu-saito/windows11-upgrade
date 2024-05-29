# windows11-upgrade
**参考要件**
|            |                                                               |
| :--------- | :------------------------------------------------------------ |
| CPU        | AMD Ryzen 7 3700X（コードネームMatisse）                      |
| マザボ     | AsRock X570 Steel Legend（BIOSバージョン1.60）                |
| メモリ     | DDR4SDRAM 32GB                                                |
| ストレージ | 1TB SSD（型番CSSD-M2B1TPG3VNF、PCIe Gen.4 x4 (NVMe 1.3)対応） |
| グラボ     | AMD Radeon RX580（メモリ8GB）                                 |


## 目次
1. Windows 11の要件にやや癖あり
2. 対応したこと
3. 対応の流れ（実際に近い生々しいもの）
4. 最後に


## 1. Windows 11の要件にやや癖あり
[Windows 11の公式サイト](https://www.microsoft.com/ja-jp/windows/windows-11)を開くと、対応PCかどうかのチェックツールをダウンロードすることができます。  
対応PCについては以下のスペックが出てきます。

|                |                                                                                                                                                 |
| :------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| CPU            | 1GHz・2コア以上の64ビット互換CPU                                                                                                                |
| メモリ         | 4GB                                                                                                                                             |
| ストレージ     | 64GB以上                                                                                                                                        |
| ファームウエア | UEFI・セキュアブート対応必須                                                                                                                    |
| TPM            | TPM 2.0対応必須                                                                                                                                 |
| グラボ         | DirectX 12以上・WDDM 2.0ドライバー対応必須                                                                                                      |
| ディスプレイ   | 720P（1,280×780）以上の高解像度ディスプレイ                                                                                                     |
| その他         | 初めて使用するときなど、Microsoftアカウントとインターネット接続が必要。更新プログラムや一部の機能のダウンロード・使用にはインターネットが必要。 |


※AMD Ryzen 7 3700XであればfTPM（TPM 2.0対応）というものが備わっており、問題ありませんでした。

## 2. 対応したこと
結論から言いますと、以下の設定を行いました。

1. fTPMを有効に変更。
2. コマンドライン（管理者モード）でmbr2gpt /convert /disk:0 /allowFullOSを実行しSSDをMBR→GPTに変換。
3. CSMをOFFに変更（Fast BootもONに変更）。
4. セキュアブートを有効に変更。
5. それでもダメなので、BIOSを最新版に更新（CPUがMatisseの場合の対応確認済）
6. Windows 11対応

    <img src="https://pbs.twimg.com/media/E4y8Qn9UYAQwqaV?format=jpg&name=small" alt="PC正常性チェック" width="500px">

    <img src="https://pbs.twimg.com/media/E4y7zLvUUAMsb0J?format=jpg&name=small" alt="PC正常性チェック" width="500px">

## 3. 対応の流れ（実際に近い生々しいもの）
- STEP1 TPM 2.0への対応  
    UEFIの設定画面を開き、CPU設定＞fTPMを有効にしました。

    AMDの場合はこれで良いそうです。Ryzen 2000シリーズ以上で対応してるとか、かんとか。

    Microsoftが対応CPUを公表するようになりましたね。

    Intelの場合はこちら
    AMDの場合はこちら
    Intelの場合は、Intel PTT対応かどうかがキーとなるようですが、これはマザボによるそうです。

    TPM 2.0が有効になっているかどうかは、設定＞システム＞詳細情報＞Windowsセキュリティで詳細を確認する、をクリック。Windowsセキュリティが表示されたら、デバイスセキュリティ＞セキュリティプロセッサの詳細をクリック。

    <img src="https://tecchan.jp/wp-content/uploads/2021/06/windows_security-1024x802.jpg?_mod=1658329543" alt="PC正常性チェック" width="500px">


    仕様バージョンが2.0になっていればOKです。


- STEP2 セキュアブートを有効にする  
    UEFIに戻り、起動＞セキュアブートを有効にしてみます。

    …が、Winsowsで確認すると、セキュアブートが有効になっていません。
    <img src="https://tecchan.jp/wp-content/uploads/2021/06/secure_boot-1024x805.png?_mod=1658329543" alt="PC正常性チェック" width="500px">

    調べていく過程で、BIOSモードがレガシになっています（UEFIが有効になっていない状態）。


- STEP3 BIOSをレガシ→UEFIに変更するには？  
    まずはCSMを無効に！
    ここで一番時間を食ってしまったのですが、まずはBIOSを無効にしてUEFIに変更するには、まずはCSMというものを無効にしなくてはなりません。その段階で注意すべきことが、「セキュアブートを有効にするにはCSMを無効にしなければならない」とのことでした。思いっきり有効にしていたよ…

    また、セキュアブートに対応するには、ストレージをMBRからGPTに変換しなくてはならないとのこと。しかも、変換をミスった方がいらっしゃるとかかんとか…（恐ろしい）

    つまり、以下の順です。

    1. ストレージをMBRからGPTに変換する
    2. CSMを無効にする
    3. セキュアブートを有効にする
    4. ついでに、Fast Bootを有効にしておく
    5. 再起動する


- STEP4 MBRをGPTに変換する  
    失敗する可能性もあるとのことで、まずはファイルのバックアップを実行し、バックアップ終了後にコマンドプロンプトを右クリックして「管理者として実行」し、以下のコマンドを入力して実行しました。

    ```cmd
    mbr2gpt /convert /disk:0 /allowFullOS
    ```
    思った以上にすんなりといきました。終わったらPCを再起動してUEFIを開きます。


- STEP5 UEFIの設定変更  
    以下の手順で設定していきます。

    1. CSMを無効にする
    2. セキュアブートを有効にする
    3. ついでに、Fact Bootを有効にしておく
    4. 再起動する

<img src="https://pbs.twimg.com/media/E4yAVG3VkAAaXvV?format=jpg" alt="PC正常性チェック" width="500px">
<img src="https://pbs.twimg.com/media/E4yAV0YUYAMMPEe?format=png" alt="PC正常性チェック" width="500px">

- STEP6 BIOSを更新  
    AsRockの公式サイトより、最新バージョンのBIOSにアップデートしました。

    まず最初にfTPMを無効にしないと、BIOSがアップデートできませんでした。

    1. fTPMを無効
    2. BIOSアップデート実行。実行後にすべての設定が初期化される。
    3. CSMを無効にする
    4. セキュアブートを有効にする
    5. ついでに、Fast Bootを有効にしておく
    6. fTPMを再度有効にする
    7. 再起動する
    
    基本的に、BIOS（UEFI）はシステム運行が良好な場合にはアップデートしない原則がありますが、今回の場合はWindows 11に対応しないので、これも調子悪いってことかな？と思って、念のため一気にアップデートして良いかどうか確認した上でアップデート。成功。

    今度こそ上手くいきました。

    <img src="https://pbs.twimg.com/media/E4y8Qn9UYAQwqaV?format=jpg&name=small" alt="PC正常性チェック" width="500px">


## 参考
- [Ryzen7 3700X](https://tecchan.jp/entry/210628-windows11/)
- [MBRからGPTに変換](https://partition.aomei.jp/gpt-mbr/convert-mbr-to-gpt-without-data-loss.html#:~:text=%E3%80%8C%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%82%BF%E3%83%BC%E3%80%8D%E3%81%BE%E3%81%9F%E3%81%AF%E3%80%8CPC%E3%80%8D,%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%8C%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%E3%80%82)
