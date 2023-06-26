# ベースイメージの指定
FROM python:3.10

# 作業ディレクトリを設定
WORKDIR /app

# 必要なファイルをコピー
COPY requirements.txt .

# 依存関係をインストール
RUN apt-get update && \
	apt-get -y upgrade

# 依存関係をインストール
RUN pip install --no-cache-dir -r requirements.txt
	
# アプリケーションのコードをコピー
COPY app.py .

# ポートのエクスポート
EXPOSE 8501

# コンテナの起動時に実行するコマンド
CMD ["streamlit", "run", "app.py", "--server.address", "0.0.0.0", "--server.port", "8501"]
 # "--server.baseUrlPath","/e/"]
