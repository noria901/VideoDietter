# VideoDietter

動画を痩せさせるやつ。

任意の動画ファイルをブラウザ内で変換して、SNS用にちょうどいい大きさ・形式にします。動画ファイルがサーバーに送信されることはありません。すべての処理はブラウザのなかで完結します。

## 使い方

1. `index.html` をブラウザで開く（または GitHub Pages 等でホスト）
2. 動画ファイルをドラッグ&ドロップ
3. プリセットを選ぶ
4. 必要ならオプション・ウォーターマークを設定
5. 「変換開始」を押す

初回は ffmpeg.wasm のコア（約30MB）を CDN から読み込みます。2回目以降はブラウザキャッシュから読まれるので速いです。

## プリセット

| プリセット | 解像度 | 設定 | 用途 |
|---|---|---|---|
| X (標準) | 1080p | H.264 / 8Mbps | X (Twitter) 通常投稿 |
| X (軽量) | 720p | 30fps / 2Mbps | モバイル投稿、回線弱め |
| Bluesky | 1080p | 5Mbps | Bluesky 投稿 |
| YT Shorts/縦 | 1080×1920 | CRF 20 | 縦動画、YouTube Shorts |
| 保管 (軽量) | 720p | CRF 28 | 容量最優先のアーカイブ |
| 保管 (画質) | 元解像度 | CRF 20 | 画質維持のアーカイブ |

## 機能

- **プリセット切替**: SNS別に最適化した6種
- **サイズ予測**: 変換前に推定出力サイズ・ビットレートを表示
- **音声なし変換**: 音声を取り除く
- **faststart**: moov atomを先頭に置いてWeb再生最適化（既定でON）
- **音量正規化**: `loudnorm` で -14 LUFS に揃える
- **回転**: 90°/-90°/180°
- **ウォーターマーク合成**: PNG等を9点位置・サイズ・不透明度・余白指定で重ねる

## 仕組み

- [ffmpeg.wasm](https://ffmpegwasm.netlify.app/) を CDN から読み込んでブラウザ内で実行
- フィルタチェーンは `-filter_complex` で組み立て（scale → 回転 → fps → ウォーターマーク overlay）
- 動画メタデータ（尺・解像度）は HTML5 `<video>` 要素で取得
- CRFモードのサイズ予測は CRF値→ビットレート換算 + 解像度補正で算出（あくまで概算）

## 制約

- ffmpeg.wasmはシングルスレッド動作のため、長尺・高解像度の動画は変換に時間がかかります
- 巨大なファイル（数GB）はメモリの都合で扱えない場合があります
- マルチスレッド版（ffmpeg.wasm-mt）に切り替えるには COOP/COEP ヘッダ設定が必要なため未採用

## ライセンス

MIT License (このアプリのコード)

ffmpeg.wasm および FFmpeg 本体のライセンスはそれぞれ [@ffmpeg/ffmpeg](https://github.com/ffmpegwasm/ffmpeg.wasm) および [FFmpeg](https://ffmpeg.org/legal.html) を参照してください。FFmpegは LGPL 2.1+ / GPL 2+ で配布されています。

## 元ネタ

[めむくろさん](https://x.com/MEMchro) の [Twitter用に動画変換するやつ (2016/2019)](https://cloth.moe/2016_twitter_convert/) を、現代のブラウザ・SNS事情に合わせて作り直したもの。本家のローカル処理の思想は引き継いでいます。

---

> Twitter→X、対応動画仕様の変化、Bluesky/Threadsの登場、ブラウザ内ffmpegの実用化 — 2016年から色々変わったので。
