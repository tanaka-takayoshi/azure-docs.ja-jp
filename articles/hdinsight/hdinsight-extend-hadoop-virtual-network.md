---
title: Virtual Network を使用した HDInsight の拡張 | Microsoft Docs
description: Azure Virtual Network を使用して HDInsight を他のクラウド リソースまたはデータ センター内のリソースに接続する方法について説明します。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun

ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 09/13/2016
ms.author: larryfr

---
# Azure Virtual Network を使用した HDInsight 機能の拡張
Azure Virtual Network では、Hadoop ソリューションを拡張して、SQL Server などのオンプレミスのリソースに組み込むことや、クラウドのリソース間にセキュリティで保護されたプライベート ネットワークを作成することができます。

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell-and-cli.md)]

## <a id="whatis"></a>Azure Virtual Network とは
[Azure Virtual Network](https://azure.microsoft.com/documentation/services/virtual-network/) によって、ソリューションに必要なリソースを含む、セキュリティで保護された永続的なネットワークを作成できます。仮想ネットワークでは、次のことが可能になります。

* プライベート ネットワーク (クラウドのみ) 内でのクラウド リソース間の接続
  
    ![クラウドのみの構成の図](media/hdinsight-extend-hadoop-virtual-network/cloud-only.png)
  
    Virtual Network を使用して Azure サービスと Azure HDInsight を連携させることで、以下のシナリオが実現できるようになります。
  
  * Azure Web サイトから、または Azure の仮想マシンで実行しているサービスから、**HDInsight サービスまたはジョブを呼び出す**。
  * HDInsight と Azure SQL Database、または SQL Server や仮想マシンで実行されている他のデータ ストレージ ソリューションとの間で、**データを直接転送する**。
  * **複数の HDInsight サーバーを結合して単一のソリューションにする。**たとえば、HDInsight Storm サーバーを使用して受信データを処理し、処理したデータを HDInsight HBase サーバーに格納します。今後 MapReduce を使用して分析するために、生データを HDInsight Hadoop サーバーに格納することもできます。
* 仮想プライベート ネットワーク (VPN) を使用したローカル データセンター ネットワークへのクラウド リソースの接続 (サイト間またはポイント対サイト)。
  
    サイト間構成では、ハードウェア VPN を使用するか、ルーティングとリモート アクセス サービスを使用して、データセンターから複数のリソースを Azure Virtual Network に接続できます。
  
    ![サイト間構成の図](media/hdinsight-extend-hadoop-virtual-network/site-to-site.png)
  
    ポイント対サイト構成では、ソフトウェア VPN を使用して、特定のリソースを Azure Virtual Network に接続できます。
  
    ![ポイント対サイト構成の図](media/hdinsight-extend-hadoop-virtual-network/point-to-site.png)
  
    Virtual Network を使用してクラウドとデータ センターをリンクすると、クラウドのみの構成と同様のシナリオを実現できます。ただし、クラウド内のリソースの操作に制限されるのではなく、データ センターのリソースも使用できます。
  
  * HDInsight とデータ センター間で**データを直接転送する**。たとえば、Sqoop を使用して SQL Server に、または SQL Server からデータを転送したり、基幹業務 (LOB) アプリケーションで生成されたデータを読み取ったりします。
  * **LOB アプリケーションから HDInsight サービスを呼び出す。**たとえば、HBase Java API を使用して HDInsight HBase クラスターにデータを格納したり、クラスターからデータを取得したりします。

Virtual Network の機能、利点の詳細については、「[Azure 仮想ネットワークの概要](../virtual-network/virtual-networks-overview.md)」を参照してください。

> [!NOTE]
> HDInsight クラスターをプロビジョニングする前に、Azure Virtual Network を作成する必要があります。詳細については、「[仮想ネットワークの構成タスク](https://azure.microsoft.com/documentation/services/virtual-network/)」を参照してください。
> 
> 

## Virtual Network に関する要件
> [!IMPORTANT]
> Virtual Network で HDInsight クラスターを作成するには、このセクションで説明する、特定の Virtual Network 構成が必要です。
> 
> 

### 場所ベースの仮想ネットワーク
Azure HDInsight は場所ベースの仮想ネットワークのみをサポートし、アフィニティ グループ ベースの仮想ネットワークは現在取り扱っていません。

### クラシックまたは v2 Virtual Network
Windows ベースのクラスターでは、クラシック Virtual Network が必要であり、Linux ベースのクラスターでは、Azure Resource Manager Virtual Network が必要です。ネットワークの種類が正しくない場合、クラスターの作成には使用できません。

作成を計画しているクラスターでは使用できない Virtual Network 上にリソースがある場合、クラスターで使用できる新しい Virtual Network を作成し、それを互換性のない Virtual Network に接続できます。その後、必要なネットワーク バージョンでクラスターを作成し、2 つは結合されているので別のネットワークのリソースにアクセスが可能となります。従来の Virtual Network と新しい Virtual Network を接続する方法の詳細については、「[従来の Vnet を新しい Vnet に接続する](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)」を参照してください。

### カスタム DNS
仮想ネットワークを作成するとき、そのネットワークにインストールされている Azure サービス (HDInsight) には Azure によって既定の名前解決が行われます。ただしクロス ネットワーク ドメイン名解決など、状況によっては独自のドメイン ネーム システム (DNS) を使用しなければならないこともあります。たとえば、参加している 2 つの仮想ネットワークに置かれているサービス間の通信が挙げられます。HDInsight は、Azure Virtual Network との組み合わせで、Azure の既定の名前解決に加え、カスタム DNS もサポートしています。

Azure Virtual Network で独自の DNS サーバーを使用する方法の詳細については、「[VM とロール インスタンスの名前解決](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-using-your-own-dns-server)」の「**独自 DNS サーバー使用の名前解決**」をご覧ください。

### セキュリティで保護された Virtual Networks
HDInsight サービスは管理されたサービスで、プロビジョニング中や実行中にインターネット アクセスが必要です。これは、クラスターの正常性を監視する、クラスター リソースのフェールオーバーを開始する、スケーリング操作によりノード数を変更するなどの管理タスクを Azure が監視できるようにすることが目的です。

セキュリティで保護された仮想ネットワークに HDInsight をインストールする必要がある場合は、次の IP アドレスのポート 443 上で着信アクセスを許可する必要があります。これにより、Azure が HDInsight クラスターを管理できるようになります。

* 168\.61.49.99
* 23\.99.5.239
* 168\.61.48.131
* 138\.91.141.162

これらのアドレスのポート 443 上で着信アクセスを許可すると、セキュリティ保護された仮想ネットワークに HDInsight を正常にインストールできます。

> [!IMPORTANT]
> HDInsight では、送信トラフィックの制限をサポートしていません。受信トラフィックのみをサポートしています。HDInsight が含まれているサブネットのネットワーク セキュリティ グループ規則を定義するときは、受信規則だけを使用します。
> 
> 

次の例では、必要なアドレスを許可し、Virtual Network内のサブネットにセキュリティ グループを適用する、新しいネットワーク セキュリティ グループを作成する方法を示します。これらの手順では、HDInsight をインストールする Virtual Network とサブネットを既に作成していることを前提としています。

**Azure PowerShell の使用**

    $vnetName = "Replace with your virtual network name"
    $resourceGroupName = "Replace with the resource group the virtual network is in"
    $subnetName = "Replace with the name of the subnet that HDInsight will be installed into"
    # Get the Virtual Network object
    $vnet = Get-AzureRmVirtualNetwork `
        -Name $vnetName `
        -ResourceGroupName $resourceGroupName
    # Get the region the Virtual network is in.
    $location = $vnet.Location
    # Get the subnet object
    $subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
    # Create a new Network Security Group.
    # And add exemptions for the HDInsight health and management services.
    $nsg = New-AzureRmNetworkSecurityGroup `
        -Name "hdisecure" `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -name "hdirule1" `
            -Description "HDI health and management address 168.61.49.99" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.49.99" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 300 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule2" `
            -Description "HDI health and management 23.99.5.239" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "23.99.5.239" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 301 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule3" `
            -Description "HDI health and management 168.61.48.131" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.48.131" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 302 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule4" `
            -Description "HDI health and management 138.91.141.162" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "138.91.141.162" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 303 `
            -Direction Inbound
    # Set the changes to the security group
    Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
    # Apply the NSG to the subnet
    Set-AzureRmVirtualNetworkSubnetConfig `
        -VirtualNetwork $vnet `
        -Name $subnetName `
        -AddressPrefix $subnet.AddressPrefix `
        -NetworkSecurityGroupId $nsg

**Azure CLI の使用**

1. 次のコマンドを使用して、`hdisecure` という名前の新しいネットワーク セキュリティ グループを作成します。**RESOURCEGROUPNAME** と **LOCATION** を、Azure Virtual Network を含むリソース グループと、そのグループの作成場所 (リージョン) に置き換えます。
   
        azure network nsg create RESOURCEGROUPNAME hdisecure LOCATION
   
    グループが作成されたら、新しいグループに関する情報が表示されます。次のような行を検索し、`/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure` の情報を保存します。これは、後の手順で使用します。
   
        data:    Id                              : /subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure
2. ポート 443 上で Azure HDInsight の正常性と管理サービスからのインバウンド通信を許可する新しいネットワーク セキュリティ グループにルールを追加するには、次を使用します。**RESOURCEGROUPNAME** を、Azure Virtual Network が含まれているリソース グループの名前に置き換えます。
   
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule1 -p "*" -o "*" -u "443" -f "168.61.49.99" -e "VirtualNetwork" -c "Allow" -y 300 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule2 -p "*" -o "*" -u "443" -f "23.99.5.239" -e "VirtualNetwork" -c "Allow" -y 301 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule3 -p "*" -o "*" -u "443" -f "168.61.48.131" -e "VirtualNetwork" -c "Allow" -y 302 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule4 -p "*" -o "*" -u "443" -f "138.91.141.162" -e "VirtualNetwork" -c "Allow" -y 303 -r "Inbound"
3. ルールが作成されたら、次を使用して新しいネットワーク セキュリティ グループをサブネットに適用します。**RESOURCEGROUPNAME** を、Azure Virtual Network が含まれているリソース グループの名前に置き換えます。**VNETNAME** と **SUBNETNAME** を、Azure Virtual Network の名前と HDInsight のインストール時に使用するサブネットの名前に置き換えます。
   
        azure network vnet subnet set RESOURCEGROUPNAME VNETNAME SUBNETNAME -w "/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
   
    このコマンドが完了すると、これらの手順で使用されるサブネット上のセキュリティで保護された Virtual Network に、HDInsight を正常にインストールできます。

> [!IMPORTANT]
> 上記の手順を使用すると、Azure クラウドの HDInsight 正常性および管理サービスへのアクセスのみが開きます。これにより、HDInsight クラスターをサブネットに正常にインストールできますが、既定で Virtual Network 外からの HDInsight クラスターへのアクセスがブロックされます。Virtual Network 外からのアクセスを可能にする場合は、追加のネットワーク セキュリティ グループ規則を追加する必要があります。
> 
> たとえば、インターネットからの SSH アクセスを許可するには、次のようなルールを追加する必要があります。
> 
> * Azure PowerShell - ```Add-AzureRmNetworkSecurityRuleConfig -Name "SSSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 304 -Direction Inbound```
> * Azure CLI - ```azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule4 -p "*" -o "*" -u "22" -f "*" -e "VirtualNetwork" -c "Allow" -y 304 -r "Inbound"```
> 
> 

ネットワーク セキュリティ グループの詳細については、[ネットワーク セキュリティ グループの概要](../virtual-network/virtual-networks-nsg.md)に関する記事を参照してください。Azure Virtual Network におけるルーティングの制御については、[ユーザー定義のルートと IP 転送](../virtual-network/virtual-networks-udr-overview.md)に関する記事を参照してください。

## <a id="tasks"></a>タスクと情報
このセクションでは、仮想ネットワークで HDInsight を使用する場合の一般的なタスクに関する情報と、必要になる可能性がある情報を記載しています。

### FQDN の決定
HDInsight クラスターには、Virtual Network インターフェイスを表す特定の完全修飾ドメイン名 (FQDN) が割り当てられます。これは、仮想ネットワーク上の他のリソースからクラスターに接続するときに使用する必要があるアドレスです。FQDN を確認するには、次の URL を使用して Ambari 管理サービスを照会します。

    https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/services/<servicename>/components/<componentname>

> [!NOTE]
> HDInsight での Ambari の使用の詳細については、「[Ambari API を使用した HDInsight の Hadoop クラスターの監視](hdinsight-monitor-use-ambari-api.md)」を参照してください。
> 
> 

クラスター名およびクラスターで実行するサービスとコンポーネント (YARN リソース マネージャーなど) を指定する必要があります。

> [!NOTE]
> 返されるデータは、コンポーネントに関する多くの情報を含む JavaScript Object Notation (JSON) ドキュメントです。FQDN のみを抽出するには、JSON のパーサーを使用して、`host_components[0].HostRoles.host_name` の値を取得する必要があります。
> 
> 

たとえば、HDInsight Hadoop クラスターから FQDN を取得するには、次のいずれかのメソッドを使用して、YARN リソース マネージャーのデータを取得します。

* [Azure PowerShell](../powershell-install-configure.md)
  
        $ClusterDnsName = <clustername>
        $Username = <cluster admin username>
        $Password = <cluster admin password>
        $DnsSuffix = ".azurehdinsight.net"
        $ClusterFQDN = $ClusterDnsName + $DnsSuffix
  
        $webclient = new-object System.Net.WebClient
        $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)
  
        $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/yarn/        components/resourcemanager"
        $Response = $webclient.DownloadString($Url)
        $JsonObject = $Response | ConvertFrom-Json
        $FQDN = $JsonObject.host_components[0].HostRoles.host_name
        Write-host $FQDN
* [cURL](http://curl.haxx.se/) および [jq](http://stedolan.github.io/jq/)
  
        curl -G -u <username>:<password> https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/services/yarn/components/resourcemanager | jq .host_components[0].HostRoles.host_name

### HBase への接続
Java API を使用して HBase にリモート接続するには、HBase クラスターの ZooKeeper クォーラム アドレスを確認し、これをアプリケーションで指定する必要があります。

ZooKeeper クォーラム アドレスを取得するには、次のいずれかのメソッドを使用して、Ambari 管理サービスに問い合わせます。

* [Azure PowerShell](../powershell-install-configure.md)
  
        $ClusterDnsName = <clustername>
        $Username = <cluster admin username>
        $Password = <cluster admin password>
        $DnsSuffix = ".azurehdinsight.net"
        $ClusterFQDN = $ClusterDnsName + $DnsSuffix
  
        $webclient = new-object System.Net.WebClient
        $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)
  
        $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum"
        $Response = $webclient.DownloadString($Url)
        $JsonObject = $Response | ConvertFrom-Json
        Write-host $JsonObject.items[0].properties.'hbase.zookeeper.quorum'
* [cURL](http://curl.haxx.se/) および [jq](http://stedolan.github.io/jq/)
  
        curl -G -u <username>:<password> "https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum" | jq .items[0].properties[]

> [!NOTE]
> HDInsight での Ambari の使用の詳細については、「[Ambari API を使用した HDInsight の Hadoop クラスターの監視](hdinsight-monitor-use-ambari-api.md)」を参照してください。
> 
> 

クォーラムの情報を取得したら、それをクライアント アプリケーションで使用します。

たとえば、HBase API を使用する Java アプリケーションでは、**hbase-site.xml** ファイルをプロジェクトに追加し、次のようにそのファイル内にクォーラム情報を指定します。

```
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper0.address,zookeeper1.address,zookeeper2.address</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
</configuration>
```

### ネットワーク接続の確認
SQL Server などの一部のサービスでは、着信ネットワーク接続数が制限される場合があります。そのようなサービスでは HDInsight が正常に機能しません。

HDInsight からサービスへのアクセスで問題が発生した場合は、サービスのドキュメントを参照して、ネットワーク アクセスが有効になっていることを確認してください。ネットワーク アクセスを確認するもう 1 つの方法では、同じ仮想ネットワーク上に Azure の仮想マシンを作成し、クライアント ユーティリティを使用して、VM から仮想ネットワーク経由でサービスに接続できることを確認します。

## <a id="nextsteps"></a>次のステップ
次の例は、Azure Virtual Network で HDInsight を使用する方法を示しています。

* [HDInsight での、Storm および HBase を使用したセンサー データの分析](hdinsight-storm-sensor-data-analysis.md) - 仮想ネットワークでの Storm および HBase の構成方法と、Storm から HBase にリモートでデータを書き込む方法を示しています。
* [HDInsight での Hadoop クラスターのプロビジョニング](hdinsight-hadoop-provision-linux-clusters.md) - Azure Virtual Network の使用に関する情報を含めて、Hadoop クラスターのプロビジョニングに関する情報を提供します。
* [HDInsight での Hadoop と Sqoop の使用](hdinsight-use-sqoop-mac-linux.md) - SQL Server と Sqoop を使用した仮想ネットワーク経由のデータ転送に関する情報を提供します。

Azure のかそうネットワークの詳細については、[Azure Virtual Network の概要](../virtual-network/virtual-networks-overview.md)に関するページを参照してください。

<!---HONumber=AcomDC_0914_2016-->