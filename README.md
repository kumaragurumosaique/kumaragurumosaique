# oauth

Slackアプリケーションサンプルプログラム  
- 配布可能なアプリケーションの要件であるSlack⇔アプリケーション間のOAuth認証を実装している。  
  参考：https://slack.dev/java-slack-sdk/guides/ja/app-distribution
- アプリケーションが参加しているチャンネル内でメンションすると、チーム or ユーザー情報を返す。  
  参考：https://slack.dev/java-slack-sdk/guides/ja/events-api

## OAuth認証
[Slack App](https://api.slack.com/apps)から取得したSigningSecret、ClientId、ClientSecretおよびURLパスを適切に設定していれば問題なく動作する。  
トークン保存処理は既定のままなので、DB等のストレージに保存する方法は要検討。

### 権限  
Features > OAuth & Permissions > Scopes にアプリケーションが必要とする権限を設定する。    
- [app_mentions:read](https://api.slack.com/scopes/app_mentions:read)：メンション処理権限  
- [commands](https://api.slack.com/scopes/commands)：スラッシュコマンド権限  
- [team:read](https://api.slack.com/scopes/team:read)：チーム情報取得権限  
- [users:read](https://api.slack.com/scopes/users:read)：ユーザー情報取得権限  
- [users:read.email](https://api.slack.com/scopes/users:read.email).email：ユーザーのメールアドレス取得権限  
- [im:history](https://api.slack.com/scopes/im:history)：ダイレクトメッセージ履歴取得権限

### アプリケーションの登録に失敗していた件  
Slackの不備？   
インストールURL（/slack/install）を直接呼び出せばよい。  
参考：https://qiita.com/seratch/items/610c14208772d49ac9e4#comment-a51ba74a52012a8e2f34  

## イベント  
Features > Event Subscriptions  
- Enable Events をONに設定する。    
- Enable Events > Request URL に、イベント受信用URL（https://{yourDomain}/slack/events）を設定する。  
- Subscribe to bot events に、ハンドリングするイベントを設定する。  
  - [app_mention](https://api.slack.com/events/app_mention)：チャンネルに登録されたアプリケーションへのメンション  
  - [message.im](https://api.slack.com/events/message.im)：ダイレクトメッセージ

## 情報取得  
[Slack API](https://api.slack.com/apis)を使用している。
- [team.info](https://api.slack.com/methods/team.info)：チーム情報取得  
- [users.info](https://api.slack.com/methods/users.info)：ユーザー情報取得    

コード的にはContext#client()から当該APIのラッパーメソッドを呼び出すだけ。

## App Manifest
```json
{
  "display_information": {
    "name": "oauthapp"
  },
  "features": {
    "app_home": {
      "home_tab_enabled": false,
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "bot_user": {
      "display_name": "oauthapp",
      "always_online": true
    },
    "slash_commands": [
      {
        "command": "/user",
        "url": "https://{yourDomain}/slack/events",
        "description": "Get user information.",
        "should_escape": false
      },
      {
        "command": "/team",
        "url": "https://{yourDomain}/slack/events",
        "description": "Get team information.",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "redirect_urls": [
      "https://{yourDomain}/slack/oauth/callback"
    ],
    "scopes": {
      "bot": [
        "app_mentions:read",
        "commands",
        "team:read",
        "users:read",
        "users:read.email",
        "im:history"
      ]
    }
  },
  "settings": {
    "event_subscriptions": {
      "request_url": "https://{yourDomain}/slack/events",
      "bot_events": [
        "app_mention",
        "message.im"
      ]
    },
    "org_deploy_enabled": false,
    "socket_mode_enabled": false,
    "token_rotation_enabled": false
  }
}
```

## 備考  
- 「このアプリへのメッセージ送信はオフにされています」への対応方法  
  1. Allow users to send Slash commands and messages from the messages tab をONにする。  
  1. アプリケーションを再インストールする。
  1. Slack（デスクトップアプリケーション）を再起動する。  

  参考：https://qiita.com/mu5dvlp/items/c9008ffb1b3a61ea9411

<div style="text-align: right;">
以上
</div>

