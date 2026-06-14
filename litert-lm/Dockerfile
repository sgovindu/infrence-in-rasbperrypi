# Use a slim, stable 64-bit Python base image targeting ARM64
FROM python:3.11-slim-bookworm

# Prevent Python from writing pyc files and enable unbuffered logging
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install baseline OS system build tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    build-essential \
    gcc \
    g++ \
    libssl-dev \
    pkg-config \
    libvulkan1 \
    && rm -rf /var/lib/apt/lists/*

# Install Rust toolchain (required if pip forces an architecture source rebuild)
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable

# Optimize Rust compilation explicitly for the Raspberry Pi 5's Cortex-A76 processor
ENV RUSTFLAGS="-C target-cpu=cortex-a76"

# Set up working directory
WORKDIR /app

# Upgrade core python deployment packages
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# Install the LiteRT-LM framework packages from PyPI
# Note: "litert-lm" installs the CLI; "litert-lm-api" provides the Python runtime
RUN pip install --no-cache-dir litert-lm litert-lm-api

# Default command to verify the CLI tool installation inside the container
CMD ["litert-lm", "--help"]
