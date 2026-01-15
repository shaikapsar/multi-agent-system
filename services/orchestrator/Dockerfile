# Stage 1: Builder
# Use a slim Python 3.13+ image for the build environment
FROM python:3.13-slim AS builder

# Install uv by copying it from the official image (fastest method)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

# Set working directory
WORKDIR /app

# Enable bytecode compilation for faster startup in production
ENV UV_COMPILE_BYTECODE=1
# Ensure uv uses the system Python and doesn't try to manage its own
ENV UV_PYTHON_DOWNLOADS=never

# 1. Copy only dependency files first to leverage Docker layer caching
COPY pyproject.toml ./

# 2. Install dependencies into a virtual environment
# Using --no-install-project ensures we only build the "heavy" dependencies first --frozen
RUN uv sync  --no-install-project --no-dev

# Stage 2: Runtime
# Use the same slim base to ensure compatibility with compiled artifacts
FROM python:3.13-slim AS runtime

WORKDIR /app

# Copy the pre-built virtual environment from the builder
COPY --from=builder /app/.venv /app/.venv

# Ensure the virtual environment's binaries are in the PATH
ENV PATH="/app/.venv/bin:$PATH"

# Copy the rest of your CrewAI source code
# (This layer changes frequently, so we put it near the end)
COPY src/ ./src/

# Security: Run as a non-root user (Standard 2026 best practice)
RUN useradd -m crewuser && chown -R crewuser /app
# Ensure the src directory is created and owned before switching users
RUN mkdir -p /app/src && chown -R crewuser /app/src
USER crewuser


# Launch the orchestrator
CMD ["python", "src/orchestrator/__init__.py"]

