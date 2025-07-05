# Dockerfile for Smart Router Agent (The Dispatcher)
FROM python:3.12.0-slim-buster

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# The command to run when the container starts.
# We use Uvicorn to run our FastAPI application.
# It listens on all interfaces (0.0.0.0) on port 5001.
# We will create the 'main.py' file it refers to in a later step.
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5001"]
