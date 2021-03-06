---
title: 監査ログを使用する方法 | Microsoft Docs
description: Azure Privileged Identity Management 拡張機能で監査ログを使用する方法について説明します。
services: active-directory
documentationcenter: ''
author: kgremban
manager: femila
editor: ''

ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/22/2016
ms.author: kgremban

---
# Azure AD Privileged Identity Management で監査ログを使用する方法
Privileged Identity Management (PIM) の監査ログを使用すると、特定の期間におけるすべてのユーザー割り当てとアクティブ化を確認できます。管理者、エンド ユーザー、同期アクティビティを含むテナントのアクティビティの完全な監査履歴を確認するには、[Azure Active Directory のアクセスおよび使用状況レポート](active-directory-view-access-usage-reports.md)を使用してください。

## 監査ログへの移動
[Azure Portal](https://portal.azure.com) のダッシュボードで、**Azure AD Privileged Identity Management** アプリを選択します。次に、PIM ダッシュボードで**[特権ロールの管理]**、**[監査履歴]**の順にクリックして、監査ログにアクセスします。

## 監査ログのグラフ
監査ログを使用すると、合計アクティブ化数、1 日あたりの最大アクティブ化数、1 日あたりの平均アクティブ化数を折れ線グラフで表示できます。監査履歴に複数のロールがある場合は、ロールによってデータをフィルターすることもできます。

**[時間]**、**[アクション]**、および **[ロール]** の各ボタンを使用して、ログを並べ替えることができます。

## 監査ログの一覧
監査ログの一覧の列は次のとおりです。

* **要求元** - ロールのアクティブ化または変更を要求したユーザー。値が "Azure システム" の場合は、Azure 監査ログで詳細を確認できます。
* **ユーザー** - ロールをアクティブ化しているユーザー、またはロールに割り当てられているユーザー。
* **ロール** - ユーザーによって割り当てられたかアクティブ化されたロール。
* **アクション** - 要求元によって実行されたアクション。これには、割り当て、割り当て解除、アクティブ化、非アクティブ化などがあります。
* **時間** - アクションが発生した時刻。
* **理由** - アクティブ化の間に理由フィールドに入力されたテキスト。
* **有効期限** - ロールのアクティブ化のみに関連します。

## 監査ログのフィルター処理
**[フィルター]** をクリックすると、監査ログに表示される情報を絞り込むことができます。**[グラフ パラメーターの更新]** ブレードが表示されます。

フィルターを設定した後、**[更新]** をクリックすると、ログのデータがフィルター処理されます。データがすぐに表示されない場合は、ページを更新してください。

### 日付範囲を変更する
**[今日]**、**[過去 1 週間]**、**[過去 1 か月]**、**[カスタム]** のいずれかのボタンを使用して、監査ログの時間範囲を変更することができます。

**[カスタム]** をクリックすると、**[開始]** および **[終了]** 日付フィールドが表示され、ログの日付範囲を指定できます。MM/DD/YYYY の形式で日付を入力するか、**カレンダー** アイコンをクリックしてカレンダーから日付を選択できます。

### ログに含まれるロールを変更する
ロールをログに含めたりログから除外したりするには、各ロールの横にある **[ロール]** チェックボックスをオンまたはオフにします。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 次のステップ
[!INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!---HONumber=AcomDC_0928_2016-->