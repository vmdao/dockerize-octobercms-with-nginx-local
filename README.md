# Step
Dockerize Octobercms - PHP
https://octobercms.com
1. Change dir volume on docker-compose.yml
``` 
- ./src:/var/www/html => - ./src: dir_project 
```

2. Clone code project to dir src
```
git clone .... src/ 
```

3. Edit wp-config.php

4. Config nginx local with letsencrypt
- config src
```
server {
        listen   80;
        listen    [::]:80;

        server_name your_domain;

        return 301 https://your_domain$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        server_name your_domain;

        ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

        ssl_session_timeout 2h;
        ssl_session_cache shared:SSL:20m;

        ssl_prefer_server_ciphers on;
        ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
        add_header Strict-Transport-Security "max-age=31536000" always;


        error_log  /your_dir_project/data/logs/your_domain_error.log error;
        access_log /your_dir_project/data/logs/your_domain_acces.log;


	    root /your_dir_project/src;
    	index index.php index.html;

	    location ~* \.php$ {
            fastcgi_index   index.php;
            fastcgi_pass    127.0.0.1:port_docker;
            include         fastcgi_params;
            fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
            fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;

            # config timeout cho proxy
            fastcgi_connect_timeout 60;
            fastcgi_send_timeout 180;
            fastcgi_read_timeout 180;
            fastcgi_buffer_size 512k;
            fastcgi_buffers 512 16k;
            fastcgi_busy_buffers_size 512k;
            fastcgi_temp_file_write_size 512k;
            fastcgi_intercept_errors on;
        }

    	# static file
    	location ~* \.(3gp|gif|jpg|jpeg|png|ico|wmv|avi|asf|asx|mpg|mpeg|mp4|pls|mp3|mid|wav|swf|flv|exe|zip|tar|rar|gz|tgz|bz2|uha|7z|doc|docx|xls|xlsx|pdf|iso|eot|svg|ttf|woff)$ {
            gzip_static off;
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            access_log off;
            expires max;
            break;
        }

        location ~* \.(css|js)$ {
            #add_header Pragma public;
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            access_log off;
            expires 30d;
            break;
        }

        # block
        location = /robots.txt  { access_log off; log_not_found off; }
        location = /favicon.ico { access_log off; log_not_found off; expires 30d; }
        location ~ /\.          { access_log off; log_not_found off; deny all; }
        location ~ ~$           { access_log off; log_not_found off; deny all; }
        location ~ /\.git { access_log off; log_not_found off; deny all; }
        location = /nginx.conf { access_log off; log_not_found off; deny all; }	
}

```

5. Create soft link with site.conf
```
sudo ln -s  config/nginx/your_domain.conf /etc/nginx/conf.d/your_domain.conf   
```


