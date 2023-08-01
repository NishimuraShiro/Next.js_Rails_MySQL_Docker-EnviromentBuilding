# Next.js Rails MySQL Dockerを使った環境構築
※Next.js Rails MySQL Docker がインストール済みが条件です。<br>

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
FROM ruby:latest

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN gem install bundler

RUN bundle install

COPY . .

CMD ["rails", "server", "-b", "0.0.0.0"]
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
[+] Running 3/3
 ✔ Container dcteamf-a-mysql-1   Started                                                                                                      1.7s 
 ✔ Container dcteamf-a-rails-1   Started                                                                                                      1.5s 
 ✔ Container dcteamf-a-nextjs-1  Started  
```
上のような結果が出たらビルド成功です。<br>

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
上のような結果が出てきたらサーバー起動成功です。<br>
### Rails
` ~$ cd ../ ` <br>
` ~$ cd backend `<br>
` ~$ docker-compose exec rails bash `<br><br>
ターミナルがこのようになったら仮想環境に入れた合図です。<br>
` root@72a63eea44a1:/app# `<br><br>
次に以下のコードを叩くとNext.jsのサーバーが起動されます。<br>
