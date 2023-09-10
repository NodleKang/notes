# crictl

crictl은 OCI(Open Container Initiative) 컨테이너 런타임을 위한 CLI(Command Line Interface) 도구입니다.

```
# 컨테이너 생성
crictl run [옵션] 이미지이름 [명령어]
```

```
# 컨테이너 목록 보기
crictl ps -a
```

```
# 컨테이너 상세 정보 보기
crictl inspect [컨테이너ID]
```

```
# 컨테이너 로그보기
crictl logs [컨테이너ID]
```

```
# 컨테이너 중지하기
crictl stop [컨테이너ID]
```

```
# 컨테이너 삭제하기
crictl rm [컨테이너ID]
```

```
# 이미지 목록보기
crictl images
```

```
# 이미지 다운로드
crictl pull [이미지이름]
```

```
# 이미지 삭제학
crictl rmi [이미지이름]
```
