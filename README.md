# Teams Bot

AgileWorks における書類の申請・承認・却下・引戻し操作を行うことができるBotです。

## 前提条件

- Node.js がインストールされている。
- Microsoft Teams がインストールされており、アカウントを所持している。
- Microsoft Azure のアカウントを所持している。
- ngrok がインストールされている。

## 設定

> ローカルコンピュータでサンプル実行するための手順を記載します。
> Teams がボットを呼び出すためにngrokを利用してボットを外部公開します。

### ボットの設定

1. manifest.jsonを編集しmanifest.zipを作成します。

- `<<MicrosoftAppID>>`にAzure Bot Service の設定 > 構成 からMicrosoft App IDを取得し設定します。`<<MicrosoftAppID>>`は複数箇所に設定することに注意してください。
- manifestフォルダー内のファイルを圧縮して`manifest.zip`ファイルを作成します。zipファイルにサブフォルダーが含まれないように注意してください。

2.  `manifet.zip` を Teams カスタムアプリとしてアップロードします。

- Temas左メニューの[アプリ] > [アプリを管理] から[アプリをアップロード]を選択します。[カスタムアプリをアップロード] から作成した `manifest.zip` をアップロードします。
- ボットを追加するスコープをチャット/チームを選択してください。

### コードの設定

.envファイルを編集します

- `MicrosoftAppID`：Azure Bot Service の設定 > 構成 からMicrosoft App IDを取得し設定します。
- `MicrosoftAppPassword`：アプリの登録からクライアントシークレットを取得し設定します。
- `Url`：AgileWorks のベースURLを設定します。
- `BearerToekn`：AgileWorks の管理画面から取得できるアクセストークンを取得し設定します。

## アプリの実行

1. ngrok を実行する

   ngrok を起動し、次のコマンドを使用して新しいトンネルを作成します。

   ```bash
   ngrok http 3978 --host-header="localhost:3978"
   ```

2. 表示される公開URLを Azure Bot Service の[メッセージング エンドポイント]に設定します。

   ```
   https://xxxx-xxx-xx-xx-xx-xxx.ngrok-free.app/api/messages
   ```

3. モジュールをインストール

   ```
   npm install
   ```

4. ボットの実行
   ```
   npm start
   ```

## コマンド一覧

- npm モジュールのインストール

  ```bash
  npm install
  ```

- ボットの起動

  ```bash
  npm start
  ```

- リアルタイム実行

  ```bash
  npm run watch
  ```

- ソースコード整形

  ```bash
  npm run format
  ```

- eslint実行

  ```bash
  npm run lint
  ```

- ソースコード整形 & eslint実行

  ```bash
  npm run fix
  ```

## Temas Bot Framwrok について

### Teams ユーザー情報の取得

[TeamsInfo class](https://learn.microsoft.com/ja-jp/javascript/api/botbuilder/teamsinfo?view=botbuilder-ts-latest) を利用して取得する

### サンプルコード

- https://github.com/OfficeDev/Microsoft-Teams-Samples/tree/main
- https://github.com/microsoft/BotBuilder-Samples/tree/main

### Adaptive Card のオプション

- https://adaptivecards.io/explorer/

# Sample Code

## パッケージ構成

```
 manifest　# manifest.zipに関連するファイルが含まれます
 src
  |-- bot
  |   `-- handler # Botのイベントハンドラーを提供します
  |-- client # API呼び出しパッケージです
  |   `-- agileworks
  |-- model # ドメインモデルを提供します
  |-- service # アプリケーションのビジネスロジックを提供します
  |-- utils # ユーティリティ関数
  `-- view
      |-- application # 申請処理に関連するビューを提供します.
      |-- approval # 承認処理に関連するビューを提供します.
      |-- circulation # 回付処理に関連するビューを提供します.
      |-- component # Teams UIコンポーネント
      |-- list # 一覧表示ビューを提供します.
      `-- result # 結果表示ビューを提供します.
```

## 申請処理

> AgileWorks のAPIを利用した各処理の実現方法と課題について説明します

### 利用した AgileWorks API

- 組織所属参照API
- 新規書類データ作成API
- 新規書類データ保存API
- 書類作成/申請API

### 処理手順概要

1. Teams BotFramework を利用して`applyUserCode`を取得※
2. ユーザーコードから組織所属参照APIを利用して`applyUnitCode`を取得
3. `formCode`,`ruleCode`※,取得した情報から新規書類データ作成APIを利用して初期状態の書類データを作成
4. 書類データからTeamsに表示する申請フォームを作成※
5. 申請フォームから取得した入力データと書類データから新規書類データ保存APIを利用して書類を保存
6. 保存された書類から`docId`を取得して書類作成/申請APIを利用して申請

### 課題

- Teamsのユーザー情報とAgileWorksのユーザーコードが一致している前提が必要
- ユーザーコードからユーザーが申請可能なフォーム一覧を取得したい

  - フォーム一覧に含めたい情報は以下の通り

  ```
  {
    formName: <フォーム名>, #フォームを識別する画面表示用
    formCode: <フォームコード>, #新規書類データ作成APIを利用するため必須
    ruleCode: <回付ルールコード>, #新規書類データ作成APIを利用するため必須
  }
  ```

- 申請フォーム作成のための詳細なフォーム情報を取得したい

  - Sampleでは新規書類データ作成APIで取得できる書類データを使用してフォームを生成している
  ```
  {
    "fieldType" : "MONTHFIELD",
    "fieldId" : "jissi_month",
    "dataType" : "STRING",
    "dataValue" : {
      "entryList" : {
        "entries" : [ ]
      },
      "value" : "04"
    },
    "name" : "実施月"
  }
  ```


## Json サンプル

```
{
	"userInfo": {
		"userPrincipal": ${usercode},
		"tenantId": ${tenantId},
		"workspaceId": ${workspaceId}
	},
	"query": "name=doc.Doc&method=selectDoc",
	"requestData": {
		"docView": {
			"columnList": {
				"entries": [
					{
						"type": "DOC_ID"
					},
					{
						"type": "APPLY_USER_CODE"
					}
				]
			}
		},
		"condition": {
			"userCode": ${usercode},
			"workflowState": "ACTIVE"
		}
	}
}
```
