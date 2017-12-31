# Alpine Nginx

## What is Nginx?


![nginx logo](https://nginx.org/nginx.png)


Nginx (pronounced "engine-x") is an open source reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols, as well as a load balancer, HTTP cache, and a web server (origin server). The nginx project started with a strong focus on high concurrency, high performance and low memory usage. It is licensed under the 2-clause BSD-like license and it runs on Linux, BSD variants, Mac OS X, Solaris, AIX, HP-UX, as well as on other nix flavors. It also has a proof of concept port for Microsoft Window..

## Installation
Automated builds of the image are available on Dockerhub and is the recommended method of installation.
```
docker pull meg4r0m/alpine-nginx:7.1
```

You can also pull the latest tag which is built from the repository HEAD : Nginx Mainline with Openssl.
```
docker pull meg4r0m/alpine-nginx:latest
```

If you are crazy, you can also pull nightly tag witch is build every day from nginx git source.
```
docker pull meg4r0m/alpine-nginx:7.2
```


## Basic usage
```
docker run -p 80:80 -p 443:443 --name nginx meg4r0m/alpine-nginx
```

### with docker-compose.yml v1

```yml
nginx:
  image: meg4r0m/alpine-nginx
  restart: always
  ports:
    - 80:80
    - 443:443
  #volumes: #Overwrite nginx configuration
  #  - /your/path/to/nginx.conf:/etc/nginx/conf/nginx.conf:ro
```

## Custom Nginx configuration

You can overwrite nginx configuration:

Create your own nginx.conf. Make sure your nginx.conf file has a volume to ```/etc/nginx/conf/nginx.conf```

```
docker run -p 80:80 -p 443:443 -v /your/path/to/nginx.conf:/etc/nginx/conf/nginx.conf:ro --name nginx meg4r0m/alpine-nginx
```


Make sure you set ```daemon off``` in your configuration otherwise the container will exit.