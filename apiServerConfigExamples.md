# Server Setup

## Case when we have one API per server with multiple servers

nginx -c /home/vs/nginx_setup//nginx.conf -p /home/vs/nginx_setup// -s [reload|stop|quit]

  Nginx host: http://dev001:8099

    Python server 1 - http://dev002:8090/
    Python server 2 - http://dev001:8090/

    /svr1 - host dev002 - API 1 
    /svr2 - host dev001 - API 2 
    /wrong - 404 error 
    /wrong_new - redirected to API 1 

  Nginx Configuration: 
    worker_processes  5;  ## Default: 1
    error_log  /home/vs/nginx_setup//logs/error.log;
    pid        /home/vs/nginx_setup//logs/nginx.pid;
    worker_rlimit_nofile 8192;

    events {
      worker_connections  4096;  ## Default: 1024
    }

    http {
        server {
            gzip on; gunzip on;
            listen 8099;
            server_name dev002 dev001;
            location /svr1 { proxy_pass http://dev002:8090/; }
            location /svr2 { proxy_pass http://dev001:8090/; }
            location /wrong { return 404; }
            location /wrong_new { return 301 http://dev001:8090/; }
        }
    }


## Case when we have multiple API servers: 

/api on API server group - big_server_com 
a) dev002:8090
b) dev001:8090 

Nginx config 
    worker_processes  5;  ## Default: 1
    error_log  /home/vs/nginx_setup//logs/error.log;
    pid        /home/vs/nginx_setup//logs/nginx.pid;
    worker_rlimit_nofile 8192;

    events {
      worker_connections  4096;  ## Default: 1024
    }

    http {
        upstream big_server_com {
          server dev002:8090 weight=5;
          server dev001:8090;
        }
        server {
            gzip on; gunzip on; listen 8099;
            server_name dev002 dev001;
            location /svr_all   { proxy_pass http://big_server_com/; }
            location /wrong     { return 404; }
            location /wrong_new { return 301 http://dev001:8090/; }
        }
    }

