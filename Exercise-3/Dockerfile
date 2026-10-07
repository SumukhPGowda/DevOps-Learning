FROM python:3.11-slim

WORKDIR /app

COPY ex3-flash-sale.py .

RUN pip install --no-cache-dir flask

EXPOSE 5000

CMD ["python", "ex3-flash-sale.py"]
