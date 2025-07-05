# Dockerfile for Django App (The Command Center)
# Start from an official Python base image. Using a specific version ensures consistency.
FROM python:3.12.0-slim-buster

# Prevent Python from writing pyc files to disc
ENV PYTHONDONTWRITEBYTECODE 1
# Ensure that Python output is sent straight to terminal (e.g. for Docker logs)
ENV PYTHONUNBUFFERED 1

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY requirements.txt .
# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# The command to run when the container starts.
# We use Gunicorn, a production-ready WSGI server, to run our Django app.
# It will listen on all network interfaces (0.0.0.0) on port 8000.
# We will need to create a 'janus_project' directory inside 'django_app' for this to work.
# CMD ["gunicorn", "--bind", "0.0.0.0:8000", "janus_project.wsgi:application"]
