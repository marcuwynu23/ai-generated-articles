# Rebuild nginx with modules addons

### Install dependencies

```sh
sudo apt-get install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
```

### Download the Nginx source code:

```sh
wget http://nginx.org/download/nginx-1.26.3.tar.gz
tar -xvzf nginx-1.26.3.tar.gz
cd nginx-1.26.3
```

### Download the headers_more module

```sh
git clone https://github.com/openresty/headers-more-nginx-module.git
```

### Configure and compile Nginx with the headers_more module

```sh
./configure \
--prefix=/etc/nginx \
--conf-path=/etc/nginx/nginx.conf \
--pid-path=/etc/nginx/nginx.pid \
--error-log-path=/var/log/nginx/error.log  \
--http-log-path=/var/log/nginx/access.log  \
--http-client-body-temp-path=/tmp/nginx/client_body_temp \
--http-proxy-temp-path=/tmp/nginx/proxy_temp  \
--http-fastcgi-temp-path=/tmp/nginx/fastcgi_temp  \
--http-uwsgi-temp-path=/tmp/nginx/uwsgi_temp \
--http-scgi-temp-path=/tmp/nginx/scgi_temp \
--with-http_ssl_module \
--with-http_v2_module \
--with-stream  \
--add-module=/root/nginx-1.26.3/headers-more-nginx-module
```

### Rebuild and install Nginx

```sh
make
sudo make install
```

### additional configurations and cleaning

```sh
sudo mv /etc/nginx/sbin/nginx /usr/sbin/nginx
sudo rm -rf /etc/nginx/sbin /etc/nginx/html
mkdir /tmp/nginx
```

### Test the Configuration

```sh
sudo nginx -t
```

If the test passes, you should see a message like:

```sh
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Then, reload Nginx:

```sh
sudo systemctl reload nginx
```
