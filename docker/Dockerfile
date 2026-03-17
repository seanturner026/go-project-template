# syntax=docker/dockerfile:1@sha256:4a43a54dd1fedceb30ba47e76cfcf2b47304f4161c0caeac2db1c61804ea3c91

# ------------------------------
FROM golang:1.25-alpine@sha256:d3f0cf7723f3429e3f9ed846243970b20a2de7bae6a5b66fc5914e228d831bbb AS build

RUN addgroup -S app_user && \
	adduser -S -u 1000 -g app_user app_user

WORKDIR /app
COPY go.mod go.sum ./

RUN --mount=type=cache,target=/go/pkg/mod \
	--mount=type=cache,target=/root/.cache/go-build \
	go mod download && \
	go mod verify

# https://stackoverflow.com/a/22276273/6004338
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod \
	--mount=type=cache,target=/root/.cache/go-build \
	go build -ldflags="-w -s" -o api ./cmd/api

# -----------------
FROM scratch AS api

# Copy user and SSL Certificates from build
# Certificates needed to prevent x509 unknown authority errors
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build /app/api /usr/local/bin/api
USER app_user

ENTRYPOINT ["/usr/local/bin/api"]

# ----------------------------
FROM golang:1.25-alpine@sha256:d3f0cf7723f3429e3f9ed846243970b20a2de7bae6a5b66fc5914e228d831bbb AS air

COPY --from=cosmtrek/air:latest@sha256:3def16c884b2b9409a4c7bb987cd76d656a12d0e0436fe8c583df93014e0356c /go/bin/air /usr/local/bin/air
COPY .air.toml .

CMD ["air"]
