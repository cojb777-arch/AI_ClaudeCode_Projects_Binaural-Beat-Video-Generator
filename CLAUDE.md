# Binaural Beat Video Maker — プロジェクト固有ルール

共通ルール（プライバシー・シークレット・Git 運用など）は全プロジェクト共通の `CLAUDE.md` に従う。ここには書き写さない。

## 1. プロジェクト概要

バイノーラルビート（うねり 1〜30Hz）・サブリミナル音声・サブリミナルフラッシュ（画像／テキスト）を組み合わせた動画を、ブラウザ内で生成して WebM として保存するアプリ。個人利用が前提。
公開範囲: GitHub リポジトリ公開、Cloudflare Workers（Git 連携）と GitHub Pages で静的配信。

## 2. 技術スタックと構成

- 依存ライブラリなしの単一 HTML（`public/index.html`）。Web Audio API で音声生成、Canvas で映像、MediaRecorder でリアルタイム録画。
- サーバー処理なし。Cloudflare Workers は静的アセットを配るだけ（`wrangler.jsonc`）。
- `public/` ＝ 配信される唯一のディレクトリ。**ここに置いたものは全て公開される**。

## 3. 主要コマンド

- ローカル確認: `python -m http.server 8330 --directory public`（または `public/index.html` を直接開く）
- デプロイ検証（認証不要）: `npx wrangler@4 deploy --dry-run`
- デプロイ: Cloudflare の Git 連携（Workers Builds）により、**`main` への push で自動デプロイされる**（デプロイコマンドは `npx wrangler deploy`、ビルドコマンドは空）。手動デプロイは `npx wrangler login` → `npx wrangler deploy`。
- **Cloudflare 側のプロジェクト名は `wrangler.jsonc` の `name`（`binaural-beat-video-maker`）と必ず一致させる**。不一致だとビルドが失敗する。名前を変える場合は両方を同時に変更する。

## 4. このプロジェクト特有の制約

- 録画は実時間方式（5分の動画＝5分）。録画中はタブを前面に保つ必要がある。
- バイノーラルは**ステレオヘッドホン前提**。出力は WebM（MP4 変換は未対応）。
- **光過敏性への安全制限は緩めない**: フラッシュ間隔は最短3秒・表示は最長100ms、うねり3Hz超ではオーブの発光振幅を抑制、録画時は確認用の大音量を強制OFF。上限を広げる変更は、理由を示して合意を取ってから行う。
- 冒頭の注意書き（てんかん・光過敏性発作・心疾患・妊娠中・運転中の使用を控える旨）は削除しない。

## 5. 公開物の一覧（§1 点検の対象）

- GitHub リポジトリ（README、コード、コミット履歴）。コミットの Author は GitHub の noreply アドレスを使う。
- Cloudflare Workers の公開 URL（`<worker名>.<アカウントのサブドメイン>.workers.dev`）。サブドメインに個人名が入っていないことは確認済み。
- 録画された動画ファイル（ファイル名は `binaural_<帯域>_<Hz>_<日時>.webm`。個人情報は含まない）。

## 6. やってはいけないこと

- `public/` に `.env`・キー・個人情報・ローカル絶対パスを置かない。
- 指示なしに push・デプロイをしない（`main` への push は本番に自動反映される）。
- 安全制限（上記 §4）を黙って緩めない。

## 7. 未解決事項 / 既知の課題

- 実時間より速いオフライン生成、MP4 変換（ffmpeg）、アイソクロニックトーン対応は未実装。
- GitHub Pages（`.github/workflows/jekyll-gh-pages.yml`、ビルド元 `./public`）と Cloudflare Workers の二重公開になっている。一本化するかは未決定。
