# Use a imagem oficial do Python
FROM python:3.12

# Define o diretório de trabalho no contêiner
WORKDIR /app

# Copia os arquivos de requisitos do projeto para o contêiner
COPY requirements.txt .

# Instala as dependências do projeto
RUN pip install --no-cache-dir -r requirements.txt

# Copia os arquivos do projeto para o contêiner
COPY . .

# Expõe a porta que o FastAPI irá rodar
EXPOSE 8000

# Inicia o servidor Uvicorn
CMD ["python", "app.py"]

