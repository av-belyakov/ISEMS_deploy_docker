FROM node:alpine AS temporary_image
WORKDIR /opt/isems-ui
RUN apk update && \
    apk add --no-cache git bash openssl && \
    git clone https://github.com/av-belyakov/ISEMS_UI.git /opt/isems-ui/
RUN mkdir /home/keys && \
    openssl genpkey -algorithm RSA -out /home/keys/rootCA.key && \
    openssl req -x509 -new -nodes -key /home/keys/rootCA.key -sha256 -days 1024 -out /home/keys/rootCA.pem -subj "/C=RU/ST=Moscow/L=Moscow/O=My Company/CN=example.org" && \
    openssl genpkey -algorithm RSA -out /home/keys/isems_ui_private_key.pem && \
    openssl req -new -key /home/keys/isems_ui_private_key.pem -out /home/keys/mydomain.com.csr -subj "/C=RU/ST=Moscow/L=Moscow/O=My Company/CN=example.org" && \
    openssl x509 -req -in /home/keys/mydomain.com.csr -CA /home/keys/rootCA.pem -CAkey /home/keys/rootCA.key -CAcreateserial -out /home/keys/isems_ui_cert.pem -days 1096 -sha256
RUN npm install --force && \
    npm run build-prod

FROM node:alpine
LABEL user="Artemy" application="ISEMS-UI"
RUN mkdir /home/keys
WORKDIR /opt/isems-ui
RUN apk update && \ 
    apk add --no-cache bash
COPY --from=temporary_image /opt/isems-ui/ .
COPY --from=temporary_image /home/keys/isems_ui_private_key.pem /home/keys/
COPY --from=temporary_image /home/keys/isems_ui_cert.pem /home/keys/
COPY config.json ./configure/
EXPOSE 8888
ENTRYPOINT [ "node", "app.js" ]