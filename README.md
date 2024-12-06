# Домашнее задание к занятию «Обновление приложений»

### Цель задания

Выбрать и настроить стратегию обновления приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).
2. [Статья про стратегии обновлений](https://habr.com/ru/companies/flant/articles/471620/).

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.

## Решение
Вариант 1. Если приложение уже протестировано ранее, то лучшим вариантом наверное будет использовать стратегию обновления Rolling Update, указанием параметров maxSurge maxUnavailable для избежания ситуации с нехваткой ресурсов. Проводить обновление следует естественно в менее загруженный момент времени сервиса. При данной стратегии(Rolling Update) k8s постепенно заменит все поды без ущерба производительности. И если что-то пойдет не так, можно будет быстро откатится к предыдущему состоянию.

Вариант 2. Можно использовать Canary Strategy. Также указав параметры maxSurge maxUnavailable чтобы избежать нехватки ресурсов. Это позволит нам протестировать новую версию программы на реальной пользовательской базе(группа может выделяться по определенному признаку) без обязательства полного развертывания. После тестирования и собирания метрик пользователей можно постепенно переводить поды к новой версии приложения.


### Задание 2. Обновить приложение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.

[deployment-nginx1.19.yaml](https://github.com/sash3939/Kubernetes14-Update_Apps/blob/main/deployment-nginx1.19.yaml)

<img width="520" alt="deployment nginx 1 19" src="https://github.com/user-attachments/assets/dbd118d8-9064-4df4-93b4-cd2d2bc01dfd">

```bash
root@kuber:~/Kubernetes14-Update_Apps# kubectl get pod nginx-multitool-6bbd4c4d6f-9sk99 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 8f762e687877f21db380c20fd1d9c96fbe33bd6c2bb7b53b6792c334c383ed6a
    cni.projectcalico.org/podIP: 10.1.106.159/32
    cni.projectcalico.org/podIPs: 10.1.106.159/32
  creationTimestamp: "2024-12-06T19:33:58Z"
  generateName: nginx-multitool-6bbd4c4d6f-
  labels:
    app: nginx-multitool
    pod-template-hash: 6bbd4c4d6f
  name: nginx-multitool-6bbd4c4d6f-9sk99
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-multitool-6bbd4c4d6f
    uid: b2bb87e9-36bf-409a-af2d-4da8ae4b3ef0
  resourceVersion: "488874"
  uid: 037143fd-b975-40de-b0ed-388fd05bf288
spec:
  containers:
  - image: nginx:1.19
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-gmzlw
      readOnly: true
  - env:
    - name: HTTP_PORT
      value: "8080"
    image: wbitt/network-multitool
    imagePullPolicy: Always
    name: multitool
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-gmzlw
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kuber
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-gmzlw
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-12-06T19:34:18Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-12-06T19:33:58Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-12-06T19:34:18Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-12-06T19:34:18Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-12-06T19:33:58Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://5bcf56a5156d66cdcf7de24f29d807eb89cf1c25e4fdb0d139a77078ee8f326a
    image: docker.io/wbitt/network-multitool:latest
    imageID: docker.io/wbitt/network-multitool@sha256:d1137e87af76ee15cd0b3d4c7e2fcd111ffbd510ccd0af076fc98dddfc50a735
    lastState: {}
    name: multitool
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-12-06T19:34:17Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-gmzlw
      readOnly: true
      recursiveReadOnly: Disabled
  - containerID: containerd://38bbbd8070d091934dc0c69a2d5440b2269297a0f129e45fe53f6e5574985cd4
    image: docker.io/library/nginx:1.19
    imageID: docker.io/library/nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-12-06T19:34:15Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-gmzlw
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 192.168.43.233
  hostIPs:
  - ip: 192.168.43.233
  phase: Running
  podIP: 10.1.106.159
  podIPs:
  - ip: 10.1.106.159
  qosClass: BestEffort
  startTime: "2024-12-06T19:33:58Z"
```

2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

## Дополнительные задания — со звёздочкой*

Задания дополнительные, необязательные к выполнению, они не повлияют на получение зачёта по домашнему заданию. **Но мы настоятельно рекомендуем вам выполнять все задания со звёздочкой.** Это поможет лучше разобраться в материале.   

### Задание 3*. Создать Canary deployment

1. Создать два deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать две версии приложения — веб-страницы.
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
