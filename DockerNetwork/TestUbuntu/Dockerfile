FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*  # 캐시된 패키지 파일 제거

# 컨테이너가 계속 실행되도록 설정
CMD ["tail", "-f", "/dev/null"]
