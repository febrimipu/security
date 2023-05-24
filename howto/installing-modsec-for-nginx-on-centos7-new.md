---
title: Installing ModSecurity with Nginx on CentOS7
subject: ModSecurity
---

# Installing ModSecurity with Nginx on CentOS7

## But first...
You need sudo access and the most current version of CentOS 7. See [Creating sudo users on CentOS](https://github.com/thermoio/docs/blob/master/getting-started/creating-sudo-users-on-centos) for more information.

## 1: Update
See [Updating CentOS](https://www.thermo.io/how-to/security/updating-centos).

## 2: Install dependencies
Install the following packages:
```shell
yum groupinstall -y "Development Tools"
yum install -y httpd httpd-devel pcre pcre-devel libxml2 libxml2-devel curl curl-devel openssl openssl-devel bison curl curl-devel doxygen flex gcc-c++ git GeoIP-devel libxml2 libxml2-devel lmdb lmdb-devel lua lua-devel pcre-devel ssdeep ssdeep-devel yajl yajl-devel zlib-devel
shutdown -r now
```

## 3: Compile ModSec

1. Download and install ModSecurity:
   ```shell
   cd /usr/local/src/
   git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
   cd ModSecurity/
   git submodule init
   git submodule update
   ./build.sh
   ./configure
   make
   make install
   ```
2. Clone ModSecurity-nginx repository:
**Attention:** This contains Nginx ModSecurity module source code.
   ```shell
   cd /usr/local/src
   git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
   ```

## 4: Compile Nginx
1. We need to download the source code for the version of Nginx you are running now:
   ```shell
   nginx -v
   ```
   
   nginx version: nginx/1.20.1
   In this case, we use Nginx 1.20.1, go to http://nginx.org/en/download.html
2. Download the source code for Nginx version you are using:
   ```shell
   cd /usr/local/src
   wget http://nginx.org/download/nginx-1.20.1.tar.gz
   tar xvf nginx-1.20.1.tar.gz
   cd nginx-1.20.1
   ```
3. Find out the configure command used to compile nginx.
   ```shell
   nginx -V
   ```
   nginx version: nginx/1.20.1
   built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
   built with OpenSSL 1.0.2k-fips  26 Jan 2017
   TLS SNI support enabled
   configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'

4. You can see configure arguments on the last line, we need to use these arguments when we compile Nginx from source code
   ```shell
   cd /usr/local/src/nginx-1.20.1
   ./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-dynamic-module=../ModSecurity-nginx
   ```

   In the above, we added –add-dynamic-module=../ModSecurity-nginx at end of the configure command to compile the Nginx module.

5. Build Nginx modules
    ```shell
	make modules
	```
	
6. Copy it to /etc/nginx/modules
    ```shell
	cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
	```
	
7. Copy ModSecurity configuration files
    ```shell
	cp /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsecurity.conf
    cp /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/unicode.mapping
	```

8. Enable ModSecurity
    ```shell
    sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/g' /etc/nginx/modsecurity.conf
	```	
	
	
## 5: Configure ModSec and Nginx
1. Configure Nginx:

   a. Issue:
      ```shell
      vi /etc/nginx/nginx.conf
      ```
   b. Find the following segment within the `top` segment:
      ```shell
      worker_processes  auto;
      ```
   Add the lines below so the final result should be:
      ```shell
      load_module modules/ngx_http_modsecurity_module.so;
      ```
   c. Edit your server config (virtual host entry), add :
   
      ```shell
      modsecurity on;
	  modsecurity_rules_file /etc/nginx/modsecurity.conf;
      ```
   
   d. Save and quit:
      ```shell
      :wq!
      ```

2. Install ModSecurity Rules:
**Attention:** You can download ModSecurity rules from https://coreruleset.org

At the time of writing this, the latest version is v3.3.2. So let’s download and install it.
   ```shell
   cd /usr/local/src
   wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.4.tar.gz
   tar xvf v3.3.4.tar.gz
   mv coreruleset-3.3.4 /etc/nginx
   cd /etc/nginx/coreruleset-3.3.4
   cp crs-setup.conf.example crs-setup.conf
   ```
3. To activate the rule, edit the file:
   ```shell
   vi /etc/nginx/modsecurity.conf
   ```
4. At end of the file, add:
   ```shell
   Include /etc/nginx/coreruleset-3.3.4/crs-setup.conf
   Include /etc/nginx/coreruleset-3.3.4/rules/*.conf
   SecRule ARGS:sec-test "@contains hacker" "id:1234,deny,status:403"       /usr/local/nginx/conf/modsecurity.conf
   ```
5. Restart Nginx
   ```shell
   systemctl restart nginx
   ```
6. To verify ModSecurity is working, access your website URL with
   ```shell
   curl -I http://YOUR-SERVER-IP-OR-DOMAIN/?sec-test=hacker
   or open your browser
   http://your-ip-server/?param="><script>alert(1);</script>
   
   ```
   You will see 403 Forbidden error.
    ``shell
   curl -I http://localhost/?param=%22%3E%3Cscript%3Ealert(1);%3C/script%3E
   HTTP/1.1 403 Forbidden
   Server: nginx/1.20.1
   Date: Mon, 12 Jul 2023 18:24:36 GMT
   Content-Type: text/html
   Content-Length: 153
   Connection: keep-alive
   
   ```