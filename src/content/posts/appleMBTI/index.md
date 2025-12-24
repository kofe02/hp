---
title: りんごのMBTI診断を作った話
published: 2025-12-24
description: 'りんご性格診断をバイブスコーディングで作りました'
image: ''
tags: [AI, app, MMA-Advent-Calendar-2025, Advent-Calendar]
category: 'コーディング'
draft: false 
lang: 'ja'
---

<iframe src="https://adventar.org/calendars/11429/embed" width="620" height="362" frameborder="0" loading="lazy"></iframe>

>この記事は[MMA Advent Calendar 2025 - Adventar](https://adventar.org/calendars/11429) の13日目の記事です。

## はじめに

こんにちは、MMAに所属しているkofeです。この記事はAIに手伝ってもらいつつ書いています。

今回は「[りんご性格診断サイト](https://www.ringo-lab.com)」がどうやって生まれたのか、その開発の流れをご紹介します。

このサイトは、あなたの性格を16種類のりんごに例えて診断するというちょっと変わった性格診断サイトです。MBTI診断のロジックを~~パクリ~~応用しながら、その人にあったりんごの品種を提案します。

## なぜりんご診断なのか

そもそもなぜりんごで性格診断？って話なのですが、とある事情で「[りんごの会](https://x.com/ut_keio_apple)」という色んな大学の学生で構成されたサークルに所属していて、青森のりんご農家さんと交流したり、学祭でりんごに関連したものを販売したりしていることから始まりました。

りんごはめちゃくちゃ種類が多く、ふじ、王林、つがる...有名なものから、祝、御所川原、はつ恋ぐりんみたいな知る人ぞ知る品種まであります。それぞれに個性があって、甘さ、酸っぱさ、食感、旬の時期が全然違います。

この種類があれば流行りの性格診断に応用できるんじゃないか？と思い立ち、今回のプロジェクトが始まりました。

## 設計思想

設計は基本的にサークルのメンバーが行ってくれ、私は主に実装担当でした。以下、設計のポイントを紹介します。この設計がしっかりしており、実装がスムーズに進みました。

### 診断ロジックの構築

まずは診断の軸を決めました。りんごの特性と性格特性を組み合わせて、4つの軸を設定：

1. **Color（色）**: Red（赤） ⇔ Green（緑/黄）
   - Red: 情熱的、明るい、外向的、好奇心旺盛
   - Green: 冷静、落ち着き、内向的、内的充実

2. **Taste（味）**: Sweet（甘） ⇔ Tart（酸）
   - Sweet: 優しい、理想家、甘えん坊
   - Tart: ストイック、現実的、厳しい

3. **Season（時期）**: Early（早生） ⇔ Late（晩生）
   - Early: せっかち、効率的、計画的、フッ軽
   - Late: マイペース、のんびり、着実、おおらか

4. **Popularity（人気度）**: Famous（有名） ⇔ Unique（レア）
   - Famous: トレンドに敏感、王道、目立つ
   - Unique: 個性的、ニッチ、自分だけの世界

この4軸の組み合わせで、2の4乗＝**16タイプ**に分類！ほぼ完全にMBTI診断のロジックですね。

### りんごの分類

次に、16タイプそれぞれに実在するりんご品種を割り当てました。例えば：

- **RFSE（Red-Famous-Sweet-Early）**: つがる
  - 早生の赤い甘いりんごで、誰もが知る人気品種
- **RFSL（Red-Famous-Sweet-Late）**: ふじ
  - 晩生の赤い甘いりんごで、日本一の生産量を誇る定番
- **GFSL（Green-Famous-Sweet-Late）**: 王林
  - 黄緑色で甘く香り高い、ふじと双璧をなす人気品種
- **RUSL（Red-Unique-Sweet-Late）**: 世界一
  - 規格外の大きさが特徴の珍しい品種

こんな感じで、実際のりんごの特性に基づいて16品種を選定しました。

## 開発フェーズ

### 0. 企画書作成

まずはGoogleDocumentに設計思想と質問案をまとめました。ここでしっかり設計を固めたおかげで、実装がスムーズに進みました。

### 1. 質問項目の作成

診断には12の質問を用意しました：

- R-G軸（色）: 3問
- F-U軸（知名度）: 3問
- S-T軸（味）: 3問
- E-L軸（時期）: 3問

各質問はMBTIと揃えて5段階評価（1〜5）で、それぞれの軸に対応するように設計。例えば：

> **「あなたのイメージカラーはどっちに近い？」**
>  1: 赤系 → 5: 緑系（Color軸）

> **「お腹がペコぺコだ…そんな時、目の前に「食べてはいけない」と書かれたりんごがありました！あなたならどうする？」**
>  1: 少しなら食べても大丈夫 → 5: なるべく食べないようにする（Taste軸）

りんご栽培に関する質問を入れたのは、実際の農作業体験から着想を得たアイデアです。

### ここからバイブスコーディングでごり押しました
とりあえずGoogleDocumentsの企画書をmdでエクスポートして、Copilot君に読ませて大まかな仕組みを作らせました。デザイン系は、Chromeで開いて、DevToolsのElementsタブから、変更箇所をコピーしてきてCopilot君に貼り付けて微調整してもらいました。どうしても言葉で伝えるとうまく調整できないことが多いので、実際のコードを貼り付けるのが早かったです。あと、デザイン案は手書きをそのまま渡しても意外と認識してくれました。OCRすげえ。

### 2. トップページ（index.html）の実装

まずは入り口となるランディングページを作成。

#### デザインコンセプト
- グラデーション背景: `rgb(225, 225, 180) → #feca57 → #ff9999`
- ブラー: `backdrop-filter: blur(20px)`で文字を見やすく
- アニメーション: フェードインやスケール変換で動きのあるUI

```html
<div class="logo">
    <img src="./etc/りんごの会ロゴ/LOGO4-removebg-preview.png" class="logo-image">
</div>
<h1 class="main-title">🍏りんご性格診断🍎</h1>
<p class="subtitle">あなたの性格をりんごで表すと…？</p>
```

「診断を始める」ボタンを押すと、診断ページ（diagnosis.html）へ遷移します。

### 3. 診断ページ（diagnosis.html）の実装

ここがメイン！2000行近いコードになりました。

#### 主要機能

##### 質問表示システム
```javascript
const questions = [
    { text: "あなたのイメージカラーは...", type: "RG" },
    // ... 12問の質問データ
];

function renderQuestions() {
    // 質問をページ分割して表示
    // 回答済み質問をハイライト
    // 進捗バーを更新
}
```

##### スコアリングシステム
```javascript
function calculateResult() {
    let scores = { R: 0, G: 0, S: 0, T: 0, E: 0, L: 0, F: 0, U: 0 };
    
    questions.forEach((q, i) => {
        const value = answers[i];
        const normalizedValue = (value - 3) / 2; // -1 〜 +1に正規化
        // 各軸にスコアを加算
    });
    
    // 各軸で高い方を選択して4文字のタイプコードを生成
    const type = color + popularity + taste + season;
    return type; // 例: "RFSE"
}
```

##### 結果表示
診断結果画面では：
- りんごの画像と名前
- タイプコード（例: RFSE）
- 性格の説明
- タイプ表示

```javascript
// 鉄アレイ型UI: 4つの軸を視覚的に表現
const axes = [['R', 'G'], ['F', 'U'], ['S', 'T'], ['E', 'L']];
// ユーザーのタイプをハイライト表示
```

スクロールに応じて色付くアニメーションも実装。灰色からオレンジのグラデーションへ変化します。

##### シェア機能
```javascript
document.getElementById('shareTwitter').onclick = () => {
    const text = `🍏あなたの性格をりんごで表すと…？🍎
私は【${appleType.name}(${result.type})】タイプ！🍎
${appleType.summary}`;
    // Twitterシェア
};
```

Twitter、LINE、Threadsへのシェアボタンを設置。バズらせたい気持ちを込めて...！

### 4. りんご図鑑（zukan.html）の実装

診断とは別に、16種類のりんご全てを紹介する図鑑ページも作成。

- カード型レイアウトで16品種を一覧表示
- 各カードには画像、名前、タイプコード、特徴説明
- ホバーエフェクトで動きをつけて、見ていて楽しいUIに

### 5. Google Analytics連携

ユーザーの行動を分析するため、GA4を導入：

```javascript
// 診断開始イベント
gtag('event', 'diagnosis_start', {
    question_total: 12
});

// 診断完了イベント
gtag('event', 'diagnosis_complete', {
    apple_type: result.type,
    apple_name: appleType.name,
    value: 1
});

// シェアイベント
gtag('event', 'share', {
    method: 'Twitter',
    apple_type: result.type
});
```

どのりんごタイプが多いのか、シェア率はどうかなど、データを見るのが楽しみです。

### 6. SEO対策

せっかく作ったので、検索で見つけてもらえるように対策：

#### OGP（Open Graph Protocol）設定
```html
<meta property="og:title" content="りんご性格診断 - あなたはどのりんごタイプ?">
<meta property="og:description" content="12の質問であなたの性格をりんごに例える診断テスト。">
<meta property="og:image" content="https://www.ringo-lab.com/src/image.png">
```

#### Twitter Card
```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@ut_keio_apple">
```

#### sitemap.xmlとrobots.txt
検索エンジンがサイトをクロールしやすいように設定ファイルを配置。

### 7. デプロイ

GitHub Pagesで公開！独自ドメイン `ringo-lab.com` を取得して、`CNAME`ファイルで設定しました。

```
www.ringo-lab.com
```

## 苦労したポイント

### 1. 質問文の調整

「これってどの軸に対応する質問だっけ？」となることが多発。質問文がわかりやすく、かつ各軸をバランスよく測定できるように、何度も見直しました。

### 2. レスポンシブデザイン
スマホでもPCでも快適に使えるように、メディアクエリで細かく調整：

```css
@media (max-width: 768px) {
    .container {
        padding: 20px;
    }
    .logo-image {
        height: 150px;
    }
    /* モバイル用の調整 */
}
```

特にタイプ表示は、画面サイズによってレイアウトが崩れやすく、調整に時間がかかりました。

### 3. パフォーマンス最適化

りんごの画像を16種類も読み込むので、ページが重くなりがち。画像を圧縮したり、遅延読み込みを検討したりしました。

## こだわりポイント（鬼の壁打ちなど）

### 1. りんごの実写画像

各りんご品種の画像は、りんご大学さんから許可をいただいて載せたものになります。本物のりんごを使うことで、リアリティを出しました。いつかは自分たちで撮ったものに差し替えたいですね。

### 2. タイプ表示

結果画面のタイプ表示、これもこだわりで、4つの軸を視覚的に表現する方法として、左右に重りがあるダンベル（鉄アレイ）のような形を採用しました。なかなかいい感じにできなくて辛かった、、、

```
R ━━━━━━━━━━ G
F ━━━━━━━━━━ U
S ━━━━━━━━━━ T
E ━━━━━━━━━━ L
```

最終的にはユーザーのタイプに該当する側が強調表示され、スクロールに応じてアニメーションで色付く仕様に。自分のタイプが一目でわかる、直感的なUIになりました。

### 3. りんごらしい配色

サイト全体の配色は、メンバーの言うとおりに：
- 暖かいグラデーション背景（黄→オレンジ→赤）
- りんごの赤（#ff6b6b）とりんごの黄色（#feca57）を基調
- 結果画面では、タイプに応じて色を変化させる

デザインセンスは皆無なので、メンバーの意見がありがたかったです。

### 4. シェアしたくなる仕掛け

診断結果をシェアするとき、自動的にハッシュタグとアカウントが入るように設定：

```javascript
const text = `🍏あなたの性格をりんごで表すと…？🍎
私は【${appleType.name}(${result.type})】タイプ！🍎
${appleType.summary}
@ut_keio_apple
#りんご診断 #りんごの会`;
```
これでSNSでの拡散を狙います！

### 5. フォームの選択部分をりんごの形に
CSSでセレクトボックスの部分をりんごの形にカスタマイズしました：

```CSS
.apple-body {
    width: 100%;
    height: 100%;
    background: white;
    border: 3px solid #dfe6e9;
    border-radius: 50% 50% 40% 40% / 30% 30% 80% 80%;
    position: relative;
    transition: all 0.3s ease;
    overflow: hidden;
}
・・・
``` 
選択するとアニメーションが入り、りんごが少し揺れるような動きをつけました。

## 技術スタック

- **HTML/CSS/JavaScript**: 純粋なバニラJSで実装
- **Google Analytics 4**: アクセス解析
- **GitHub Pages**: ホスティング
- **独自ドメイン**: ringo-lab.com

フレームワークやライブラリは使わず、シンプルに作りました。ですが、それぞれ一ファイルのhtmlとなってしまい、めちゃくちゃ読みづらいです。。。

## おわりに

「りんご性格診断サイト」、いかがでしたか？

りんごの魅力と性格診断を組み合わせた、ちょっと変わったプロジェクトでしたが、作っていてとても楽しかったです。実際のりんご農家さんとの交流から生まれたアイデアが、こうして形になるのは嬉しいですね。

もしまだ診断していない方がいたら、ぜひやってみてください！
👉 [https://www.ringo-lab.com/](https://www.ringo-lab.com/)

あなたはどのりんごタイプでしょうか？私は **王林（GFSL）** タイプでした。マイペースで優しい、王道の性格らしいです。当たってる...かも？

それでは、良いりんごライフを！🍎🍏

## 参考リンク

- [りんご性格診断](https://www.ringo-lab.com/)
- [りんご図鑑](https://www.ringo-lab.com/zukan.html)
- [りんごの会 Twitter](https://twitter.com/ut_keio_apple)
