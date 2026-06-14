FROM ubuntu:24.04 AS builder

RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN git clone https://github.com/ggerganov/llama.cpp.git

WORKDIR /app/llama.cpp
# Compile specifically with ARM NEON acceleration optimized for Pi CPU architectures
RUN cmake -B build -DBUILD_SHARED_LIBS=OFF -DGGML_NATIVE=ON && cmake --build build --config Release

FROM ubuntu:24.04
RUN apt-get update && apt-get install -y libgomp1
WORKDIR /workspace
# Copy only the compiled server binary to keep the image tiny (< 150MB)
COPY --from=builder /app/llama.cpp/build/bin/llama-server /usr/local/bin/llama-server

EXPOSE 8080
ENTRYPOINT ["llama-server"]
