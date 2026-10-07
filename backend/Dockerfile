# Use Python 3.11 Slim
FROM python:3.11-slim

# 1. Install System Dependencies
RUN apt-get update && apt-get install -y \
    poppler-utils \
    libgl1 \
    libglib2.0-0 \
    wget \
    tar \
    xz-utils \
    && rm -rf /var/lib/apt/lists/*

# 2. Install FFmpeg
RUN wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz && \
    tar -xvf ffmpeg-release-amd64-static.tar.xz && \
    mv ffmpeg-*-amd64-static/ffmpeg /usr/bin/ && \
    mv ffmpeg-*-amd64-static/ffprobe /usr/bin/ && \
    rm -rf ffmpeg-*

# 3. Setup App & Permissions
WORKDIR /app
RUN useradd -m -u 1000 user
ENV PATH="/home/user/.local/bin:$PATH"

# 4. Copy Code
COPY --chown=user requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=user app ./app

# 5. Create Startup Script (Explicitly as ROOT first)
USER root
RUN echo '#!/bin/bash\n\
celery -A app.celery_app worker --loglevel=info --concurrency=1 &\n\
uvicorn app.main:app --host 0.0.0.0 --port 10000\n\
' > /app/start.sh && chmod +x /app/start.sh && chown user:user /app/start.sh

# Now safely switch to the restricted user right before running it
USER user

# 6. Start Command
CMD ["/app/start.sh"]
