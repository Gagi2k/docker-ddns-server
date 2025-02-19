FROM golang:latest as builder

ENV GO111MODULE=on
ENV GOPATH=/root/go
RUN mkdir -p /root/go/src
COPY dyndns /root/go/src/dyndns
RUN cd /root/go/src/dyndns && go mod download && GOOS=linux GOARCH=amd64 go build -o /root/go/bin/dyndns && go test -v

FROM debian:buster-slim

RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
	apt-get install -q -y bind9 dnsutils curl rsyslog && \
	apt-get clean

RUN chmod 770 /var/cache/bind
COPY deployment/setup.sh /root/setup.sh
RUN chmod +x /root/setup.sh
COPY deployment/named.conf.options /etc/bind/named.conf.options

WORKDIR /root
COPY --from=builder /root/go/bin/dyndns /root/dyndns
COPY dyndns/views /root/views
COPY dyndns/static /root/static

RUN mkdir -p /var/cache/bind/database/
RUN ln -s /var/cache/bind/database /root/database
VOLUME /var/cache/bind/

EXPOSE 53 8080
CMD ["sh", "-c", "/root/setup.sh ; service rsyslog start; service bind9 start ; /root/dyndns"]
