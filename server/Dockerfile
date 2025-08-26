# Use a Python image with uv pre-installed
FROM ghcr.io/astral-sh/uv:python3.13-bookworm-slim

# Install the project into /app
WORKDIR /app

# Enable bytecode compilation and copy mode
ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy

# Database URL (runtime)
ARG database_url
ENV database_url=${database_url}

# Copy dependency files first
COPY pyproject.toml ./
COPY uv.lock* ./

# Install the project's dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    if [ -f uv.lock ]; then \
        echo "✅ Using uv.lock for reproducible dependencies"; \
        uv sync --no-install-project --no-dev; \
    else \
        echo "⚠️ uv.lock not found, resolving from pyproject.toml"; \
        uv export --no-dev --format=requirements.txt | uv pip install --system -r /dev/stdin; \
    fi

# Add the rest of the project source code and install it
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --no-dev

# Extract, init, and compile translations
RUN uv run pybabel extract -F babel.cfg -o messages.pot . \
 && uv run pybabel init -i messages.pot -d lang -l zh_CN || true \
 && uv run pybabel compile -d lang -l zh_CN

# Install netcat + curl for DB connectivity checks
RUN apt-get update && apt-get install -y curl netcat-openbsd \
 && rm -rf /var/lib/apt/lists/*

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"

# Copy and make the start script executable
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh

# Reset entrypoint, don’t invoke uv
ENTRYPOINT []

EXPOSE 5678

# Use the start script
CMD ["/app/start.sh"]