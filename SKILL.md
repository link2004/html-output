---
name: html-output
description: 内容を「ぱっと見で全体像が掴める視覚優位なHTML」にして個人S3に公開し、共有URLを返す。Use when (1)「HTML化して」「リンクで渡して」「ブラウザで見れる形に」 (2)「s3に上げて」 (3) 表・コード・比較・グラフ・タイムライン・図解が必要で Markdown では伝わりにくい時 (4) 後で見返す中長文出力時。
---

# html-output

`$ARGUMENTS` または直前の会話文脈にある内容を、**視覚要素優位のHTML** にして publish.sh で S3 公開、URL を返す。

## 設計原則

「Markdown を `<p>` に置き換えただけ」を絶対やらない。**HTMLの強みを活かす**:

- **5秒ルール**: ページ開いて5秒で「何の話で結論は何か」が掴めること
- **視覚階層 (size+color+weight)**: 重要度の高い情報ほど大きく・色付きで・太く。地の文に埋めない
- **数値は巨大カード化**: 主要メトリクスは 30-50px の数字で並べる
- **比較は side-by-side**: 表より grid でカード化、ヘアラインで色分け
- **フロー・関係は SVG**: inline `<svg>` で手書き。CDN（Chart.js / Mermaid）は重いので原則使わない
- **数値の比較はバーチャート**: CSS の横バー + 緑→赤グラデで視覚化
- **構造はツリー**: ファイル構成等はモノスペース・インデント・色分け
- **結論は callout**: 色付き左ボーダーボックスで目立たせる
- **アンチパターン**: `<p>` の壁 / 全部同じ文字サイズ / 装飾だけのSVG / 絵文字の連打 / 「念のため」CDN

CSS / HTML の具体は **AI の創造性に任せる**。コンポーネントコードを定型化しない（毎回同じ見た目になるとつまらない）。

## カラー基調

**常に白ベース（ライト）固定**。ダークモード自動切替（`prefers-color-scheme: dark`）は **入れない**。

- 背景: `#ffffff`
- 本文: `#111827` 系（やや濃いグレー）
- 補助情報: `#6b7280`（ミュート）
- 罫線: `#e5e7eb`（薄い）
- カード/コードブロック背景: `#fafafa` 〜 `#f3f4f6`（うっすらグレー）
- 主アクセント: **オレンジ（#d97706）** / アクセント薄背景: `#fef3c7`
- 状態色: 緑 `#16a34a` / 黄 `#f59e0b` / 赤 `#dc2626`

## 言語

入力（brief・会話）の言語をそのまま使う。**勝手に翻訳しない**。

## 公開ステップ

```bash
bash ~/.claude/skills/html-output/scripts/publish.sh <<'HTML'
<!DOCTYPE html>
<html lang="ja">
<head>...</head>
<body>...</body>
</html>
HTML
```

引数:

| 引数 | 挙動 | 用途 |
|---|---|---|
| なし | `<timestamp>-<8hex>.html` 新規 | 一時HTML |
| `<name>` | `<slug>-<8hex>.html` 新規 | 永続ドキュメント（wiki用） |
| `<key>.html` | 既存key上書き | 既存ドキュメント更新 |

URL を返す時はコードフェンスで囲む（Markdownの `&` 解釈で壊れないため）。

## wiki cross-link

既存ドキュメントを別ドキュメントから参照したい時:

```bash
bash ~/.claude/skills/html-output/scripts/list.sh <name-prefix>
# → 該当URL一覧
```

得たURLを `<a href="...">` で埋め込む。

## ファイル構成

| パス | 役割 |
|---|---|
| `scripts/publish.sh` | upload + 公開URL返却（stdin pipe） |
| `scripts/list.sh` | 既存docs検索（wiki cross-link用） |
| `config.json` / `config.example.json` | bucket/profile 設定 |
| `references/setup.md` | 初回セットアップ |
