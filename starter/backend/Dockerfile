FROM public.ecr.aws/docker/library/python:3.10-alpine
ENV FLASK_RUN_HOST=0.0.0.0

WORKDIR /

# Create a non-root user
RUN adduser -u 1000 -D app

# Install only runtime dependencies (no C compilation needed)
RUN apk update && \
    apk add --no-cache \
      libffi \
      openssl \
      pcre \
    && python -m pip install --upgrade pip pipenv

RUN mkdir -p /var/www/app && mkdir /app

COPY Pipfile* /app
RUN cd /app && pipenv lock && pipenv install --system --deploy

COPY . /app

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--worker-class", "sync", "--access-logfile", "-", "app:app"]