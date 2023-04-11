# Nginx 설정 공유
- mime.types: Nginx 서버에서 처리할 MIME 타입 모음
- upstream.conf: upstream(서버 그룹)을 모아둔 설정 파일
- header.conf: 요청을 다른 서버로 프록시할 때, 전달해주는 헤더 설정 파일 
- ssl.conf: SSL과 관련된 설정 파일, SSL 인증서 파일을 경로로 참조함
- ssl-server.conf: SSL 요청(443 포트)로 들어오는 요청과 관련된 서버 블럭 설정 파일
- http-server.conf: Http 요청(80 포트)로 들어오는 요청과 관련된 서버 블럭 설정 파일
- nginx.conf: Nginx 서버에서 참고하는 설정 파일로 다른 설정 파일들을 include 함
