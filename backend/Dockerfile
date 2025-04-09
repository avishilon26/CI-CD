# backend/Dockerfile

FROM python:3.11-slim

# Install dependencies
WORKDIR /app

# Install system dependencies
#RUN apt-get update && apt-get upgrade -y && apt-get install -y postgresql-client


# Copy requirements and install
COPY requirements.txt requirements.txt
#RUN pip install --no-cache-dir -r requirements.txt
RUN pip install --trusted-host=pypi.org --trusted-host=files.pythonhosted.org jinja2
RUN pip install --no-cache-dir -r requirements.txt --timeout=100 --retries=10


# Copy the application code
COPY . .

# Set environment variables
ENV FLASK_APP=app.py
ENV FLASK_ENV=development

# Command to run the application
CMD ["flask", "run", "--host", "0.0.0.0"]
