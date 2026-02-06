# Use a lightweight official Python image
FROM python:3.11-slim

# Set working directory inside the container
WORKDIR /stream_stock_data

# Install system dependencies required by pandas / yfinance
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency list first (better layer caching)
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY stream_stock_data.py .

# Run the producer
CMD ["python", "stream_stock_data.py"]