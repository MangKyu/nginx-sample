# 1. Nginx 설정 공유
- mime.types: Nginx 서버에서 처리할 MIME 타입 모음
- upstream.conf: upstream(서버 그룹)을 모아둔 설정 파일
- header.conf: 요청을 다른 서버로 프록시할 때, 전달해주는 헤더 설정 파일 
- ssl.conf: SSL과 관련된 설정 파일, SSL 인증서 파일을 경로로 참조함
- ssl-server.conf: SSL 요청(443 포트)로 들어오는 요청과 관련된 서버 블럭 설정 파일
- http-server.conf: Http 요청(80 포트)로 들어오는 요청과 관련된 서버 블럭 설정 파일
- nginx.conf: Nginx 서버에서 참고하는 설정 파일로 다른 설정 파일들을 include 함


<br>
<br>
<br>

# 2. Nginx 설정 시의 주의사항
## server_name 주의사항

nginx는 요청이 들어오면 먼저 어느 서버 블록이 요청을 처리할 지 판단하게 되는데, server_name에 설정된 값이 이를 결정한다. 예를 들어, 80번 포트에 대한 3개의 서버 블록이 존재하는 상황이라고 하자.

```nginx
server {
    listen      80;
    server_name mangkyu.org www.mangkyu.org;
    ...
}

server {
    listen      80;
    server_name mangkyu.net www.mangkyu.net;
    ...
}

server {
    listen      80;
    server_name mangkyu.com www.mangkyu.com default_server;
    ...
}
```



nginx는 요청 헤더의 “Host” 값을 바탕으로 어느 서버에 요청을 라우팅할지 결정한다. 그리고 만약 매칭되는 server_name이 없거나 Host 헤더 값이 없다면, 해당 포트의 default 서버로 요청을 보내게 되고, 만약 default로 지정된 server_name이 없다면 위에서부터 가장 먼저 매칭되는 서버 블록이 처리하게 된다. 
Host 헤더 필드가 없는 경우에 요청을 drop 시키려면 다음과 같이 설정할 수도 있다.

```nginx
server {
    listen      80;
    server_name "";
    return      444;
}
```



하나의 서버에 여러 개의 DNS가 매핑되는 상황에서 nginx를 설정하려다 보면 원하지 않는 server 블록이 요청을 처리하게 되는 문제가 발생할 수 있다. 이러한 경우에는 server_name이 올바르게 매핑되는지 확인해보면 된다.

<br>
<br>

## proxy_set_header 주의사항
proxy_pass 시에 추가적인 헤더를 전달하기 위해서는 proxy_set_header를 사용해야 하며, http, server, location 블록에서 사용할 수 있다. 만약 별도의 설정이 없다면 아래의 2가지 헤더를 기본적으로 갖게 된다.

```nginx
proxy_set_header Host       $proxy_host;
proxy_set_header Connection close;
```

<br>
<br>

proxy_set_header는 기본적으로 상위 수준의 설정을 상속받는다. 즉, 상위 블록에 해당 설정이 있다면 하위 블록에서도 사용하게 되는 것이다. 예를 들어 proxy_set_header가 http 또는 server 블록에 존재한다면, location 블록에서도 이를 사용할 수 있다.

하지만 하위 수준의 블록에서 proxy_set_header를 해준다면, 상위 수준의 블록에 존재하는 proxy_set_header는 날라가고 해당 블록의 proxy_set_header만 남게 된다.

예를 들어 다음과 같은 server 블록이 있다고 하자. 서로 다른 2개의 key(Hello, MangKyu)를 갖는 proxy_set_heade가 각각 server 블록과 locatino 블록에 존재한다. 이 경우에 전달되는 헤더는 어떤 것이 있을까?

```nginx
 server {
    listen       80;
    server_name  localhost;
    
    # 상위 블록에 존재하는 설정은 key가 다름에도 불구하고 무시됨
    proxy_set_header Hello "Hello";
    
    location / {
        proxy_set_header MangKyu "MangKyu";
        proxy_pass http://developers;
    }        
    
    ...
}
```


<br>
<br>

여기서 proxy_pass로 전달되는 헤더는 MangKyu 뿐이다. 왜냐하면 더 좁은 블록인 location에 proxy_set_header가 존재하므로, 상위 블록들의 proxy_set_header 설정들은 무시되기 때문이다. 상위 블록에 존재하는 proxy_set_header 설정은 key와 하위 블록에 존재하는 key가 다름에도 불구하고 무시된다. 그러므로 이러한 부분을 주의해서 사용해야 한다.
