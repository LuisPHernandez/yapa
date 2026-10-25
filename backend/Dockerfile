FROM python:3.13-slim-trixie

# Native libraries GeoDjango needs: GDAL, GEOS, PROJ
RUN apt-get update \
    && apt-get install -y --no-install-recommends binutils gdal-bin libproj-dev \
    && rm -rf /var/lib/apt/lists/*

# The uv program, pinned to the version we use locally
COPY --from=ghcr.io/astral-sh/uv:0.13.5 /uv /uvx /bin/

ENV UV_PROJECT_ENVIRONMENT=/opt/venv \
    UV_PYTHON_DOWNLOADS=never \
    PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

WORKDIR /app

# Dependencies first, so code changes don't reinstall them
COPY pyproject.toml uv.lock .python-version ./
RUN uv sync --frozen

COPY . .

EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
