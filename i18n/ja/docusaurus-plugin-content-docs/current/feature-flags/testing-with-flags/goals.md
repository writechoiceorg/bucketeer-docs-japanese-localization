---
title: ゴール
# sidebar_position: 
slug: /feature-flags/testing-with-flags/goals
description: Describes how to create goals, what to consider when creating them, and how to use them.
tags: ['goals']
---

import CenteredImg from '@site/src/components/centered-img/CenteredImg';

Bucketeerシステムのゴール機能は、エクスペリメントの整理および分析を容易にするために設計されています。ゴールは、特定の行動に基づいてエンドユーザーのデータをグループ化するための手段として機能します。ただし、ゴールを使用する主な目的は、ユーザージャーニーを追跡することです。ゴールを定義することで、エクスペリメント内のフィーチャーフラグによって影響を受けるユーザーの行動を効果的に測定できます。ゴールは、エクスペリメントの作成と評価の両方において重要な役割を果たします。

## ゴールの作成

Bucketeerでゴールを作成するには、まずダッシュボードにアクセスします。ダッシュボードにアクセス後、**ゴール**タブに移動します。このタブには、現在の環境内のすべての既存のゴールの概要が表示されます。

特定のゴールを見つける必要がある場合は、Bucketeerに用意されている検索機能を使用して、名前、ID、または説明に基づいてゴールを検索することができます。また、新しいゴールを作成したい場合は、**+ 追加**ボタンをクリックすることができます。

ゴールを作成するには、**+ 追加**を選択すると表示される作成パネルの項目に入力する必要があります。ゴールのID、名前、および説明を入力することが不可欠です。ゴールのIDは、アプリケーションでエンドユーザーの進捗状況を報告するために使用します。以下の画像は、作成パネルの例を示しています。

<CenteredImg
  imgURL="img/feature-flags/goals/create-goal-v2.png"
  alt="create goal example"
  wSize="400px"
  borderWidth="1px"
/>

:::tip 説明
Bucketeerチームは、将来のエクスペリメント分析においてデータをよりよく理解できるように、わかりやすい名前を使用し、ゴールの説明を徹底的に文書化することを強く推奨しています。
:::


### ゴール例

Bucketeerでは、システムに関連するあらゆるシナリオに対してゴールを定義することができます。可能なシナリオをよりよく理解するために、以下のサブセクションでは、A/Bテストと多変量テストに関連するゴールの説明をご紹介します。

#### A/Bテストの例

- **ゴールID**: ab_test_contact_button_position
- **ゴール名**: お問い合わせボタンの位置を比較（上部と下部）
**ゴール説明**: ゴールは、ウェブページ上のお問い合わせボタンの2つの異なる位置（上部と下部）のパフォーマンスとユーザーエンゲージメントを評価することです。両方のバリエーションでお問い合わせボタンとのユーザーのやり取りを追跡することで、どの位置が高いクリック率、コンバージョン率、および全体的なユーザー満足度をもたらすかを決定できます。結果は、ユーザーエクスペリエンスの向上と望ましいビジネス成果を実現するために、お問い合わせボタンの最適な配置を知らせる洞察を提供します。

#### 多変量の例

- **ゴールID**: multivariable_test_cta_performance
- **ゴール名**: CTAパフォーマンスの評価（4つの定義されたCTA）
- **ゴール説明**: ゴールは、機能内の4つの異なるコールトゥアクション（CTA）バリエーションの効果と影響を評価することです。このエクスペリメントでは、各CTAバリエーションのユーザーインタラクション、クリック率、およびコンバージョン率を測定します。データを分析することで、ユーザーエンゲージメント、コンバージョン、および望ましいアクションに関してパフォーマンスの高いCTAデザイン、文言、または視覚要素を特定できます。結果は、CTA戦略を最適化し、全体的なユーザーエンゲージメントとコンバージョン率を高めるための貴重な洞察を提供します。

## ゴールの使い方

ゴールのデータを関連付けるには、エンドユーザージャーニーを報告する必要があります。これを実現するには、Bucketeer SDKで提供されているレポートソリューションを使用します。ゴールの達成を報告するには、SDKの`track`関数を使用します。この関数を使用すると、クライアント（エンドユーザー）が定義したジャーニーのゴールに到達したときに登録することができます。次のコードブロックは、Javascriptで`track`関数を使用してクライアントによってゴールの達成を登録する例を示しています。

```js showLineNumbers
client.track("GOAL_ID");
```

ゴールの追跡に関する詳細については、SDKドキュメントの**顧客イベントの報告**セクションで、使用するプログラミング言語に関連するセクションをご覧ください。

