# Usa imagem oficial do Python
FROM python:3.11-slim

# Define o diretório de trabalho
WORKDIR /app

# Instala dependências do sistema necessárias para psycopg2
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev && \
    rm -rf /var/lib/apt/lists/*

# Copia o arquivo de requirements
COPY requirements.txt .

# Instala as dependências Python
RUN pip install --no-cache-dir -r requirements.txt

# Copia o código da aplicação
COPY . .

# Cria o diretório para métricas do Prometheus
RUN mkdir -p /tmp/prometheus_multiproc

# Expõe a porta da aplicação
EXPOSE 8000

# Define variáveis de ambiente padrão
ENV PYTHONUNBUFFERED=1
ENV PROMETHEUS_MULTIPROC_DIR=/tmp/prometheus_multiproc

# Comando para executar a aplicação com Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--timeout", "120", "main:app"]