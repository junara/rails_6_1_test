# Rails 6.1の最小のdocker

## 参考
https://docs.docker.jp/compose/rails.html

versionを書き換えるだけ

## 手順

```
touch Dockerfile

```

```
FROM ruby:2.7.1
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs default-mysql-client
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
ADD . /myapp
```

```
touch Gemfile 
```

```
source 'https://rubygems.org'
gem 'rails', '~> 6.1.0'
```


```
touch Gemfile.lock
```


```
touch docker-compose.yml
```

```
version: '3'
services:
  db:
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=password
    volumes:
      - ./tmp/mysql:/var/lib/mysql
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

* 最小限のrailsをインストール

```
docker-compose run web rails new . --database=mysql --minimal    
```

* config/database.ymlを編集

```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password
  host: db

development:
  <<: *default
  database: myapp_development

test:
  <<: *default
  database: myapp_test

production:
  <<: *default
  database: myapp_production
  username: myapp
  password: <%= ENV['MYAPP_DATABASE_PASSWORD'] %>
```

## Test

```
docker-compose exec web rails db
# password と入力する
# mysqlに接続できればOK
```
