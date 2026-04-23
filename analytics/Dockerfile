# Use official Python slim image matching the app's requirements
FROM python:3.11-slim

# Set working directory inside the container
WORKDIR /app

# Install system dependencies required to build psycopg2
RUN apt-get update -y && apt-get install -y build-essential libpq-dev && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip setuptools wheel && pip install -r requirements.txt

# Copy application code
COPY . .

# Expose the port the app runs on
EXPOSE 5153

# Run the application
CMD ["python", "app.py"]
