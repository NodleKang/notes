# Pod와 Deployment 수정

## Pod Edit

기존 파드를 편집할 때는 아래 사항 외에는 수정할 수 없습니다.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

실행 중인 Pod의 환경 변수, 서비스 계정, 리소스 제한(이후에 논의할 내용)과 같은 사양을 편집할 수는 없습니다. 

하지만 만약 정말로 하고 싶다면 두 가지 옵션이 있습니다:

### 방법1

`kubectl edit pod <파드이름>` 명령을 실행하면 vi 에디터에서 파드 사양이 열립니다.

에디터 하단에 임시로 사용되는 임시파일 이름이 보입니다.

필요한 속성을 편집하고 저장하면 거부되겠지만 vi 에디터에서 확인한 임시파일에는 저장됩니다. (메모리에는 반영이 안 되지만, 파일에는 저장되는 겁니다.)

다음 명령으로 파드를 삭제합니다.
```
kubectl delete pod 파드
```

그 후에 vi 에디터에서 확인했던 임시파일을 사용해서 새 파드를 생성합니다.
```
kubectl create -f 임시파일
```

### 방법2

yaml 형식의 파드 정의를 파일로 추출합니다.

```
kubectl get pod <파드이름> -o yaml > my-new-pod.yaml
```

파일을 수정한 후에는 위에 방법 1과 같은 식으로 기존 파드를 삭제하고, 새 파드를 생성합니다.

## Deployment 편집

```
kubectl edit deployment my-deployment
```
