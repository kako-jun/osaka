# GEMINI.md: おおさかけんぽう 開発計画書 (改訂版)

## 1. プロジェクト概要

### 1.1. プロジェクト名
おおさかけんぽう

### 1.2. コンセプト
難解な法律の条文を、親しみやすい大阪弁に翻訳して解説するWebサイト。法律への心理的なハードルを下げ、多くの人が気軽に法知識に触れられるようにすることを目的とする。

### 1.3. ターゲットユーザー
- 法律に苦手意識を持つ一般の人々
- 法律の勉強を始めた学生
- 大阪弁が好きな人

### 1.4. デザインコンセプト
- 『あずまんが大王』のキャラクター「春日歩（大阪さん）」をイメージした、温かみのある親しみやすいデザイン。
- モバイルファーストなUI/UX。

---

## 2. 要件定義

### 2.1. 機能要件
- **法律/条例の表示:**
    - 国の法律および全国の地方自治体の条例の原文を表示できる。
    - 原文を大阪弁に翻訳して表示できる。翻訳のトーンは「春日歩が喋っているような」親しみやすい口調とする。
    - 原文と大阪弁訳を簡単に切り替えられるUIを提供する。
- **ナビゲーション/検索機能:**
    - 法律名やキーワードで条文を検索できる。
    - **日本地図UI:** 都道府県を選択し、その自治体の条例を検索できる。
    - **【NEW】 タイムマシンUI:** 大日本帝国憲法など、過去の歴史的な法律へ楽しくアクセスできる。
    - **【NEW】 ワールドツアーUI:** ドイツ法など、他国の法律へ楽しくアクセスできる。UIのテーマは「宇宙」などを検討。
- **比較・分析機能:**
    - **改正履歴:** 現行法の改正履歴（diff）を確認できる。
    - **【NEW】 歴史法比較:** 歴史的な法律（例: 大日本帝国憲法）と現行法を並べて比較できる。
    - **【NEW】 国際法比較:** 他国の法律（例: ドイツ基本法）と日本の法律を比較できる。
    - **【NEW】 AIによる要約・解説:** 法律間の思想的な違いや歴史的背景をAIが要約して解説する。
    - **【NEW】 概念マッピング:** ある条文について、他の法体系に相当する概念があるか否かを表示する。「そもそも概念がない！」といったケースは、面白く目立たせるUIで表現する。
- **共有機能:**
    - 各条文にユニークなURL（パーマリンク）を持たせる。
    - 気に入った条文（原文または大阪弁訳）を画像として生成し、SNSで共有できる。

### 2.2. 非機能要件
- **デザイン:**
    - **カラー:** 柔らかい赤系（例: `#E94E77`）をベースに、白、クリーム、茶色をアクセントとして使用する。
    - **フォント:** 原文は標準的なフォント、大阪弁訳は漫画の吹き出しのような手書き風・丸ゴシック系のフリーフォントを使用し、切り替え可能にする。
- **パフォーマンス:**
    - 主要な法律ページは静的サイト生成（SSG）により高速に表示する。
- **アクセシビリティ:**
    - モバイルデバイスでの閲覧に最適化されたレスポンシブデザイン。

---

## 3. 仕様・設計

### 3.1. 技術スタック
- **フレームワーク:** Next.js (React)
- **言語:** TypeScript, Python (データ処理・翻訳スクリプト用)
- **翻訳エンジン:** 大規模言語モデル (LLM) のAPIを利用
- **データストア:** 静的ファイル (Markdown or JSON)
- **インフラ:** 未定 (Vercel, AWS Amplifyなどが候補)

### 3.2. アーキテクチャ
- **レンダリング:**
    - **SSG (静的サイト生成):** アクセス頻度の高い法律（憲法、民法など）や、一度生成すれば更新頻度の低いページに適用。ビルド時にHTMLを生成しておくことで、CDNから高速配信する。
    - **SSR (サーバーサイドレンダリング):** 検索結果など、リクエスト毎に内容が変わる動的なページに適用。
- **データ管理:**
    - **データソース:** 国の法律、地方条例に加え、比較対象となる歴史法、外国法も収集対象とする。
    - **データ形式:** 取得した法律・条例データは、条文ごとにMarkdown (`.md`) またはJSON形式のファイルとして整形・保存する。
    - **翻訳・分析プロセス:**
        1. 原文の静的ファイル群を用意する。
        2. LLM APIを利用したバッチ処理スクリプトで「大阪弁翻訳」および「法律間の差異の要約」を事前に行う。
        3. 結果を静的ファイルとして保存する。
    - **ファイル構造 (例):** 国や時代でディレクトリを分け、管理しやすくする。
      ```
      /laws/
      ├── jp/ (日本・現行)
      │   ├── constitution/ (日本国憲法)
      │   │   ├── 1.md
      │   │   ├── 1.osaka.md
      │   │   └── ...
      │   └── civil_code/ (民法)
      ├── jp_empire/ (大日本帝国)
      │   └── constitution/ (大日本帝国憲法)
      │       ├── 1.md
      │       └── 1.osaka.md
      └── de/ (ドイツ)
          └── basic_law/ (基本法)
              ├── 1.md
              └── 1.osaka.md
      ```

### 3.3. URL構造
- 条文ごとに分かりやすいURLを設計する。
- 例: `/law/憲法/1` `/law/民法/709`

---

## 4. 実装の段取り

### フェーズ1: MVP（Minimum Viable Product）開発
**目標:** コア機能である「法律の大阪弁翻訳」を体験できる最小限のプロダクトを構築する。
1.  **環境構築:** Next.jsプロジェクトをセットアップし、基本的なレイアウトとデザイン（色、フォント）を適用する。
2.  **データ準備:** 対象を「日本国憲法」のみに絞り、全条文のMarkdownファイルを手動で作成する。
3.  **翻訳スクリプト:** LLM APIを呼び出すPythonスクリプトを作成し、憲法の全条文を大阪弁に翻訳して別ファイルとして保存する。
4.  **静的ページ生成:** `getStaticProps` を利用し、憲法の各条文ページを生成する。URLは `/law/憲法/[articleId]` とする。
5.  **UI実装:** ページ上で「原文」と「大阪弁訳」を切り替えるタブまたはボタンを実装する。

### フェーズ2: 機能拡張とデータ拡充
**目標:** 対象とする法律を増やし、ユーザー体験を向上させる機能を追加する。
1.  **データ収集自動化:** e-Gov法令検索から主要な法律データを自動で取得・整形するクローラーを開発する。
2.  **画像生成機能:** 条文を画像化するAPIを実装する。デザインテンプレートを数種類用意する。
3.  **SNS共有機能:** 生成した画像をSNSに共有するボタンを実装する。
4.  **検索機能:** まずはサイト内コンテンツ（静的ファイル）に対するキーワード検索機能を実装する。

### フェーズ3: 比較・分析機能の実装
**目標:** 法律を多角的に比較できる「楽しいdiff」機能を実装する。
1.  **データ拡充:** 比較対象として「大日本帝国憲法」のデータを追加する。
2.  **比較UI実装:** 2つの法律（例: 現行憲法 vs 帝国憲法）を並べて表示し、差分をハイライトするUIを実装する。
3.  **AI要約機能:** 法律間の思想的な違いをLLMで要約し、表示する機能を実装する（バッチ処理）。
4.  **概念マッピング表示:** 条文ごとに、比較対象に相当概念がない場合にそれを表示するUIを実装する。

### フェーズ4: グローバル・ヒストリカル対応
**目標:** ナビゲーションを拡張し、条例や国際法へも対応範囲を広げる。
1.  **ナビゲーションUI開発:** 「タイムマシン」「ワールドツアー」をテーマにしたナビゲーションUIを実装する。
2.  **国際法データ追加:** 比較対象として「ドイツ基本法」などのデータを追加し、国際法比較機能を実装する。
3.  **条例対応:** 日本地図UIと全国の条例データを連携させる。

---

## 確認事項

- **データソースについて:** 地方自治体の条例について、ブレインストーミングでは「同志社大学の条例DB」を参考にするとありましたが、外部データベースの利用規約（API連携、スクレイピングの可否など）を確認する必要があります。まずは国の法律を対象として開発を進め、条例対応はフェーズ3で実現可能性を再検討する、という進め方でよろしいでしょうか？
