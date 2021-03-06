---
title: Azure CDN の既定の HTTP 動作を規則エンジンを使用してオーバーライドする | Microsoft Docs
description: 規則エンジンでは、Azure CDN における HTTP 要求の処理方法 (特定種類のコンテンツの配信のブロックなど) のカスタマイズのほか、キャッシュ ポリシーの定義、HTTP ヘッダーの変更などを行えます。
services: cdn
documentationcenter: ''
author: camsoper
manager: erikre
editor: ''

ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/28/2016
ms.author: casoper

---
# 規則エンジンを使用して既定の HTTP 動作をオーバーライドする
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## 概要
規則エンジンでは、特定の種類のコンテンツの配信のブロック、キャッシュ ポリシーの定義、HTTP ヘッダーの変更など、HTTP 要求の処理方法をカスタマイズできます。このチュートリアルでは、CDN 資産のキャッシュ動作を変更するルールの作成について説明します。また、「[関連項目](#see-also)」セクションではビデオ コンテンツもご覧いただけます。

## チュートリアル
1. [CDN プロファイル] ブレードで、**[管理]** をクリックします。
   
    ![[CDN プロファイル] ブレードの [管理] ボタン](./media/cdn-rules-engine/cdn-manage-btn.png)
   
    CDN 管理ポータルが開きます。
2. **[HTTP ラージ]** タブ、**[規則エンジン]** の順にクリックします。
   
    新しい規則のオプションが表示されます。
   
    ![CDN の新しい規則オプション](./media/cdn-rules-engine/cdn-new-rule.png)
   
   > [!IMPORTANT]
   > 複数の規則が表示される順序は、これらの規則の処理方法に影響します。前の規則で指定したアクションは、後続の規則でオーバーライドされます。
   > 
   > 
3. **[名前 / 説明]** テキストボックスに名前を入力します。
4. 規則を適用する要求の種類を指定します。既定では、一致条件 **[常に]** が選択されています。このチュートリアルでは **[常に]** を使用するため、選択されたままにしておきます。
   
    ![CDN の一致条件](./media/cdn-rules-engine/cdn-request-type.png)
   
   > [!TIP]
   > ドロップダウンには多くの種類の一致条件が用意されています。一致条件の左側の青い情報アイコンをクリックすると、現在選択している状態の詳細が表示されます。
   > 
   > 一致条件の詳しい完全な一覧については、「[CDN ルール エンジンの一致条件と機能詳細](https://msdn.microsoft.com/library/mt757336.aspx#Anchor_0)」を参照してください。
   > 
   > 
5. **[機能]** の横にある **[+]** をクリックして、新しい機能を追加します。左側のドロップダウンで、**[内部の最長期間を強制する]** を選択します。表示されるテキスト ボックスに、「**300**」と入力します。他の既定値はそのまま使用します。
   
   ![CDN の機能](./media/cdn-rules-engine/cdn-new-feature.png)
   
   > [!NOTE]
   > 一致条件と同様に、新しい機能の左側にある青い情報アイコンをクリックすると、その機能に関する詳細が表示されます。**[内部の最長期間を強制する]** では、**Cache-Control** および **Expires** ヘッダーをオーバーライドして、CDN エッジ ノードが配信元からの資産を更新するタイミングを制御します。今回の 300 秒のサンプルでは、その配信元から資産を更新する前に、CDN エッジ ノードは 5 分間、資産をキャッシュします。
   > 
   > 機能の詳しい完全な一覧については、「[CDN ルール エンジンの一致条件と機能詳細](https://msdn.microsoft.com/library/mt757336.aspx#Anchor_1)」を参照してください。
   > 
   > 
6. **[追加]** をクリックして、新しい規則を保存します。次に、新しい規則は承認を待機します。承認されると、ステータスが**保留中の XML** から**アクティブな XML** に変わります。
   
   > [!IMPORTANT]
   > 規則の変更が CDN に反映されるまでに最大 90 分かかる場合があります。
   > 
   > 

## 関連項目
* [Azure Friday: Azure CDN に新しく追加された強力な Premium 機能](https://azure.microsoft.com/documentation/videos/azure-cdns-powerful-new-premium-features/) (ビデオ)
* [ルール エンジンの一致条件と機能詳細](https://msdn.microsoft.com/library/mt757336.aspx)

<!---HONumber=AcomDC_0803_2016-->