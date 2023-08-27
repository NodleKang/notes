# docker 명령어

```
# 컨테이너 이미지를 받아서 즉시 실행
docker run ubuntu

# 컨테이너 이미지를 실행하고 커맨드 실행하기
docker run ubuntu [명령어]
docker run ubuntu sleep 5

# 도커 프로세스 확인
docker ps -a
```

```
# dockerfile
FROM Ubuntu
CMD ["명령어",  "파라미터"] 혹은 CMD 명령어 파라미터
```

```
# dockerfile
FROM Ubuntu
CMD sleep 5
```

```
# 도커 이미지 빌드: ubuntu-sleep 태그 지정, 현재 디렉토리에 dockerfile 있음
docker build -t ubuntu-sleeper .
docker run ubuntu-sleep
```

```
# dockerfile
FROM Ubuntu
ENTRYPOINT ["명령어"]
CMD ["파라미터"]
```

```
docker run ubuntu-sleeper 10
```

```
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
