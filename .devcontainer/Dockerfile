# ベースイメージとして、公式の .NET 8.0 SDK が入っているLinux環境を使用する
FROM mcr.microsoft.com/dotnet/sdk:8.0

# Linuxパッケージのアップデートと、Vue.jsの開発に必要な Node.js (バージョン20) をインストールする
RUN apt-get update && apt-get install -y curl \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# コンテナが起動したときの初期作業ディレクトリを /workspace に指定する
WORKDIR /workspace