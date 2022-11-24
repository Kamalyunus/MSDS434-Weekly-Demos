FROM python:3.9-slim

WORKDIR /app
EXPOSE 8501

COPY . ./
RUN pip install -r requirements.txt

ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]