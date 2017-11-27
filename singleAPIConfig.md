# Server Setup

## Case when we have one API per server with multiple servers

nginx -c /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/nginx.conf -p /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/ -s [reload|stop|quit]

  Nginx host: http://ld-dbn-trdev004:8099

    Python server 1 - http://ld-dbn-trdev009:8090/
    Python server 2 - http://ld-dbn-trdev004:8090/

    /svr1 - host ld-dbn-trdev009 - API 1 
    /svr2 - host ld-dbn-trdev004 - API 2 
    /wrong - 404 error 
    /wrong_new - redirected to API 1 

  Nginx Configuration: 
    worker_processes  5;  ## Default: 1
    error_log  /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/logs/error.log;
    pid        /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/logs/nginx.pid;
    worker_rlimit_nofile 8192;

    events {
      worker_connections  4096;  ## Default: 1024
    }

    http {
        server {
            gzip on; gunzip on;
            listen 8099;
            server_name ld-dbn-trdev009 ld-dbn-trdev004;
            location /svr1 { proxy_pass http://ld-dbn-trdev009:8090/; }
            location /svr2 { proxy_pass http://ld-dbn-trdev004:8090/; }
            location /wrong { return 404; }
            location /wrong_new { return 301 http://ld-dbn-trdev004:8090/; }
        }
    }


## Case when we have multiple API servers: 

/api on API server group - big_server_com 
a) ld-dbn-trdev009:8090
b) ld-dbn-trdev004:8090 

Nginx config 
    worker_processes  5;  ## Default: 1
    error_log  /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/logs/error.log;
    pid        /devsandbox/margin/vsachdev/git/prac/nginxTestSetup/nginx/logs/nginx.pid;
    worker_rlimit_nofile 8192;

    events {
      worker_connections  4096;  ## Default: 1024
    }

    http {
        upstream big_server_com {
          server ld-dbn-trdev009:8090 weight=5;
          server ld-dbn-trdev004:8090;
        }
        server {
            gzip on; gunzip on; listen 8099;
            server_name ld-dbn-trdev009 ld-dbn-trdev004;
            location /svr_all   { proxy_pass http://big_server_com/; }
            location /wrong     { return 404; }
            location /wrong_new { return 301 http://ld-dbn-trdev004:8090/; }
        }
    }

