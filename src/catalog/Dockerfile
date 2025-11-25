# # ---------- Stage 1: Build ----------
# FROM golang:1.23-alpine AS builder

# # Install needed C build tools because go-sqlite3 needs CGO
# RUN apk add --no-cache gcc g++ musl-dev sqlite-dev

# WORKDIR /appsrc
# ENV GOPROXY=https://goproxy.io,direct
# ENV CGO_ENABLED=1
# ENV GOOS=linux
# ENV GOARCH=amd64

# COPY go.mod go.sum ./
# RUN go mod download

# COPY . .

# RUN go build -o main main.go


# # ---------- Stage 2: Runtime ----------
# FROM alpine:3.19

# # Install SQLite runtime libraries (required by go-sqlite3)
# RUN apk add --no-cache sqlite-libs

# WORKDIR /app
# COPY --from=builder /appsrc/main .

# EXPOSE 8080
# ENV GIN_MODE=release

# ENTRYPOINT ["/app/main"]


# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# We tell DNF not to install Recommends and Suggests packages, which are
# weak dependencies in DNF terminology, thus keeping our installed set of
# packages as minimal as possible.
RUN dnf --setopt=install_weak_deps=False install -q -y \
    git \
    golang \
    && \
    dnf clean all

RUN mkdir -p "${GOPATH}/src" "${GOPATH}/bin" /appsrc

WORKDIR /appsrc

ENV GOPROXY=https://goproxy.io,direct

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN go build -o main main.go

# Final stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

# We tell DNF not to install Recommends and Suggests packages, which are
# weak dependencies in DNF terminology, thus keeping our installed set of
# packages as minimal as possible.
RUN dnf --setopt=install_weak_deps=False install -q -y \
    shadow-utils \
    && \
    dnf clean all

RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

WORKDIR /app
USER appuser

COPY --chown=appuser:appuser --from=build-env /appsrc/main /app/

COPY ./ATTRIBUTION.md ./LICENSES.md

ENV GIN_MODE=release

ENTRYPOINT ["/app/main"]
