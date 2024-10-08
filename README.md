# Домашнее задание к занятию «Запуск приложений в K8S» - `Мурчин Артем`

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

### Решение 1.

Конфиг Deployment - https://github.com/artmur1/22-03-K8S/blob/main/files/deployment.yaml. Контейнеры nginx и multitool занимают одинаковые порты 80, 8080, 443. Понимаю, что для одного из контейнеров нужно прописать в Deployment.yaml переадресацию порта 8080 к примеру на порт 9080. После долгого поиска информации, нашел как переадресовать порт в Deployment.yaml у wbitt/network-multitool:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-01.png)

Созданный под:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-02.png)

Оба контейнера работают:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-03.png)

Результат после увеличения количества реплик до 2-х:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-04.png)

Написал сервис - https://github.com/artmur1/22-03-K8S/blob/main/files/svc-nginx-multitool.yaml:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-05.png)

Запустил сервис:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-06.png)

Под wbitt/network-multitool - https://github.com/artmur1/22-03-K8S/blob/main/files/pod-multitool.yaml:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-07.png)

Под запущен:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-08.png)

Ответ с 80 порта от nginx:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-09.png)

Ответ с 8080 порта от wbitt/network-multitool. Видно, что при повторых запросах ответ приходит с разных реплик контейнера. Это пример работы балансировщика нагрузки.

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-01-10.png)


------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

### Решение 2.

После чтения документации по Init-контейнерам, нашел как реализовать задачу - https://github.com/artmur1/22-03-K8S/blob/main/files/pod-nginx.yaml:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-01.png)

Под запущен, но Status Init:0/1:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-02.png)

В описании пода видно, что статус у инит контейнера False:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-03.png)

Написал сервис - https://github.com/artmur1/22-03-K8S/blob/main/files/myservice.yaml:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-04.png)

Запустил сервис, под поднялся:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-05.png)

статус у инит контейнера True:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-06.png)

В Event видно, что nginx стартовал успешно:

![](https://github.com/artmur1/22-03-K8S/blob/main/img/22-03-02-07.png)

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
