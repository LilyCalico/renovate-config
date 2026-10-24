# renovate-config

LilyCalico のリポジトリで共通利用する Renovate の Shareable Config Preset です。

運用方針は Renovate + Dependabot Alerts のハイブリッドです。

| ツール            | 役割                                                                    |
| ----------------- | ----------------------------------------------------------------------- |
| Renovate          | パッケージ更新の検知・PR 作成、スケジュール管理、グルーピング、automerge |
| Dependabot Alerts | 脆弱性の検知・可視化（Version Updates / Security Updates は無効）       |

## Preset 一覧

| Preset         | 用途                         | 参照方法                                   |
| -------------- | ---------------------------- | ------------------------------------------ |
| `default.json` | 全リポジトリ共通の基本設定   | `local>LilyCalico/renovate-config:default` |
| `nextjs.json`  | Next.js / React プロジェクト | `local>LilyCalico/renovate-config:nextjs`  |

### default.json の内容

| 設定              | 内容                                                              |
| ----------------- | ----------------------------------------------------------------- |
| スケジュール      | 平日 9〜18 時 (JST) に更新 PR を作成                              |
| minimumReleaseAge | リリースから 7 日経過したバージョンのみ対象                       |
| グルーピング      | patch アップデートは 1 つの PR にまとめる                         |
| automerge         | patch、`@types/*` の minor / patch、Linter / Formatter の minor / patch |
| 同時 PR 数        | 最大 10                                                           |
| セキュリティ      | 脆弱性対応 PR はスケジュールを無視して即時作成し、`security` ラベルを付与 |

### nextjs.json の内容

`default` を継承し、以下をそれぞれ 1 つの PR にまとめます。

- Next.js（`next`, `@next/*`, `eslint-config-next`）
- React（`react`, `react-dom`, `@types/react`, `@types/react-dom`）
- Storybook（`storybook`, `@storybook/*`）
- TanStack（`@tanstack/*`）

## 導入手順

> [!IMPORTANT]
> 必ず **手順 2 の `renovate.json` を default ブランチにマージしてから** 手順 3 で Renovate App のアクセスを許可してください。
> `renovate-config` は Renovate の予約名で、`renovate.json` が無いリポジトリには `default.json` が暗黙適用され、automerge を含む設定がそのまま動き出します。

1. **Dependabot Alerts を有効化する**
   Settings → Advanced Security で `Dependabot alerts` を Enable にする。
   `Dependabot security updates` と `Dependabot version updates` は無効のままにする（更新 PR は Renovate が作成する）。

2. **`renovate.json` をリポジトリのルートに追加してマージする**

   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": ["local>LilyCalico/renovate-config:nextjs"]
   }
   ```

   リポジトリ固有の設定は同じファイルで上書きできます。

   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": ["local>LilyCalico/renovate-config:nextjs"],
     "ignoreDeps": ["some-problematic-package"]
   }
   ```

3. **Renovate App にリポジトリへのアクセスを許可する**
   https://github.com/apps/renovate → Configure → Only select repositories で対象リポジトリを追加する。
   初回は Mend Developer Portal に遷移するので、**Renovate Only** → **Scan and Alert** を選ぶ。

> [!NOTE]
> このリポジトリ（`renovate-config`）自体には Renovate App のアクセスを許可しないでください。public リポジトリなので、アクセスを許可しなくても Preset は読み込まれます。

## automerge の前提

automerge は PR のステータスチェックがすべて成功した場合のみ実行されます。
ステータスチェックが 1 つも無いリポジトリでは automerge されません。

GitHub の `Allow auto-merge` を有効にする場合は、ブランチ保護で必須ステータスチェックも設定してください。必須チェックが無いと、CI の完了前にマージされることがあります。

## Preset の追加・変更

- 新しい Preset はルートに `<name>.json` を追加し、`"extends": ["local>LilyCalico/renovate-config:default"]` で基本設定を継承する
- 全リポジトリに影響するため、変更は PR で行い、`Validate Renovate presets` ワークフローが通ることを確認してからマージする
- ローカルでの検証: `npx --package renovate -- renovate-config-validator --strict --no-global *.json`
