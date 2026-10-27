## How to make a curl without curl

Minimal containers do not always include tools such as `curl` or `wget`.

If the shell supports `/dev/tcp`, it is possible to manually open a TCP connection and send an HTTP request.

### Define the target

```bash
HOST="127.0.0.1"
PORT="9000"
ENDPOINT="/health/ready"
```

### Open a TCP connection

```bash
exec 3<>/dev/tcp/${HOST}/${PORT}
```

This opens a TCP connection to `${HOST}:${PORT}` and assigns it to file descriptor `3`.

The `<>` operator opens the descriptor for both reading and writing.

### Send an HTTP request

```bash
printf "GET ${ENDPOINT} HTTP/1.1\r\nHost: ${HOST}\r\nConnection: close\r\n\r\n" >&3
```

`>&3` sends the HTTP request through the TCP connection associated with file descriptor `3`.

### Read the response

```bash
cat <&3
```

`<&3` reads the response from the TCP connection and `cat` prints it to stdout.

### Complete example

```bash
HOST="127.0.0.1"
PORT="9000"
ENDPOINT="/health/ready"

exec 3<>/dev/tcp/${HOST}/${PORT} && \
printf "GET ${ENDPOINT} HTTP/1.1\r\nHost: ${HOST}\r\nConnection: close\r\n\r\n" >&3 && \
cat <&3
```

This is roughly equivalent to:

```bash
curl "http://${HOST}:${PORT}${ENDPOINT}"
```

### Use it as a health check

Instead of printing the response, we can check the HTTP status:

```bash
HOST="127.0.0.1"
PORT="9000"
ENDPOINT="/health/ready"

exec 3<>/dev/tcp/${HOST}/${PORT} && \
printf "GET ${ENDPOINT} HTTP/1.1\r\nHost: ${HOST}\r\nConnection: close\r\n\r\n" >&3 && \
grep -q "200 OK" <&3
```

If `200 OK` is found, `grep` returns exit code `0`.

> **Note:** `/dev/tcp` is a shell feature and is not available in every shell.