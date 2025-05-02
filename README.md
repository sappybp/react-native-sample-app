# 📦 React Native + VOICEVOX 開発環境構築手順（Docker使用）

この手順書では、**React Native（Expo）と VOICEVOXエンジンを Docker上で動かす環境構築**の方法をまとめています。  
プロジェクト名は `palmo` を前提としています。

まずはpalmoディレクトリを作成し、その中でDockerfileとdocker-commpose.ymlを作り、以下内容をコピペ。palmo(React Nativeプロジェクト)はあとでコマンドで作成

---

## 📁 ディレクトリ構成（例）

palmo/  
├── docker-compose.yml  
├── Dockerfile  
└── palmo（ここにReact Nativeのプロジェクトが生成される）

---

## 🐳 1. Dockerfile（React Native用）

```Dockerfile
# DebianベースのNode環境（安定版）
FROM node:20-bullseye

# 作業フォルダ
WORKDIR /home/react-app

# bash やその他必要パッケージをインストール
RUN apt-get update && apt-get install -y \
  bash \
  curl \
  git \
  libgtk-3-0 \
  libxss1 \
  libasound2 \
  libnss3 \
  libx11-xcb1 \
  libxcomposite1 \
  libxdamage1 \
  libxtst6 \
  libgbm1 \
  xdg-utils \
  ffmpeg \
  && apt-get clean

# npmを最新化
RUN npm install -g npm

# expo を npx 経由で便利に叩けるようにする
RUN echo '#!/bin/sh\nnpx expo "$@"' > /usr/local/bin/expo && chmod +x /usr/local/bin/expo

# 開発でよく使うライブラリ
RUN npm install -g react-native-elements moment

# コンテナ起動時にbashへ
CMD ["/bin/bash"]

```
## 🧩 2. docker-compose.yml

```docker-compose.yml
version: "3.9"

services:
  palmo-app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: palmo-dev
    volumes:
      - .:/home/react-app
    ports:
      - "8081:8081"    # Metro bundler
      - "19000:19000"  # Expo Dev Tools
      - "19001:19001"  # Expo Go LAN
      - "19002:19002"  # Web UI
    working_dir: /home/react-app
    stdin_open: true
    tty: true
    command: /bin/bash
    depends_on:
      - voicevox

  voicevox:
    image: voicevox/voicevox_engine:nvidia-ubuntu20.04-latest
    container_name: voicevox-engine
    ports:
      - "50021:50021"
    environment:
      - VV_ENV=development
    deploy:
      resources:
        limits:
          memory: 2g

```

## 🚀 3. セットアップ手順
初回セットアップ時のみ、Dockerイメージをビルドして起動
```
docker compose up -d --build
```
開発用コンテナに入る
```
docker exec -it palmo-dev /bin/bash
```
React Native プロジェクトを作成（初回のみ）
```
npx create-expo-app@latest palmo
```
palmoディレクトリにReactNativeProjectが作成される

## ▶️ 4. アプリの起動
コンテナ内で以下を実行：

```
cd palmo
npm start
```
ブラウザで http://localhost:8081 にアクセスすれば開発用UIが開けます。

## 🔉 5. VOICEVOXエンジンの確認
以下のURLを開いてみてください：
```
http://localhost:50021/docs
```
VOICEVOXエンジンのAPIドキュメントが表示されれば成功です。

⏹️ 停止方法
```
# コンテナを停止（バックグラウンド実行中）
docker compose down
```