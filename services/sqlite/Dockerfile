FROM alpine:3.18
RUN apk add --no-cache sqlite
VOLUME ["/data"]
WORKDIR /data
CMD ["sh", "-c", "touch /data/sqlite.db || true; exec tail -f /dev/null"]
