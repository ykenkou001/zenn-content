# CLAUDE.md - zenn-content

このリポジトリは Zenn 記事の正本を管理するリポジトリです。GitHub と Zenn が連携しており、`main` ブランチへの push で記事が自動同期されます。

## プロジェクト概要

- 個人ブログ的な技術記事リポジトリ
- 言語：日本語
- 著者プロフィール：AIエンジニア／データサイエンティスト。Kotlin・Android開発は未経験。Claude Codeで越境的に個人開発をするスタイル。ギター歴約1年（初心者）

## ディレクトリ構成

```
zenn-content/
├── articles/                 # 記事Markdown（ファイル名 = slug）
├── images/<slug>/            # 記事ごとの画像/GIF
├── books/                    # Zenn Book（未使用）
├── package.json              # zenn-cli v0.4.7
└── README.md
```

## 公開済み記事

| Slug | 公開日 | 内容 |
|------|--------|------|
| `clo-logistics-decision-support` | (既公開) | CLO支援アプリ PoC（業務×エンジニア両軸エッセイ）。emoji 🚚, topics: 物流/CLO/mcp/fastapi/aws |
| `fret-interval-android-guitar-app` | 2026-05-09 | FretInterval開発記（Kotlin未経験×Claude Code×CI/CD）。emoji 🎸, topics: claudecode/android/kotlin/jetpackcompose/個人開発。元プロジェクト: `c:/Users/ykenk/repos/FretInterval` |

## ワークフロー

### 新規記事作成

```powershell
npm run new:article    # 雛形生成（slug は手動指定推奨）
```

### ローカルプレビュー

```powershell
npm run preview        # http://localhost:8000 でホットリロード
```

### 画像配置

- `images/<slug>/` に配置
- 記事内では絶対パスで参照: `/images/<slug>/<filename>`
- 推奨サイズ: GIF 1MB以下, PNG 300KB前後
- GIF生成: ffmpeg + palette生成の2パスで `fps=12, scale=480:-2:flags=lanczos`

### 公開

1. frontmatter `published: false` → `true` に変更
2. git add + commit + push origin main
3. Zenn が数分で自動同期

### コミット規約

`docs: <内容>` の1行サマリ。

例：
- `docs: FretIntervalの開発記事を追加（Kotlin未経験×Claude Code×CI/CD）`
- `docs: CLO記事にMCPファースト設計記事への相互リンクを追加`

## 記事執筆ガイドライン（Zennバズ戦略反映）

### 構成原則

- **文字数**: 6,000〜10,000字 を目標（Zennは文字数U字曲線で 3,000-5,000字 が死の谷、6,000字超 or 1,500字以下 が伸びる）
- **冒頭150字以内** にデモGIF・要点・`for me` 宣言を置いて離脱防止
- 冒頭は `:::message` ボックスで PoC共有メッセージから始める（CLO/FretInterval記事と統一）
- **続編予告**を末尾に置いて二次拡散を仕込む

### タイトル3要素

必ず以下の3つを満たす：

1. **対象読者の明確化**（誰向けか）
2. **何ができるか**（内容の要約）
3. **面白さ・独自性**（数字・意外性・越境性）

例：「Kotlin未経験のAIエンジニアが、Claude Codeで3日でギター練習Androidアプリを作った」
- 対象＝AIエンジニア / 内容＝Androidアプリ開発記 / 独自性＝Kotlin未経験×3日

### 公開タイミング

- **火 or 水 9:00-10:00** が最強（通勤時の閲覧）
- 平日昼〜夕も可
- 土日は初動が弱い（ただし週末読み物として機能する）

### X連動 3段ロケット

1. **公開直後**: メインツイート＋30分以内に自分でリプライ（アルゴリズム評価対象）
2. **木曜 20-22時**: 引用RPで違う切り口を投下
3. **日曜 13時以降**: 反応を添えて再投稿

### topics 5個推奨

- カバレッジ最大化のため5個まで埋める
- ジェネリックなタグ（android, kotlin, python等）と固有タグ（claudecode, mcp等）を混ぜる
- `音楽` `物流` 等の業界タグも当事者性が出るなら入れる

### avoidパターン

- タイトルにツール名を3つ以上詰める
- 〇〇入門系（SEOで埋もれる）
- 冒頭で経歴・環境構築から始める（離脱率↑）
- topics 1〜2個（カバレッジ不足）
- クリックベイト（中身と乖離したタイトル）

## 関連リポジトリ

- `c:/Users/ykenk/repos/FretInterval` — FretInterval（Androidアプリ本体）
- 記事執筆時、コード抜粋・git log・lessons.md を参照

## 重要な制約

- **published: true で push = 即公開**。下書きで作業するなら `published: false` のまま push しないこと（または別ブランチ）
- 画像のパスは `/images/<slug>/...` で始まる絶対パス（相対パスでは描画されない）
- `:::message` `:::message alert` の閉じ忘れに注意
