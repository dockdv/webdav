# HTTPS WebDAV 

---

This image is based on the [Apache HTTP Server] image. 

HTTPS is the only protocol enabled. Default authentication is Basic.

### Volumes

There are two volumes which need to be defined while starting a service.

- /config : The folder where configuration files are stored. If not defined, this folder will be created in the container.
- /dav/data : The path to the webdav data folder.

WebDAV server can be accessed using the /webdav subdirectory.

    e.g., https://www.example.com/webdav

### Apache Server Configuration

When the container is first created, configuration files are copied to the /config path. The files can be freely modified. It may be necessary to restart the container after a change.

### SSL Certificates

When the container is first created, OpenSSL in the container creates the certificate files. 

```
openssl req -x509 -newkey rsa:4096 -days 9999 -nodes \
  -keyout $CONFIGPATH/server.key \
  -out $CONFIGPATH/server.crt \
  -subj "/CN=${SERVER_NAME:-selfsigned}"
```

These files are then copied to /config folder. If you want to use your own certificates, you can replace them keeping the names of the files.

### Environment variables

Environment variables are optional. If not defined, default values are used.

```
SERVER_NAME : The default is "localhost".
USERNAME : The default username is "webdav".
PASSWORD : The default password is "webdav".
```

### Usage

##### - Docker Run:
```
docker run -d \
    -v /srv/config:/config \
    -v /srv/data:/dav/data \
    -e SERVER_NAME=www.example.com:443 \
    -e USERNAME=webdav \
    -e PASSWORD=webdav \
    -p 8443:443 \
    --restart=unless-stopped \
    --name=webdav \
    dockdv/webdav
```

##### - Docker Compose:

```
services:
  webdav:
    image: dockdv/webdav
    restart: unless-stopped
    ports:
      - "8443:443"
    environment:
      SERVER_NAME: www.example.com:443
      USERNAME: webdav
      PASSWORD: webdav
    volumes:
      - ./config: /config
      - ./data: /dav/data
```

If macvlan network is preferred, docker container can be accessed without the requirement of port forwarding. This option should be carefully considered as all ports will be open.

```
services:
  webdav:
    image: dockdv/webdav
    restart: unless-stopped
    environment:
      SERVER_NAME: www.example.com:443
      USERNAME: webdav
      PASSWORD: webdav
    volumes:
      - ./config: /config
      - ./data: /dav/data
    networks:
      macvlan:
        ipv4_address: 10.100.10.10
      
networks:
  macvlan:
    external: true
```


### Source

https://github.com/dockdv/webdav

https://hub.docker.com/r/dockdv/webdav

[Apache HTTP Server]: <https://hub.docker.com/_/httpd>
