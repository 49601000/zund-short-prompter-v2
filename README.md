# ずんだもんショート演出エンジン v1.3

ずんだもんショート用の会話プロンプトを生成する静的Webアプリです。

`index.html` はモード選択ページで、実際の生成画面は2つのHTMLに分かれています。

- `ver-dialogue.html`: ずんだもん・めたんの2人会話版
- `ver-Ten-no-koe.html`: 現行の天の声あり3人構成版

つまりこういうことや。

```text
index.html
  ├─ ver-dialogue.html      # 天の声なし
  └─ ver-Ten-no-koe.html    # 天の声あり
```

## 主な機能

- 議題からショート動画向けの会話プロンプトを生成
- 1d20ロールで会話ルールを決定
- Style / Opening Mode / Length Preset を選択
- custom min/max で文字数レンジを調整
- Patch Selector でイベント・ゲスト・文脈パッチを適用
- Conversation Arc Roll で会話の流れを変化
- Final Prompt のコピー
- iOSスタンドアロン時のみ `Geminiで開く` ボタンを表示
- localStorageで入力状態と生成結果を保存

## モード構成

### ver-dialogue

`ver-dialogue.html` は、ずんだもんとめたんだけで会話を組みます。

このモードでは `app.js` が出力プロンプト素材から天の声関連を除外します。
AIへの「天の声を出すな」という禁止指示で制御するのではなく、HTML側のモードによって最終プロンプトのベースを切り替えます。

### ver-Ten-no-koe

`ver-Ten-no-koe.html` は、現行の天の声あり構成です。

`base/`, `data/personas/`, `data/styles/`, `data/rules/`, `base/output_format.txt` に含まれる天の声ルールをそのまま使います。
天の声は補助キャラクターとして扱い、現行プロンプト通り必要時に一言だけ参入する建付けです。

## 起動方法

`fetch` でJSONやテキストを読むため、`file://` 直開きではなくローカルサーバ経由で起動してください。

```bash
python -m http.server 8000
```

ブラウザで開きます。

```text
http://localhost:8000/
```

直接各モードを開く場合はこちらです。

```text
http://localhost:8000/ver-dialogue.html
http://localhost:8000/ver-Ten-no-koe.html
```

## 使い方

1. `index.html` でモードを選ぶ
2. 議題を入力する
3. `Roll (1d20)` を押す
4. Style / Opening / Length / Patch を必要に応じて調整する
5. `Generate Prompt` を押す
6. `Copy` でFinal Promptをコピーする

iOSスタンドアロン表示では、生成後に `Geminiで開く` も使えます。

## Prompt Assembly

`app.js` の `assemblePrompt()` は、次の順番で最終プロンプトを組み立てます。

1. Layer 1: Debate Engine
2. Layer 2: Persona Brain Layers
3. Layer 3: Rule
4. Layer 4: Style Renderer
5. Layer 4.5: Conversation Arc Variant
6. Layer 5: Opening Policy
7. Layer 6: Length Policy
8. Layer 7: Topic
9. Layer 8: Output Format

`window.PROMPT_VARIANT` によって、読み込む人格とプロンプト素材の扱いが変わります。

```js
window.PROMPT_VARIANT = "dialogue";    // 天の声なし
window.PROMPT_VARIANT = "ten_no_koe";  // 天の声あり
```

## localStorage

主な保存キーです。

- `resuba_topic`
- `resuba_enabled_patch_ids`
- `resuba_dice_result`
- `resuba_selected_rule_id`
- `resuba_selected_style`
- `resuba_selected_opening_mode`
- `resuba_selected_length_preset`
- `resuba_custom_length_min`
- `resuba_custom_length_max`
- `resuba_final_prompt_dialogue`
- `resuba_final_prompt_ten_no_koe`

Final Prompt はモードごとに分けて保存されます。

## ディレクトリ構成

```text
.
├─ index.html                 # モード選択
├─ ver-dialogue.html          # ずんだもん・めたん版
├─ ver-Ten-no-koe.html        # 天の声あり版
├─ app.js                     # プロンプト生成ロジック
├─ style.css
├─ manifest.webmanifest
├─ base
│  ├─ base_prompt.txt
│  ├─ debate_engine.txt
│  ├─ dictionary.txt
│  ├─ length_policy.json
│  ├─ opening_policy.json
│  └─ output_format.txt
├─ data
│  ├─ characters              # 旧形式フォールバック
│  ├─ personas                # zundamon / metan / ten_no_koe
│  ├─ rules                   # state_rule.json
│  ├─ styles                  # zundamon_short_dialogue.json
│  └─ trait
├─ patches
│  ├─ index.json
│  └─ *.json
└─ scripts
   └─ build-patch-index.js
```

## Patch Index 再生成

Patchファイルを追加・整理したら、次のコマンドで `patches/index.json` を再生成します。

```bash
node scripts/build-patch-index.js
```

スクリプトは `patches/*.json` を走査し、`id`, `label`, `type` を持つJSONだけを採用します。

## 開発メモ

- `ver-dialogue.html` と `ver-Ten-no-koe.html` は同じ `app.js` を使います。
- モード差分はHTML側の `window.PROMPT_VARIANT` で決まります。
- 天の声なし版は、AIへの禁止文ではなく、出力プロンプト素材から天の声関連を抜く方針です。
- `index.html` は生成画面ではなく、モード選択ランチャーです。
