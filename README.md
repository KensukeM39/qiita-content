Editing now...

# qiita-content

WSL2 の Ubuntu 24.04 の上で Docker で管理する環境を構築する手順

## 前提条件

以下の環境を前提に話を進めていきます．

- Docker Desktop: インストール済み．
- WSL2 + Ubuntu 24.04: セットアップ済み．
- Cursor: 今回使用するエディタ．Terminal も Cursor 上で操作していく．

以降の内容は，Qiita CLI の README.md に準拠したものにしています．
ChatGPT 4o を利用して環境を構築したため，一部不備や誤りがある可能性があります．ご了承ください．

## STEP 1 作業フォルダの準備 (WSL2 内)

1. GitHub 上で，Qiita の記事管理用のリポジトリを新規作成する．
   リポジトリ名は `qiita-content` として話を進めるため，自身の設定したリポジトリ名に置き換えてコマンドを実行してください．
2. Cursor を開く
3. 左下にある Remote Host(<>のマーク)をクリック
4. `Connect to WSL2 using Distro...`を選択
5. `Docker-Desktop`ではなく`Ubuntu-24.04`を選択
6. Cursor 上の Terminal を開き，(1)で作成したリモートリポジトリをローカルディレクトリに`git clone`する
   僕の場合は，`\\wsl.localhost\Ubuntu-24.04\home\ken\github.com\`配下で管理しようと思います．
   ```bash
   git clone <あなたのリモートリポジトリのHTTPS>
   # 僕の場合：https://github.com/KensukeM39/qiita-content.git
   ```
   clone したローカルリポジトリ内に移動
   ```bash
   cd qiita-content
   ```
   この場所が「Qiita 記事を書くフォルダ」になります．

## STEP2 設定ファイルの作成

7.  `...\github.com\qiita-content\`配下に以下の 3 つのファイルを作成する．
    a. Dockerfile (整備する環境の設定ファイル)
    Cursor 上で`Dockerfile`を開き，編集&保存．
    `bash
cursor Dockerfile
`

    ````Dockerfile
    FROM node:20

        ENV TZ=Asia/Tokyo
        WORKDIR /app

        RUN apt-get update && apt-get install -y git
        RUN npm install @qiita/qiita-cli --save-dev

        CMD [ "bash" ]
        ```

    b. docker-compose.yml (起動方法の設定ファイル)
    Cursor 上`docker-compose.yml`を開き，編集&保存．
    `bash
    cursor Dockerfile
    `
    `yaml
    services:
        qiita:
            build: .
            container_name: qiita-cli
            volumes:
            - .:/app
            tty: true
    `
    c. .gitignore (GitHub のリポジトリで管理しないファイルの設定ファイル)
    Cursor 上`.gitignore`を開き，編集&保存．
    `bash
    cursor .gitignore
    `
    ```gitignore
    # Qiita CLI 関連
    qiita.config.json
    credentials.json
    .qiita-cache/
    .remote

    # Node.js
    node_modules/
    npm-debug.log*
    yarn-debug.log*
    yarn-error.log*

    # Docker
    .env
    *.log

    # VSCodeや一時ファイル
    .vscode/
    .idea/
    *.swp
    .DS_Store
    Thumbs.db
    ````

## STEP 4 Docker のビルド

8. Docker 環境を構築する．
   ```bash
   docker-compose build
   ```
   これにより，Node.js + Qiita CLI が入ったイメージが完成．

## STEP 5 Docker コンテナを起動

9. Docker コンテナを起動する．

   ```bash
   docker-compose run qiita
   ```

10. Qiita CLI のバージョン確認
    (9)のコマンドで Docker コンテナを起動したことにより，Terminal 上で以下のように表示されます．

    ```bash
    root@xxxx:/app#
    ```

    Qiita CLI の`npx`コマンドは，Docker コンテナ上でのみ実行可能です．
    終了したい場合は，以下のコマンドを入力してください．

    ```bash
    root@xxxx:/app# exit
    ```

    Qiita CLI のバージョンを確認します．

    ```bash
    root@xxxx:/app# npx qiita version
    # 僕の場合は 1.6.2 と返ってきました．
    ```

11. Qiita CLI のアップデート
    今後 Qiita CLI を運用するにあたって，アップデートするときは以下のコマンドを実行してください．
    ```bash
    root@xxxx: /app# npm install @qiita/qiita-cli@latest
    ```

## Qiita CLI のセットアップ

12. init コマンドを実行
    本来であれば，以下のコマンドを実行することで`.gitignore`が作成されます．
    既に作成している`.gitignore`には，提供される`.gitignore`の内容を含んでいるため，問題ありません．
