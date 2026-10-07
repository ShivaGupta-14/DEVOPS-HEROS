FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app ./app

# run as non-root user
RUN useradd -m -u 1001 appuser
USER 1001

EXPOSE 5001

CMD ["python", "app/app.py"]
