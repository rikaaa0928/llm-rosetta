# syntax=docker/dockerfile:1

ARG PYTHON_VERSION=3.12
ARG REGISTRY_MIRROR
FROM ${REGISTRY_MIRROR:-docker.io}/python:${PYTHON_VERSION}-alpine AS base

ARG PYPI_MIRROR
# Dev builds only (e.g. TestPyPI) — do not set for production images
ARG EXTRA_INDEX_URL
ARG LOCAL_WHEEL
ARG PACKAGE_VERSION

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN addgroup -g 1000 appgroup && \
    adduser -D -u 1000 -G appgroup appuser

COPY pyproject.toml README.md ./
COPY src/ ./src/

RUN if [ -n "$PYPI_MIRROR" ]; then \
        pip install -i "$PYPI_MIRROR" --no-cache-dir ".[gateway,profiling]"; \
    else \
        pip install --no-cache-dir ".[gateway,profiling]"; \
    fi && \
    pip uninstall -y pip setuptools 2>/dev/null || true

RUN mkdir -p /config && chown appuser:appgroup /config
VOLUME /config

COPY --chmod=755 docker/entrypoint.sh /entrypoint.sh

USER appuser
EXPOSE 8765

ENTRYPOINT ["/entrypoint.sh"]
CMD ["llm-rosetta-gateway", "--config", "/config/config.jsonc", "--host", "0.0.0.0"]
