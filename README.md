# HatenaBlog Workflows Boilerplate(β)

- このBoilerplateは、企業がはてなブログで技術ブログを運営する際のレビューや公開作業など、運営ワークフローを支援する目的で作成しています
- GitHub 上で、はてなブログとの記事の同期、下書きの作成・編集・公開、公開記事の編集などを行うことができます。下書きの作成時にプルリクエストが作成されるため、記事のレビューなどの業務のワークフローに組み込むことが容易になります
- 本機能はベータ版です。正常に動作しない可能性がありますが、予めご了承下さい


## セットアップ

1. 本リポジトリトップに表示されている、「Use this template ボタンクリック > Create a new repository」から、新規にリポジトリを作成する
    - ![Use this templateボタンの位置](https://cdn-ak.f.st-hatena.com/images/fotolife/h/hatenablog/20231107/20231107164142.png)
2. `blogsync.yaml`の各種項目を記述し、変更を `main` ブランチにコミットしてください
    - `<BLOG DOMAIN>` にはブログのブログ取得時に設定したドメインを指定してください(独自ドメインではありません)
    - `<BLOG OWNER HATENA ID>` にはブログのオーナー(ブログ作成者)のはてなIDを指定してください
    - 上記のどちらの項目もブログの「詳細設定 > AtomPub > ルートエンドポイント」から確認できます。ルートエンドポイントは以下のように構成されています
        - `https://blog.hatena.ne.jp/<BLOG OWNER HATENA ID>/<BLOG DOMAIN>/atom`
```yaml
<BLOG DOMAIN>:
  username: <BLOG OWNER HATENA ID>
default:
  local_root: entries
```
3. GitHub リポジトリの設定 「`Secrets and variables` > `actions` > `Repository variables`」 から以下のVariableを登録する
    - Name: `BLOG_DOMAIN`
    - Value: ブログのドメインを指定してください 例) staff.hatenablog.com
4. GitHub リポジトリの設定 「`Secrets and variables` > `actions` > `Repository Secrets`」 から以下のSecretを登録する
    - Name: `OWNER_API_KEY`
    - Secret: ブログのオーナーはてなアカウントの APIキーを指定してください
        - APIキーは、ブログオーナーアカウントでログイン後、[アカウント設定](https://blog.hatena.ne.jp/-/config) よりご確認いただけます
5. GitHub リポジトリの設定 「`Actions` > `General`」 の `Workflow permissions` の設定を以下の通り変更する
    - `Read and write permissions` を選択する
    - `Allow GitHub Actions to create and approve pull requests` にチェックを入れる
6. GitHub リポジトリの設定 「`Branches`」 の`Add branch protection rule`ボタンから、ルールを作成する
    - `Branch name pattern` に `main` を指定する
7. GiHub リポジトリの設定 「`General`」 の `Pull Requests` 項の `Allow auto merge` にチェックを入れる
8. リポジトリにはてなブログの記事を同期させる
    - Actions タブを開き `initialize` workflow を選択する
    - Run workflow をクリック
    - `Branch: main` が指定されていることを確認し、`Run workflow`ボタンをクリック
    - 全記事が含まれたプルリクエストが作成されます。これをマージしてはてなブログとリポジトリの状況を同期させてください
    - ![Actionsタブ、workflowリスト、Run workflowボタン](https://cdn-ak.f.st-hatena.com/images/fotolife/h/hatenablog/20231107/20231107163433.png)
9. はてなブログの「[設定 > 編集モード](https://blog.hatena.ne.jp/my/config#blog-config-syntax)」設定を「Markdownモード」に設定する

### Renovate / Dependabot のセットアップ

- ワークフローの機能は[hatena/hatenablog-workflows](https://github.com/hatena/hatenablog-workflows)のReusable Workflowを呼び出すことで提供されています
- hatenablog-workflows の機能追加やバグ修正を反映するにはGitHub Actionsの参照を更新する必要があり、自動的な更新にRenovateやDependabotをご利用いただけます
- それぞれのセットアップ方法はドキュメントをご参照ください
  - Renovate: [Automated Dependency Updates for GitHub Actions - Renovate Docs](https://docs.renovatebot.com/modules/manager/github-actions/)
  - Dependabot: [Dependabot でアクションを最新に保つ - GitHub Docs](https://docs.github.com/ja/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot)

## オプション
- 下書きの作成時のプルリクエストをドラフトプルリクエストとして作成するかどうかのオプション
  - ドラフトプルリクエストは[利用できるプランに制限](https://docs.github.com/ja/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/changing-the-stage-of-a-pull-request)があります。対象外のプランを利用している場合、以下のファイルの該当行を `draft: false` に変更してください 
  - `/.github/workflows/pull-draft.yaml#L15`
  - `/.github/workflows/create-draft.yaml#L15`

## 想定ワークフロー

このツールで想定している下書き作成から記事公開までのワークフローは以下のとおりです。

1. 下書きを作成する
2. 下書き記事をプルリクエスト上で編集する
3. 適宜レビューなどを行い、通れば次の公開手順に進む
4. 下書き記事を公開する

## 手順の詳細

### 下書きの作成

下書きの作成方法は以下の2通りの方法があります。

- ブログメンバーが個人のアカウントで投稿する(記事の署名は個人のアカウントになります)
- ブログオーナーのアカウントで投稿する(記事の署名はブログオーナーアカウントになります)

それぞれ、下書き作成の手順が異なります。ブログの運営方針や記事の内容に沿った方法を選択してください。

### ブログメンバーが個人のアカウントで投稿する場合

1. 投稿したいブログの編集画面を開く
2. 下書き記事の記事タイトルを `{{username}}-{{日付}}` 等、ユニークな記事タイトルに設定し、クリップボードにコピーしておく
3. 下書きを投稿する
4. Actions から `pull draft from hatenablog`を選択し、`Draft Entry Title`に先程コピーしたタイトルを設定、`Branch: main`に対して実行する
5. 投稿した下書きを含むプルリクエストが作成される

### ブログオーナーのアカウントで投稿する場合

1. Actions から  `create draft` を選択し、`Title`に記事タイトルを設定、`Branch: main`に対して実行する
2. 作成した下書きを含むプルリクエストが作成される

### 下書き記事の編集

- 手順「下書きの作成」で作成したプルリクエスト上で記事を編集してください
- はてなブログでプレビューできるようにするため、下書き記事に限りプッシュされた時点ではてなブログに同期されます
- 記事の編集画面の URL は、プルリクエストに記載されています。編集画面に遷移した後、下書きプレビューの URL を発行し、プルリクエストの概要に記載しておくとプレビューが容易になります

### 下書き記事の公開

- 下書き記事の `Draft: true` 行を削除し、プルリクエストを main ブランチにマージすると記事がはてなブログで公開されます
- 記事を公開すると、下書き記事は 下書き記事用ディレクトリ `draft_entries` から 公開記事用`entries` ディレクトリに移管されます

### 既存記事を修正する場合

- 修正ブランチを作成し、main ブランチにマージすると修正がはてなブログに反映されます


## 予約投稿について

- はてなブログでは記事の予約投稿が利用可能ですが、このBoilerplateから記事の予約投稿を行うことはできません。記事を予約投稿するには、はてなブログの記事の編集画面から設定を行ってください。
  - [日時を指定して予約投稿する（編集オプション） - はてなブログ ヘルプ https://help.hatenablog.com/entry/editor/publish/schedule]
  - ※予約投稿の設定を行ったあと、記事が投稿されるまでにこのワークフローを経由して記事の変更を行うと、予約投稿の設定が解除されてしまいますのでご注意ください
- リポジトリで管理していた記事を予約投稿する場合、以下のいずれかの手順で予約投稿による変更を同期して頂く必要があります。

### プルリクエストをマージする方法
1. プルリクエストをデフォルトブランチにマージする
  - この際、`draft:true` を削除してはいけません
2. はてなブログ側で予約投稿を行う
3. 記事が公開されたら `pull form hatenablog` アクションを実行する
4. `/draft_entries` から予約投稿記事を削除する

### プルリクエストをクローズする方法
1. プルリクエストをクローズする
2. はてなブログ側で予約投稿を行う
3. 記事が公開されたら `pull from hatenablog` アクションを実行する


## プルリクエストテンプレートを利用する

- `create draft` アクションまたは `pull draft from hatenablog` アクションによってプルリクエストを作成する際、リポジトリに `.github/PULL_REQUEST_TEMPLATE.md` または `.github/PULL_REQUEST_TEMPLATE/draft.md` という名前のテンプレートファイルが存在すれば、そのテンプレートを利用してプルリクエストが作成されます。
- プルリクエストテンプレートでは以下のプレースホルダが利用可能です。これらのプレースホルダはプルリクエストの作成時には記事に固有の値に置き換えられます

| プレースホルダ | 値 |
| :------------: | :----: |
| `${EDIT_URL}`   | 下書き記事の編集画面のURL |
| `${PREVIEW_URL}` | 下書き記事のプレビューURL（まだプレビューURLがない場合は `なし`） |
| `${TITLE}` | 下書き記事の記事タイトル |
| `${ENTRY_ID}` | 下書き記事のID |
| `${OWNER_NAME}` | ブログのオーナーのアカウント名 |


## Boilerplateに新しく追加されたWorkflowを取得する

- workflowの変更は [hatena/hatenablog-workflows](https://github.com/hatena/hatenablog-workflows) を更新することによって提供されますので、GitHub Actionsの参照を更新していただくことで、機能の更新を利用できます。
- ただし、新しくworkflowが追加されたりした場合は、Boilerplateを元に作成されたリポジトリに新しいファイルを追加したり既存のファイルを更新する必要があります
- 新しいファイルを取得するには`scripts/download_boilerplate_workflows.sh`を実行してください

```bash
bash scripts/download_boilerplate_workflows.sh
```

### Scriptが見つからない場合

- 手元のリポジトリに上記のファイルがない場合があります
- お手数ですが、その場合は[こちらのファイル](https://github.com/hatena/Hatena-Blog-Workflows-Boilerplate/blob/main/scripts/download_boilerplate_workflows.sh)を自身のリポジトリに追加してください

## Tips

### カスタムURLを指定する
記事のURLは、特に指定しない場合投稿日時から自動的に決定されますが、カスタムURL機能を利用すると任意のURLを指定することができます。

GitHubから記事を編集する場合にもカスタムURLを指定することが可能です。指定する場合、記事の設定領域に `CustomPath` フィールドを追加します。
設定例は以下の通りです。

```
---
Title: カスタムURLの設定例
EditURL: https://blog.hatena.ne.jp/hatenablog/example.hatenablog.com/atom/entry/0123456789
PreviewURL: https://example.hatenablog.com/draft/entry/xxxxxxx
Draft: true
CustomPath: custom/url
---
```
この記事を公開すると `https://example.hatenablog.com/entry/custom/url` として公開されます。

## トラブルシューティング

### はてなブログ側のデータとリポジトリのデータとで差分が発生した場合

はてなブログのWebの編集画面から記事を更新するなど、はてなブログ側のデータとリポジトリのデータに差異が発生してしまう場合があります。
この場合、 Actions の `pull from hatenablog` を選択、`Branch: main`に対して実行してください。
実行すると、リポジトリの更新日時以降に更新された公開記事のデータを更新するプルリクエストが作成されます。
これをマージすることで、最新のデータに更新することができます。

## workflow に関する詳細

- 各 workflow では下記で提供されている Reusable workflows を利用しています
  - https://github.com/hatena/hatenablog-workflows

## textlint による文章チェック

このリポジトリでは、Markdownファイルの文章品質を保つためにtextlintを導入しています。textlintは日本語の文章をチェックするツールで、文法や表記の一貫性を維持するのに役立ちます。

### 設定内容

- 対象ファイル: `draft_entries` ディレクトリ内のMarkdownファイル
- 使用ルール:
  - `preset-japanese`: 日本語の一般的なルールセット
  - `preset-ja-technical-writing`: 技術文書向けのルールセット
  - `spellcheck-tech-word`: 技術用語のスペルチェック
  - `prh`: 表記ゆれを検出・修正するためのルール

### 使用方法

手動でtextlintを実行する場合:

```powershell
# 文章をチェックする
npm run textlint

# 自動修正可能な問題を修正する
npm run textlint:fix
```

### 自動チェック

huskyとlint-stagedを使用して、コミット時に変更されたMarkdownファイルに対して自動的にtextlintが実行されます。エラーがある場合はコミットが中断されます。
