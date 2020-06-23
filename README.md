# Configuring Openresty on MAC
The main purpose of the script is to capture POST requests sent to an server (we use openresty here) and analyse them to check for duplicates or invalid elements present in the request.


## Installing required components (for MAC).
1. Install postman desktop client to handle GET and POST requests to server.
2. Install Lua and Luajit using ```brew```
```
	$ brew install Lua
	$ brew install luajit
```
3. Install openresty using ```brew```
```
	$ brew install openresty
```
4. Install PCRE, curl, OpenSSL
```
	$ brew install pcre openssl curl
```
5. Configure nginx 
```lua
 wget 'https://nginx.org/download/nginx-1.19.0.tar.gz'
 tar -xzvf nginx-1.19.0.tar.gz
 cd nginx-1.19.0/ # You may want to replace the version number with the latest one


 # tell nginx's build system where to find LuaJIT 2.0:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.0

 # tell nginx's build system where to find LuaJIT 2.1:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.1

 # Here we assume Nginx is to be installed under /opt/nginx/.
 ./configure --prefix=/opt/nginx \
         --with-ld-opt="-Wl,-rpath,/path/to/luajit/lib" \
         --add-module=/path/to/ngx_devel_kit \
         --add-module=/path/to/lua-nginx-module

 # Note that you may also want to add `./configure` options which are used in your
 # current nginx build.
 # You can get usually those options using command nginx -V

 # you can change the parallism number 2 below to fit the number of spare CPU cores in your
 # machine.
 make -j2
 make install
```
These installations were enough for me to launch openresty on my Mac. If case of any issues, refer: https://openresty.org/en/installation.html ; https://github.com/openresty/lua-nginx-module

6. Install Postman desktop client. This is essential to handle requests to and from server.

## Configuring for launch
In this task, a lua script is run directly in openresty via its config file. In order to do this it is essential to identify the location of the `nginx.conf` file. It will probably be in this location : `/usr/local/etc/openresty`

Open `nginx.conf` in a text editor. The basic layout of the file would be of this format:
```

#user  nobody;
worker_processes  1;

error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    #include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /lua {
            default_type 'text/plain'; # To produce the response in a text format
            root   html;
            index  index.html index.htm;
        }
		# To allow POST on static pages
        error_page  405     =200 $uri;
    }
}

```
create a folder `logs` under `/path/to/openresty/nginx` (default location: `/usr/local/openresty/nginx`). This path stores the error logs as included in the conf file above. 

## Launching Openresty
Perform this command on Terminal or any other unix console
```
sudo /path/to/openresty/bin/openresty 
```
To stop the server 
```
sudo /path/to/openresty/bin/openresty -s quit
```

The `nginx.conf` file gets executed when openresty server is launched, so the lua script under it gets executed. This process can be verified on postman.
 
## Postman
Send the JSON payload in the text file as a POST request using postman. If everything works perfectly, the output of the lua script embedded in the `nginx.conf` file should return as a response.

## Lua
In this task, `content_by_lua_block{}` is used as the directive to write the lua script. For details, refer : https://openresty-reference.readthedocs.io/en/latest/Directives/#content_by_lua_file ; https://openresty-reference.readthedocs.io/en/latest/Lua_Nginx_API/
 
 
 
 
 
 
