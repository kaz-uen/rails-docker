## 開発環境構築

1. Docker Desktopをインストールする

    公式サイト：https://www.docker.com/products/docker-desktop/

2. ローカル環境に任意のプロジェクト用ディレクトリを作成する

    ※以降では、「myapp」というプロジェクト用ディレクトリを作成したものとして説明する。

4. ターミナルを開いて上記ディレクトリに移動する

5. ターミナルで下記コマンドを実行してリモートリポジトリをクローンする

```shell
git clone <リモートリポジトリURL>
```

5. Dockerfileとdocker-compose.ymlを作成し、それぞれ以下のコードをコピペする

```dockerfile:Dockerfile
FROM ruby:3.2.2
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    nodejs \
    postgresql-client \
    yarn
WORKDIR /myapp
COPY Gemfile Gemfile.lock /myapp/
RUN bundle install
```

```yaml:docker-compose.yml
version: '3'

volumes:
  db-data:

services:
  web:
    build: .
    ports:
      - '3000:3000'
    volumes:
      - '.:/myapp'
    environment:
      - 'DATABASE_PASSWORD=postgres'
    tty: true
    stdin_open: true
    depends_on:
      - db
    links:
      - db
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"

  db:
    image: postgres:12
    volumes:
      - 'db-data:/var/lib/postgresql/data'
    environment:
      - 'POSTGRES_USER=postgres'
      - 'POSTGRES_PASSWORD=postgres'

```

6. ターミナルで下記コマンドを実行してビルドする

```shell
docker-compose build
```
   
7. config/database.ymlに以下の内容を追加する

```yaml:database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db #追加
  user: postgres #追加
  port: 5432 #追加
  password: <%= ENV.fetch("DATABASE_PASSWORD") %> #追加
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
```

8. ターミナルで下記コマンドを実行する

```shell
docker-compose up -d
```

9. ターミナルで下記コマンドを実行する

```shell
docker-compose run web rails db:create
```

10. ターミナルでコマンドを実行する

```shell
docker-compose exec web bundle exec rails db:migrate
```

11. ブラウザで`http://localhost:3000`にアクセスしてページが表示されるか確認する。
