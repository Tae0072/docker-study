# Ubuntu 24.04 이미지 사용
FROM ubuntu:24.04

# vim 패키지 설치
RUN apt update && apt install -y vim

# 기본 작업 경로를 /app으로 설정
WORKDIR /app

# 로컬의 이미지 파일을 컨테이너로 복사
COPY ./images/image.png ./image.png

# 컨테이너 시작 시 bash 실행
CMD ["/bin/bash"]