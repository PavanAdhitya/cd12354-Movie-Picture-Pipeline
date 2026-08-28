FROM python:3.10-slim
ENV FLASK_RUN_HOST=0.0.0.0

WORKDIR /

# Create a non-root user
RUN useradd -u 1000 -m app || true

# Install build dependencies required to compile uWSGI and other native extensions
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      build-essential \
      gcc \
      python3-dev \
      libpcre3 \
      libpcre3-dev \
      libssl-dev \
      libffi-dev \
      ca-certificates \
    && python -m pip install --upgrade pip pipenv \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/www/app && mkdir /app

COPY Pipfile* /app
RUN cd /app && pipenv install --system --deploy

COPY . /app

CMD uwsgi --http :5000 --master --enable-threads -s /var/www/app/app.sock --manage-script-name --mount /=app:app