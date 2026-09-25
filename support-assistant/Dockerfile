
```dockerfile
# Use a lightweight Python image
FROM python:3.11-slim

# Prevent Python from creating .pyc files
# and ensure logs are displayed immediately
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set the working directory inside the container
WORKDIR /app

# Install system packages required by some Python ML/vector packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency file if you create one
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Copy the support assistant application
COPY . .

# The assignment requires MOCK_LLM to default to the offline mode
ENV MOCK_LLM=1

# FastAPI will be served on port 7860
EXPOSE 7860

# Start the FastAPI application
CMD ["uvicorn", "support_assistant:app", "--host", "0.0.0.0", "--port", "7860"]
```
