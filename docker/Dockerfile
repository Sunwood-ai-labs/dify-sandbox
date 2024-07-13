# ビルドステージ
FROM python:3.10-slim AS builder

# 必要な依存関係をインストール（sudoを含む）
RUN apt-get update && apt-get install -y \
    sudo \
    pkg-config \
    libseccomp-dev \
    wget \
    xz-utils \
    gcc \
    g++ \
    git \
    make \
    cmake \
    curl

# Go言語のインストール
RUN wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && \
    rm go1.21.6.linux-amd64.tar.gz

# 環境変数の設定
ENV PATH="/usr/local/go/bin:${PATH}"

# Dify-Sandboxのソースコードを取得
WORKDIR /tmp
RUN git clone https://github.com/langgenius/dify-sandbox.git

# Dify-Sandboxのビルド
WORKDIR /tmp/dify-sandbox
# install.shとbuild_amd64.shを実行
RUN ./install.sh && ./build/build_amd64.sh # amd64の場合
# ARM64の場合は以下を使用
# RUN ./install.sh && ./build/build_arm64.sh

# 実行ステージ
FROM python:3.10-slim

# ビルドしたDify-Sandboxバイナリをビルドステージからコピー
COPY --from=builder /tmp/dify-sandbox/main /main

# 設定ファイルと依存関係をコピー
COPY conf/config.yaml /conf/config.yaml
COPY dependencies/python-requirements.txt /dependencies/python-requirements.txt

# Pythonの依存関係をインストール
RUN pip3 install -r /dependencies/python-requirements.txt

# Node.jsとcurlをインストール
RUN apt-get update && apt-get install -y wget xz-utils curl && \
    wget -O /opt/node-v20.11.1-linux-x64.tar.xz https://npmmirror.com/mirrors/node/v20.11.1/node-v20.11.1-linux-x64.tar.xz && \
    tar -xvf /opt/node-v20.11.1-linux-x64.tar.xz -C /opt && \
    ln -s /opt/node-v20.11.1-linux-x64/bin/node /usr/local/bin/node && \
    rm -f /opt/node-v20.11.1-linux-x64.tar.xz && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 実行権限を付与
RUN chmod +x /main

# コンテナ起動コマンド
ENTRYPOINT ["/main"]
