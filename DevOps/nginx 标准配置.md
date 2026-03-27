```apache_conf
user nginx;
worker_processes auto;
pid /var/run/nginx.pid;
worker_rlimit_nofile 51200;

events {
    use epoll;
    worker_connections 51200;
    multi_accept on;
}
 

http {
    include mime.types;
    default_type application/octet-stream;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32m;
    large_client_header_buffers 4 32m;
    client_max_body_size 1024000m;                     #最大请求文件大小
    client_body_buffer_size 1024m;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 86400s;
    server_tokens off;
    tcp_nodelay on;

    fastcgi_connect_timeout 300s;
    fastcgi_send_timeout 300s;
    fastcgi_read_timeout 300s;
    fastcgi_buffer_size 16m;
    fastcgi_buffers 4 16m;
    fastcgi_busy_buffers_size 16m;
    fastcgi_temp_file_write_size 16m;
    fastcgi_intercept_errors on;

    gzip on;
    gzip_buffers 16 8k;
    gzip_comp_level 6;
    gzip_http_version 1.1;
    gzip_min_length 256;
    gzip_proxied any;
    gzip_vary on;
    gzip_types
        text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
        text/javascript application/javascript application/x-javascript
        text/x-json application/json application/x-web-app-manifest+json
        text/css text/plain text/x-component
        font/opentype application/x-font-ttf application/vnd.ms-fontobject
        image/x-icon;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

    include vhost/*.conf;
  



    # simflex 访问
    server {
        listen 80;
        server_name simflex.funsine.com;

        location / {
            proxy_ignore_client_abort on;
            proxy_pass http://simflextraefik;
            proxy_set_header REMOTE_ADDR $remote_addr;
            proxy_set_header Host $http_host;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_set_header Upload-Offset $http_upload_offset;
            proxy_set_header Upload-Length $http_upload_length;
            proxy_set_header Upload-Metadata $http_upload_metadata;
            
            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 8640s0;
            
            client_body_timeout 86400s;
            chunked_transfer_encoding on;
        }
    }

    # hpc3 演示环境
    server {
        listen   80 ; 
        server_name  demo.funsine.com;                                                 #修改 - rancher80端口

        location / {
            proxy_ignore_client_abort on;
            proxy_pass http://apptraefik;
            proxy_set_header REMOTE_ADDR $remote_addr;
            proxy_set_header Host $http_host;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_set_header Upload-Offset $http_upload_offset;
            proxy_set_header Upload-Length $http_upload_length;
            proxy_set_header Upload-Metadata $http_upload_metadata;
            
            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 8640s0;
            
            client_body_timeout 86400s;
            chunked_transfer_encoding on;
        }

        location /hpc-user-image {
            proxy_pass http://apptraefik;
            proxy_set_header REMOTE_ADDR $remote_addr;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Proto https;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_buffering off;
            proxy_request_buffering off;
        }

        location /hpc-plugin-svc {
            proxy_pass http://apptraefik;
            proxy_set_header REMOTE_ADDR $remote_addr;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Proto https;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_buffering off;
            proxy_request_buffering off;
        }
    }

    # ai 平台访问域名
    server {
        listen   80 ;
        server_name  ai.funsine.com;                                                 #修改 - rancher80端口

        location / {
            proxy_ignore_client_abort on;
            proxy_pass http://aitraefik;
            proxy_set_header REMOTE_ADDR $remote_addr;
            proxy_set_header Host $http_host;
            #proxy_set_header X-Forwarded-Proto https;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_set_header Upload-Offset $http_upload_offset;
            proxy_set_header Upload-Length $http_upload_length;
            proxy_set_header Upload-Metadata $http_upload_metadata;

            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 8640s0;

            client_body_timeout 86400s;
            chunked_transfer_encoding on;
        }
    }

    # nexus 内部访问镜像仓库
    server {
        listen 80 default;
        listen 443 ssl;
        server_name idocker.io ai.idocker.io ;                                                          #内部访问本地私服使用的域名（无需修改）
        ssl_certificate /etc/nginx/ssl/idocker-io.crt;                                    #idocker证书（无需修改）
        ssl_certificate_key /etc/nginx/ssl/idocker-io.key;                                #idocker证书（无需修改）

        client_max_body_size 0;
        chunked_transfer_encoding on;
        # 设置默认使用推送代理
        set $upstream_arm "nexus_docker_hosted";
        set $upstream "nexus_docker_x86_hosted";

        # 当请求是GET，也就是拉取镜像的时候，这里改为拉取代理，如此便解决了拉取和推送的端口统一
        if ( $request_method ~* 'GET') {
            set $upstream_arm "nexus_docker_group";
            set $upstream "nexus_docker_x86_group";
        }
        # 只有本地仓库才支持搜索，所以将搜索请求转发到本地仓库，否则出现500报错
        if ($request_uri ~ '/search') {
            set $upstream_arm "nexus_docker_hosted";
            set $upstream "nexus_docker_x86_hosted";
        }


        index index.html index.htm index.php;

        location / {
                proxy_pass http://$upstream;
                #if ($remote_addr = 192.168.1.66) {                        #修改（根据实际的节点进行配置与增减）（除以下配置之外的所有主机的请求，皆被识别为arm机器）
                #    proxy_pass http://$upstream_arm;
                #}
                proxy_set_header Host $host;
                proxy_connect_timeout 60;
                proxy_send_timeout 60;
                proxy_read_timeout 86400;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_buffering off;
                proxy_request_buffering off;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto http;
        }
        # 将主机端口的 9090 端口暴露出来
    }

    # simflex 平台 上传流
    upstream simflextraefik {
        server 192.168.100.131:81 weight=100  max_fails=3 fail_timeout=5s ;      
        keepalive 16;
    }

    # hpc3 平台 上传流
    upstream apptraefik {
        server 192.168.100.131:81 weight=100  max_fails=3 fail_timeout=5s ;      
        keepalive 16;
    }

    # ai 平台 上传流
    upstream aitraefik {
        #least_conn;
        server 192.168.1.48:81 weight=5  max_fails=3 fail_timeout=5s ;
        server 192.168.1.49:81 weight=1  max_fails=3 fail_timeout=5s ;
        keepalive 16;
    }
  
    #nexus同域名根据条件不同端口
    upstream nexus_docker_hosted {
        server 192.168.100.131:8085;                                                           #修改
    }
    upstream nexus_docker_group {
        server 192.168.100.131:8084;                                                           #修改
    }

    #nexus同域名根据条件不同端口
    upstream nexus_docker_x86_hosted {
        server 192.168.100.131:8083;                                                           #修改
    }
    upstream nexus_docker_x86_group {
        server 192.168.100.131:8082;                                                           #修改
    }
}

```
