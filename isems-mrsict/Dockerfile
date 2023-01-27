FROM golang:1.19.4-alpine AS temporary_image
WORKDIR /go/src/
ENV PATH /usr/local/go/bin:$PATH
RUN apk update && \
    apk add --no-cache git openssl && \
    git clone https://github.com/av-belyakov/ISEMS-MRSICT.git /go/src/
RUN go build
RUN mkdir /home/keys && \
    openssl genpkey -algorithm RSA -out /home/keys/rootCA.key && \
    openssl req -x509 -new -nodes -key /home/keys/rootCA.key -sha256 -days 1024 -out /home/keys/rootCA.pem -subj "/C=RU/ST=Moscow/L=Moscow/O=My Company/CN=example.org" && \
    openssl genpkey -algorithm RSA -out /home/keys/isems_mrsict_private_key.pem && \
    openssl req -new -key /home/keys/isems_mrsict_private_key.pem -out /home/keys/mydomain.com.csr -subj "/C=RU/ST=Moscow/L=Moscow/O=My Company/CN=example.org" && \
    openssl x509 -req -in /home/keys/mydomain.com.csr -CA /home/keys/rootCA.pem -CAkey /home/keys/rootCA.key -CAcreateserial -out /home/keys/isems_mrsict_cert.pem -days 1096 -sha256

FROM alpine
LABEL user="Artemy" application="ISEMS-MRSICT"
WORKDIR /opt/isems-mrsict
RUN mkdir /opt/isems-mrsict/defaultsettingsfiles && \
    mkdir /home/logs && \
    mkdir /home/keys
COPY --from=temporary_image /go/src/ISEMS-MRSICT /opt/isems-mrsict/ 
COPY --from=temporary_image /go/src/defaultsettingsfiles/* /opt/isems-mrsict/defaultsettingsfiles/
COPY --from=temporary_image /home/keys/isems_mrsict_private_key.pem /home/keys/
COPY --from=temporary_image /home/keys/isems_mrsict_cert.pem /home/keys/
COPY config.json README.md /opt/isems-mrsict/
EXPOSE 13000
ENTRYPOINT [ "./ISEMS-MRSICT" ]