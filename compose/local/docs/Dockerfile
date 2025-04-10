# Define an alias for the specific python version used in this file.
FROM python:3.11-slim-bookworm as python

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install --no-install-recommends -y \
        build-essential \
        libpq-dev \
        gettext \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Python build stage
FROM python as python-build-stage

# Ensure the latest package updates in the build stage
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install --no-install-recommends -y \
    build-essential \
    libpq-dev \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

ENV PYTHONDONTWRITEBYTECODE 1

# Requirements are installed here to ensure they will be cached.
COPY ./requirements /requirements

# Create python dependency wheels
RUN pip wheel --no-cache-dir --wheel-dir /usr/src/app/wheels  \
  -r /requirements/local.txt -r /requirements/production.txt \
  && rm -rf /requirements

# Python 'run' stage
FROM python as python-run-stage

# Ensure the latest package updates in the run stage
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install --no-install-recommends -y \
  # To run the Makefile
  make \
  # psycopg2 dependencies
  libpq-dev \
  # Translations dependencies
  gettext \
  # Uncomment below lines to enable Sphinx output to latex and pdf
  # texlive-latex-recommended \
  # texlive-fonts-recommended \
  # texlive-latex-extra \
  # latexmk \
  # cleaning up unused files
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/*

# Copy python dependency wheels from python-build-stage
COPY --from=python-build-stage /usr/src/app/wheels /wheels

# Use wheels to install python dependencies
RUN pip install --no-cache /wheels/* \
  && rm -rf /wheels

COPY ./compose/local/docs/start /start-docs
RUN sed -i 's/\r$//g' /start-docs
RUN chmod +x /start-docs

WORKDIR /docs