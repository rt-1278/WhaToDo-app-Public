## WhatToDo-app

**1. プロジェクトの概要**  
- **アプリ名**
  - WhaToDo-app  
- **概要**
  - モバイルアプリのWhaToDoと連携して、外部APIとの連携、データ管理、情報をレスポンスする  
- **主な機能**  
  - ユーザー管理：UUIDを発行して、ユーザーの利用状況を管理
  - データ管理：プラン情報やプラン詳細情報のCRUD操作
  - バッチ処理：毎日0時に過去のプラン情報の削除

**2. 環境設定及び使用技術**
- 言語：Ruby（3.3.3）
- フレームワーク：Rails（7.1.3.4）
- データベース：Mysql（8.0.28）
- キャッシュ：Redis（7.0）
- ジョブ管理：Sidekiq（7.2.4）
- 仮想環境構築：Docker Desktop（4.34.0）
- 自動テスト: Gihub Actions
- テスト: Rspec

- #### セットアップ
  - **Dockerイメージの生成**
    - sudo docker build -t app .
  - **コンテナの生成および起動**
    - sudo docker compose up
  - **コンテナでbundle installが必要な場合**
    - docker compose run app bundle install
  - **dbの作成**
    - docker compose run app bin/rake db:creat
  - **テーブルの生成&スキーマの変更反映**
    - docker compose run app bundle exec ridgepole --config config/database.yml --env development --file db/Schemafile --apply
    - docker compose run app bin/rake active_storage:install
    - docker compose run app bin/rake db:migrate
  - **※ 開発環境でアプリケーションサーバの動作確認する場合、証明書を個人で発行し
puma.rbのssl_bindにkeyとcertがあるディレクトリを指定すること**  

**3. 環境変数の設定**  
  
| 環境変数名     | 説明                           | 値の例                       |
|----------------|--------------------------------|------------------------------|
| `OPEN_AI_API_KEY` | Open AI APIの認証キー            | `各自でアカウントを発行してキーを取得する必要` |
| `MAX_REQUEST_PLAN_COUNT_FOR_1_DAY`   | 1日当たりのプラン取得回数上限 | `3`            |
| `GEOCODING_API_KEY`      | Geocording APIの認証キー              | `各自でGoogle Cloudのアカウント発行とGeocording API利用の設定を行いAPIキーを取得する必要`               |
| `CUSTOM_SEARCH_API_KEY`   | Google Custom Search APIの認証キー | `各自でGoogle Cloudのアカウント発行とCustom Seach API利用の設定を行いAPIキーを取得する必要`            |
| `CX`      | Google Custom Search APIの検索エンジンID              | `各自でGoogle Cloudのアカウント発行とCustom Seach API利用の設定を行い検索エンジンIDを取得する必要`               |
| `CUSTOM_SEARCH_URL`   |  Google Custom Search APIの検索URL | `https://www.googleapis.com/customsearch/v1?`            |
| `SCHEDULE_FILE_PATH`      | Schedule.ymlのパス              | `config/schedule.yml`               |
| `SMTP_USERNAME`   | SMTPの送信元のメールアドレス（gmail） | `ooo@gmail.com`            |
| `SMTP_PASSWORD`      | GMAILのSMTPとして利用する場合のパスワード              | `GmailからSMTP利用の設定を行い、パスワードを取得する必要`               |
| `RECIPIENTS_EMAIL`      | エラー通知メールの送信先メールアドレス              | `ooo@error.com`               |


**4. API仕様**  
- **プラン情報取得API** 
  - メソッド：getリクエスト
  - リクエスト先：サーバーのURL + api/v1/plans
  - リクエストパラメータ
    - user_id: String
    - latitude: Double
    - longitude: Double
    - is_update: Double
  - レスポンスパラメータ
    - plans: リスト形式のPlan
    - errors: リスト形式のString
    
- **プラン詳細情報取得API**  
  - メソッド：getリクエスト
  - リクエスト先：サーバーのURL + api/v1/plans/details
  - リクエストパラメータ
    - plan_id: String
  - レスポンスパラメータ
    - plans: リスト形式のPlanDetail
    - errors: リスト形式のString

**5. 利用上の注意点**
- 各ビルドバリアントに応じて、ホスト先のURLとActiveStorageのホスト先（画像の保存先）のURLを指定すること
  - Rails.application.routes.default_url_options[:host] = ホスト先のURL
  - ActiveStorage::Current.url_options = { host: ActiveStorageのホスト先のURL }
