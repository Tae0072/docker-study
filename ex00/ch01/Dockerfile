# 베이스 이미지 지정. 태그 없이 ubuntu만 쓰면 latest 태그가 적용된다.
FROM ubuntu

# apt-get update: 패키지 목록 최신화
# apt-get install -y cron: cron 설치 (-y는 자동 yes)
# &&와 \로 한 RUN에 묶는 이유: RUN 하나 = Docker 레이어 1개. 나눠 쓰면 캐시가 남아 이미지가 커진다.
RUN apt update && \
    apt install -y nano && \
    apt install -y cron

# 호스트의 scripts/ 폴더를 컨테이너 내부 /scripts/로 복사.
COPY scripts/ /scripts/
# 복사된 .sh 파일에 실행 권한 부여. 없으면 cron 실행 시 Permission denied 오류 발생.
RUN chmod +x /scripts/*.sh

# 컨테이너 시작 시 cron을 포그라운드(-f)로 실행.
# -f가 중요한 이유: Docker는 PID 1 프로세스가 종료되면 컨테이너도 종료된다.
# -f 없이 백그라운드로 실행하면 PID 1이 즉시 종료되어 컨테이너가 죽는다.
CMD ["cron", "-f"]
