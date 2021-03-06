---
title: Azure SDK for PHP をダウンロードする
description: Azure SDK for PHP をダウンロードしてインストールする方法について説明します。
documentationcenter: php
services: app-service\web
author: allclark
manager: douge
editor: ''

ms.service: app-service-web
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 06/01/2016
ms.author: allclark;yaqiyang

---
# Azure SDK for PHP をダウンロードする
## 概要
Azure SDK for PHP には、Azure 向けの PHP アプリケーションを開発、デプロイ、管理するためのコンポーネントが用意されています。Azure SDK for PHP には次のコンポーネントが用意されています。

* **Azure 用 PHP クライアント ライブラリ**。これらのクラス ライブラリには、データ管理サービスやクラウド サービスなどの Azure の機能にアクセスするためのインターフェイスが用意されています。  
* **Mac、Linux、Windows の Azure コマンド ライン インターフェイス (Azure CLI)**これは、Azure Websites や Azure Virtual Machines などの Azure サービスをデプロイおよび管理するためのコマンドのセットです。Azure CLI は、Mac、Linux、Windows など、すべてのプラットフォームで動作します。
* **Azure PowerShell (Windows のみ)**。これは、Cloud Services や Virtual Machines などの Azure サービスをデプロイおよび管理するための PowerShell コマンドレットのセットです。
* **Azure エミュレーター (Windows のみ)**。コンピューティング エミュレーターとストレージ エミュレーターは、アプリケーションをローカルでテストできるようにするためのクラウド サービスおよびデータ管理サービスのローカル エミュレーターです。Azure エミュレーターは Windows 上でのみ動作します。

以下のセクションでは、上に示したコンポーネントをダウンロードおよびインストールする方法を説明します。

このトピックの手順では、[PHP][install-php] がインストールされていることを前提としています。

> [!NOTE]
> Azure 用 PHP クライアント ライブラリを使用するには、PHP 5.5 以上が必要です。
> 
> 

## Microsoft Azure 用 PHP クライアント ライブラリ
Azure 用 PHP クライアント ライブラリには、任意のオペレーティング システムからデータ管理サービスやクラウド サービスなどの Azure の機能にアクセスするためのインターフェイスが用意されています。これらのライブラリは、Composer からインストールできます。

Azure 用 PHP クライアント ライブラリを使用する方法については、「[BLOB サービスの使用方法][blob-service]」、「[テーブル サービスの使用方法][table-service]」、および「[キュー サービスの使用方法][queue-service]」を参照してください。

### Composer 経由でインストールする
1. [Git をインストールします][install-git]。

    > [AZURE.NOTE] Windows では、Git 実行可能ファイルを PATH 環境変数に追加する必要があります。

1. プロジェクトのルートに **composer.json** という名前のファイルを作成して、次のコードを追加します。
   
        {
            "require": {
                "microsoft/windowsazure": "^0.4"
            }
        }
2. **[composer.phar][composer-phar]** をプロジェクトのルートにダウンロードします。
3. コマンド プロンプトを開き、次のコマンドをプロジェクトのルートで実行します。
   
        php composer.phar install

## Azure PowerShell と Azure エミュレーター
Azure PowerShell は、Cloud Services や Virtual Machines などの Azure サービスをデプロイおよび管理するための PowerShell コマンドレットのセットです。Azure エミュレーターは、アプリケーションをローカルでテストできるようにするためのクラウド サービスおよびデータ管理サービスのエミュレーターです。これらのコンポーネントは、Windows のみでサポートされています。

Azure PowerShell と Azure エミュレーターは、[Microsoft Web プラットフォーム インストーラー][download-wpi]を使用してインストールすることをお勧めします。PHP、SQL Server、Microsoft Drivers for SQL Server for PHP、WebMatrix など、他の開発用コンポーネントをインストールすることもできます。

Azure PowerShell の使用方法については、「[Azure PowerShell の使用方法][powershell-tools]」を参照してください。

## Azure CLI
Azure CLI は、Azure Websites や Azure Virtual Machines などの Azure サービスをデプロイおよび管理するためのコマンド ライン ツールのセットです。Azure CLI のインストール方法については、「[Azure CLI のインストール](xplat-cli-install.md)」を参照してください。

## 次のステップ
詳細については、[PHP デベロッパー センター](/develop/php/)を参照してください。

[install-php]: http://www.php.net/manual/en/install.php
[composer-github]: https://github.com/composer/composer
[composer-phar]: http://getcomposer.org/composer.phar
[nodejs-org]: http://nodejs.org/
[install-node-linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
[download-wpi]: http://go.microsoft.com/fwlink/?LinkId=253447
[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[blob-service]: http://go.microsoft.com/fwlink/?LinkId=252714
[table-service]: http://go.microsoft.com/fwlink/?LinkId=252715
[queue-service]: http://go.microsoft.com/fwlink/?LinkId=252716
[azure cli]: http://go.microsoft.com/fwlink/?LinkId=252717
[powershell-tools]: http://go.microsoft.com/fwlink/?LinkId=252718
[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git

<!---HONumber=AcomDC_0601_2016-->