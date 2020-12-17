# Материалы для вебинара Kubernetes - Быстрый старт (GeekBrains)

## Установка KiND

KiND (Kubernetes iN Docker) - это кластер Kubernetes, развернутый в Docker, поэтому он должен быть передварительно установлен:

* [Docker Desktop](https://www.docker.com/products/docker-desktop) (Windows, MacOS)
* [Docker Engine](https://docs.docker.com/engine/install/) (Linux)
* [Go](https://golang.org/doc/install)

Утанавка KiND для разных ОС

https://kind.sigs.k8s.io/docs/user/quick-start/#installation

> Если запустить KiND на вашем компьютере не удается, можно использовать сервис Play with Kubernetes
>
> https://labs.play-with-k8s.com/


## Запускаем кластер Kubernetes

> Docker должен быть запущен

> Если у вас уже есть файл kubeconfig сохраните его, иначе он будет перезаписан

Запускаем кластер с одним узлом
(1 Master + Worker)
```
kind create cluster

kubectl cluster-info --context kind-kind
```

Запускаем кластер с несколькими узлами
(1 Master, 2 Workers)
```
kind create cluster --config kind-config.yaml  --name cluster

kubectl cluster-info --context kind-cluster
```

Запускаем отказоустойчивый кластер
(3 Masters, 3 Workers)
```
kind create cluster --config kind-ha-config.yaml  --name cluster-ha

kubectl cluster-info --context kind-clister-ha
```

Показать все кластера Kubernetes, созданные KiND
```
kind get clusters
```

Показать созданные контейнеры Docker
```
docker ps
```

Удалить все кластера, созданные KiND
```
kind delete clusters --all
```


## Исследуем кластер Kubernetes

Показать все ноды кластера
```
kubectl get node
```

Вывести подробную информацию о нодах кластера
```
kubectl get node -o wide
```

Показать все pod в namespace kube-system
```
kubectl get pod -n kube-system
```

Вывести подробную информацию о pod в namespace kube-system
```
kubectl get pod -n kube-system -o wide
```


## Запускаем первое приложение

Создаем Namespace
```
kubectl create ns hpa-test
```

Устанавливаем тестовое приложение
```
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml -n hpa-test
```

Проверяем работу приложения
```
kubectl port-forward service/php-apache 8080:80

curl http://localhost:8080
```

Создаем Horizontal Pod Autoscaler 
```
kubectl autoscale deployment php-apache --cpu-percent=30 --min=1 --max=5
```

Устанавливаем Merics Server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Устанавливаем Merics Server (для KiND)

> Добавлен аргумент
>
>   --kubelet-insecure-tls

```
kubectl apply -f metrics-server.yaml
```

Проверяем, что Metrics Server запущен и собирает метрики
```
kubectl get pod -n kube-system -w |grep metrics

kubectl top pod

kubectl top node
```

Показать объекты hpa
```
kubectl get hpa -n hpa-test
```

Генерируем нагрузку на приложение
```
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

Проверяем состояние hpa и pods
```
kubectl get hpa -n hpa-test

kubectl get pod -w
```

## Проверяем надежность кластера Kubernetes

Удалить все pod в кластере
```
kubectl delete pod -A --all
```

## Chaos Engineering

Тестируем стабильность кластера Kubernetes с помощью приложения KubeDOOM, убивая монстра удаляем случайный Pod :)

Устанавливаем VNC Viewer

https://www.realvnc.com/en/connect/download/viewer/macos/

Подключаемся к KubeDoom через VNC **localhost:7000**

Включаем port-forward
```
kubectl port-forward deployment/kubedoom 7000:5900 -n kubedoom
```

>Пароль: **idbehold**

>Получить все оружие: **idkfa**

>Убрать стенку справа: **idspispopd**
