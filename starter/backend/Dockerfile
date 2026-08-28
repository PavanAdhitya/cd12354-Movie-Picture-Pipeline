FROM public.ecr.aws/docker/library/python:3.10-alpine
ENV FLASK_RUN_HOST=0.0.0.0

WORKDIR /

# Create a non-root user
RUN adduser -u 1000 -D app

# Install build dependencies required to compile uWSGI and other native extensions on Alpine
# Keep packages needed for building: build-base, python3-dev, musl-dev, libffi-dev, openssl-dev, pcre-dev
RUN apk update && \
    apk add --no-cache \
      build-base \
      python3-dev \
      musl-dev \
      libffi-dev \
      openssl-dev \
      pcre-dev \
      pcre \
      openssl \
      linux-headers \
    && python -m pip install --upgrade pip pipenv

RUN mkdir -p /var/www/app && mkdir /app

COPY Pipfile* /app
RUN cd /app && pipenv lock && pipenv install --system --deploy

COPY . /app

CMD uwsgi --http :5000 --master --enable-threads -s /var/www/app/app.sock --manage-script-name --mount /=app:app