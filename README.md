Apache with HTTPS example
===

This project represents a simple example of how to expose a service on Apache using HTTPS.

These instructions were tested on MacOS. The app behind Apache is just a demo
server that dumps all HTTP headers on the console. The app-specific configuration
is contained in fil `httpd-ssl.conf`:

```
<Location /app>
  ProxyPass "http://host.docker.internal:9176"
  ProxyPassReverse "http://host.docker.internal:9176"
</Location>
```

You can replace it by pointing at your app.

## Creating the server private key and SSL certificate

Create the private key and the self-signed SSL certificate using [mkcert](https://github.com/FiloSottile/mkcert):

```
mkcert --install
mkcert localhost
openssl x509 -outform der -in localhost.pem -out localhost.crt
openssl pkey -in localhost-key.pem -out localhost.key
```

## Setting up Apache with HTTPS

```
docker rm -f apache-https
docker run -dit --name apache-https -p 443:443 httpd:2.4.49
docker cp httpd.conf apache-https:/usr/local/apache2/conf
docker cp httpd-ssl.conf apache-https:/usr/local/apache2/conf/extra
docker cp localhost.pem apache-https:/usr/local/apache2/conf/server.crt
docker cp localhost.key apache-https:/usr/local/apache2/conf/server.key
docker restart apache-https
```

# Installing the dummy app

Issue the following command:

```
nc -l -p 9176
```

Or this is you have the BSD version of Netcat:

```
nc -l 9176
```

If you navigate to the app at <https://localhost/app> the page will stay there hanging, but on the console
you can see all HTTP headers coming through in clear text (i.e., unencrypted), which means that the Apache HTTPS
proxy is working:

```
GET / HTTP/1.1
Host: host.docker.internal:9176
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-gb
Accept-Encoding: gzip, deflate, br
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1 Safari/605.1.15
X-Forwarded-For: 172.17.0.1
X-Forwarded-Host: localhost
X-Forwarded-Server: www.example.com
Connection: Keep-Alive
```

# Checking that HTTP/2 works

```
$ curl --http2 https://localhost/ -I
HTTP/2 200 
date: Tue, 05 Oct 2021 20:10:02 GMT
server: Apache/2.4.49 (Unix) OpenSSL/1.1.1d
last-modified: Mon, 11 Jun 2007 18:53:14 GMT
etag: "2d-432a5e4a73a80"
accept-ranges: bytes
content-length: 45
content-type: text/htm
```

## References

* <https://devcenter.heroku.com/articles/ssl-certificate-self>
* <https://hub.docker.com/_/httpd>
* <https://docs.docker.com/docker-for-mac/networking/> (`host.docker.internal`)
* <https://httpd.apache.org/docs/2.4/howto/http2.html>
* <https://www.inmotionhosting.com/support/server/apache/enable-http-2-apache/>
