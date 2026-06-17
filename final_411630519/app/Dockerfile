FROM python:3.12-slim

# 要求 2：建立非 root 使用者 appuser
RUN useradd -m -u 1000 appuser

WORKDIR /app

# 要求 1：依 layer 快取原則排序 (變動少的放前面)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 變動頻繁的程式碼放後面
COPY . .

# 修正權限並切換至非 root 使用者
RUN chown -R appuser:appuser /app
USER appuser

# 要求：改聽 8080
EXPOSE 8080

# 要求 3：使用 exec form 的 CMD
CMD ["python", "app.py"]
