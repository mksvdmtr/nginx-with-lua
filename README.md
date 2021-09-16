## Nginx installation with lua on Debian 10/11
```
apt update && apt install -y git nginx wget build-essential libreadline-dev libpcre-ocaml-dev libssl-dev zlib1g-dev 

for i in /etc/nginx/modules-enabled/*; do unlink $i; done > /dev/null 2>&1 || echo done

mkdir -p /var/cache/nginx/client_temp

cd /tmp
git clone https://luajit.org/git/luajit.git
cd luajit
make install PREFIX=/usr/local/LuaJIT

cd ..
wget https://github.com/openresty/lua-resty-core/archive/refs/tags/v0.1.22.tar.gz
tar xvf v0.1.22.tar.gz
cd lua-resty-core-0.1.22
make install

cd ..
git clone https://github.com/openresty/lua-resty-lrucache.git
cd lua-resty-lrucache
make install

cd /opt
wget https://github.com/vision5/ngx_devel_kit/archive/refs/tags/v0.3.1.tar.gz
tar xvf v0.3.1.tar.gz

wget https://github.com/openresty/lua-nginx-module/archive/refs/tags/v0.10.20.tar.gz
tar xvf v0.10.20.tar.gz

export LUAJIT_LIB=/usr/local/LuaJIT/lib
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.1

cd /tmp
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar xvf nginx-1.20.1.tar.gz
cd nginx-1.20.1

./configure --prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--modules-path=/usr/lib/nginx/modules \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nginx \
--group=nginx \
--with-compat \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-cc-opt='-g -O2 -fdebug-prefix-map=/data/builder/debuild/nginx-1.19.10/debian/debuild-base/nginx-1.19.10=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie' \
--with-ld-opt=-Wl,-rpath,/usr/local/LuaJIT/lib \
--add-module=/opt/ngx_devel_kit-0.3.1 \
--add-module=/opt/lua-nginx-module-0.10.20

make -j$(nproc)
make install

Add to /etc/nginx/nginx.conf in http section:
lua_package_path "/usr/local/lib/lua/?.lua;;";

systemctl restart nginx
```