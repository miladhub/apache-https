# Apache with HTTPS example

This repo represents a simple example of how to expose a service on Apache using HTTPS.

These instructions were tested on MacOS. The relevant, app-specific information is contained in file `httpd-ssl.conf`:

    <Location /helk>
      ProxyPass "http://host.docker.internal:9176"
      ProxyPassReverse "http://host.docker.internal:9176"
    </Location>

The app will be available at <https://localhost/helk>.

## Creating the server private key and SSL certificate

Create the private key `server.key` and the self-signed SSL certificate `server.crt` as follows:

    openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
    openssl rsa -passin pass:x -in server.pass.key -out server.key
    openssl req -new -key server.key -out server.csr \
      -subj "/C=IT/ST=Italy/L=Bologna/O=FooBar inc/OU=Nerds/CN=foo.bar.baz"
    openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
    rm server.pass.key server.csr

## Setting up Apache with HTTPS

    docker run -dit --name apache -p 8080:80 -p 443:443 httpd:2.4
    docker cp httpd.conf apache:/usr/local/apache2/conf
    docker cp httpd-ssl.conf apache:/usr/local/apache2/conf/extra
    docker cp server.crt apache:/usr/local/apache2/conf
    docker cp server.key apache:/usr/local/apache2/conf
    docker exec apache apachectl restart

## References

- <https://devcenter.heroku.com/articles/ssl-certificate-self>
- <https://hub.docker.com/_/httpd>
- <https://docs.docker.com/docker-for-mac/networking/> (`host.docker.internal`)

