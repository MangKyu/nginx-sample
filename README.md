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

여기서 proxy_pass로 전달되는 헤더는 MangKyu 뿐이다. 왜냐하면 더 좁은 블록인 location에 proxy_set_header가 존재하므로, 상위 블록들의 설정들은 무시되기 때문이다. 상위 블록에 존재하는 설정은 key와 하위 블록에 존재하는 key가 다름에도 불구하고 무시된다. 그러므로 이러한 부분을 주의해서 사용해야 한다.
