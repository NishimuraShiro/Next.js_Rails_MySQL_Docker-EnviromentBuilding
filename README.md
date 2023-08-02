# Next.js Rails MySQL Dockerを使った環境構築
※Next.js Rails MySQL Docker がインストール済みが条件です。<br>
## バージョン確認
- Next.js： @13.4.12
- Ruby： @3.1.2
- Rails： @7.0.6
- MySQL： @8.0.32
- Docker： @24.0.2

## 構築ディレクトリの作成
``` ~$ mkdir sample-environment ```<br>
sample-environmentというフォルダを作成します。<br>

## 構築ディレクトリ内でのフォルダ作成
sample-environmentフォルダの中に２つのフォルダ（frontendとbackend）を作成します。<br>
``` ~$ npx create-next-app frontend ```<br>
``` ~$ rails new backend ```<br>
これによって、Next.jsのfrontendフォルダ、railsのbackendフォルダができるはずです。<br>

## Dockerfileの作成
次にDockerfileの作成をします。<br>
初めに、**frontendフォルダ直下とbackendフォルダ直下**にDockerfileを用意します。<br><br>

### frontendフォルダのDockerfile
frontend > Dockerfile
``` 
FROM node:latest

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "run", "dev"]
```
### backendフォルダのDockerfile
backend > Dockerfile
```
FROM ruby:3.1.2

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN gem install bundler

RUN bundle install

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

COPY . .

CMD ["rails", "server", "-b", "0.0.0.0"]
```
## entrypoint.shの作成
次に、entrypoint.shを**backend直下**に用意します。<br><br>
entrypoint.sh<br>
```
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```
## docker-compose.ymlの作成
次に、docker-compose.ymlを**sample-environment直下**に用意します。<br><br>
docker-compose.yml<br>
```
version: "3"
services:
  nextjs:
    build:
      context: ./frontend # Next.jsアプリケーションのパスを指定
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app # Next.jsアプリケーションのパスを指定
    depends_on:
      - mysql

  rails:
    build:
      context: ./backend # Railsアプリケーションのパスを指定
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    volumes:
      - ./backend:/app # Railsアプリケーションのパスを指定
    depends_on:
      - mysql
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3001 -b '0.0.0.0'"

  mysql:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: database_name
      MYSQL_USER: username
      MYSQL_PASSWORD: password
```
## Dockerの実行
順に以下をターミナルで実行します。<br>
` ~$ docker-compose build `<br>
` ~$ docker-compose up -d `<br><br>
```
[+] Running 4/4
 ✔ Network dcteamf-a_default     Created                                                                                                                                                                                         0.2s 
 ✔ Container dcteamf-a-mysql-1   Started                                                                                                                                                                                         2.9s 
 ✔ Container dcteamf-a-rails-1   Started                                                                                                                                                                                         4.1s 
 ✔ Container dcteamf-a-nextjs-1  Started    
```
上のような結果が出たらビルド成功です。<br><br>

※※<br>
```
------                                                                                                                                                                                                              
 > [rails 5/6] RUN bundle install:
1.260 Your Ruby version is 3.2.2, but your Gemfile specified 3.1.2
------
failed to solve: process "/bin/sh -c bundle install" did not complete successfully: exit code: 18
```
上記のようなエラーが出てしまったら、Gemfileの4行目を以下のように編集しましょう。<br>
` ruby "3.2.2" `<br>
そして、再度Dockerの実行を試してみましょう。

## Docker環境に入る
ビルドができたら、次にDocker環境に入って実行をしていきます。<br>
### Next.js
` ~$ cd frontend `<br>
` ~$ docker-compose exec nextjs bash `<br><br>
ターミナルがこのようになったら仮想環境に入れた合図です。<br>
` root@5b5f5587cf95:/app# `<br><br>
次に以下のコードを叩くとNext.jsのサーバーが起動されます。<br>
` root@5b5f5587cf95:/app# npm run dev `<br><br>

```
> project-frontend@0.1.0 dev
> next dev

- warn Port 3000 is in use, trying 3001 instead.
- ready started server on 0.0.0.0:3001, url: http://localhost:3001
- event compiled client and server successfully in 265 ms (20 modules)
- wait compiling...
- event compiled client and server successfully in 280 ms (20 modules)
```
上のような結果が出てきたらサーバー起動成功です。<br><br>
確認として` localhost:3000 `にアクセスしてみましょう。<br>
以下の図のようになれば成功です。<br>
<img width="1440" alt="スクリーンショット 2023-08-02 13 13 40" src="https://github.com/NishimuraShiro/Next.js_Rails_MySQL_Docker-EnviromentBuilding/assets/73762800/1d50003b-5586-4c15-913a-81f6401b3f6c">

### Rails
` ~$ cd ../ ` <br>
` ~$ cd backend `<br>
` ~$ docker-compose exec rails bash `<br><br>
ターミナルがこのようになったら仮想環境に入れた合図です。<br>
` root@72a63eea44a1:/app# `<br><br>
次に以下のコードを叩くとNext.jsのサーバーが起動されます。<br>
` root@72a63eea44a1:/app# rails s `<br><br>
```
=> Booting Puma
=> Rails 7.0.6 application starting in development 
=> Run `bin/rails server --help` for more startup options
A server is already running. Check /app/tmp/pids/server.pid.
Exiting
```
上のような結果が出てきたらサーバー起動成功です。一見うまく行っていないかと思いますが、サーバーを起動してみると確認できます。<br><br>
確認として` localhost:3001 `にアクセスしてみましょう。<br>
以下の図のようになれば成功です。<br>
<img width="1440" alt="スクリーンショット 2023-08-02 13 14 02" src="https://github.com/NishimuraShiro/Next.js_Rails_MySQL_Docker-EnviromentBuilding/assets/73762800/47da066a-0c12-4c14-99f3-12b6cd09f21f">

### MySQL
` ~$ docker-compose exec mysql bash `<br><br>
ターミナルがこのようになったら仮想環境に入れた合図です。<br>
` bash-4.4# `<br><br>
次に以下のコードを叩くとNext.jsのサーバーが起動されます。<br>
` bash-4.4# mysql -u root -p `<br>
` Enter password: root `<br><br>
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.34 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
上のような結果が出ればMySQLにログイン成功です。
