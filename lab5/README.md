# Лабораторная работа №5 - Мониторинг k8s с помощью prometheus и grafana.

## Цель работы

- настроить мониторинг кластера Kubernetes.

## Выполение работы

### Установка Helm.
Для удобства установки нам понадобится Helm.
Helm – это популярный диспетчер пакетов для k8s, который значительно упрощает процесс установки, управления и масштабирования приложений в кластере.
Для установки идём [сюда](https://helm.sh/docs/intro/install/) и смотрим вариант для нашей системы:

```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
Для проверки успешности выполним `Helm help`:

![image](img/Screenshot_1.png)

### Устанавливаем Prometheus

Для начала создадим namespace, в котором будет расположен мониторинг:
```
minikube kubectl -- create namespace monitoring
```

![image](img/Screenshot_2.png)

Теперь добавим репозиторий с prometheus: 
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

![image](img/Screenshot_3.png)

и обновим его:
```
helm repo update
```

![image](img/Screenshot_4.png)

После чего устанавливаем prometheus в недавно созданное пространство:
```
helm install prometheus prometheus-community/prometheus --namespace monitoring
```

![image](img/Screenshot_5.png)
![image](img/Screenshot_6.png)

Проверяем, всё ли хорошо:
```
minikube kubectl -- get pods -n monitoring
```

![image](img/Screenshot_7.png)


Теперь нам нужно, чтобы сервис мониторинга был доступен не только внутри кластера, но и снаружи.
Для этого делаем `expose` сервиса, и меняем его тип с ClusterIP на NodePort:
```
minikube kubectl -- expose service prometheus-server --namespace monitoring --type=NodePort --target-port=9090 --name=prometheus-server-ext
```
Проверим, что всё хорошо, выполнив команду `minikube kubectl -- get svc -n monitoring`.
Должны увидеть сервис с именем *prometheus-server-ext* и типом *NodePort*. 
Так же мы видим внешний порт, по которому он доступен (в данном случае - 30247):
![image](img/Screenshot_9.png)

Узнаем ip миникуба с помощью команды `minikube ip`:
 ![image](img/Screenshot_10.png)

Теперь у нас есть всё необходимое, чтобы открыть вебморду prometheus в браузере: 
![image](img/Screenshot_11.png)

Ура, prometheus успешно установлен!

### Устанавливаем Grafana

Добавим репозиторий с grafana: 
```
helm repo add grafana https://grafana.github.io/helm-charts
```

![image](img/Screenshot_12.png)

Устанавливаем grafana в пространство monitoring:
```
helm install grafana grafana/grafana --namespace monitoring
```

![image](img/Screenshot_13.png)

В логах видим команду для получения пароля, выполняем:

```
minikube kubectl -- get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

![image](img/Screenshot_14.png)

Делаем графану доступной извне по аналогии с prometheus:
```
minikube kubectl -- expose service grafana --namespace monitoring --type=NodePort --target-port=3000 --name=grafana-ext
```
![image](img/Screenshot_15.png)

И проверяем всё той же командой: `minikube kubectl -- get svc -n monitoring`.
![image](img/Screenshot_16.png)

Идём в браузер и пробуем попасть в графану:
![image](img/Screenshot_17.png)

Всё хорошо, графана работает.
В качестве логина используем `admin`, а пароль мы уже узнали чуть выше.

### Добавление источника данных в Grafana

Для этого кликает по плитке `Data source`.
Там выбираем prometheus:
![image](img/Screenshot_18.png)

Указываем адрес сервиса prometheus:
![image](img/Screenshot_19.png)

И нажимаем `Save & test`. Это сохранит настройки и проверит работу источника данных.
![image](img/Screenshot_20.png)

### Добавление дашборда

В меню выбираем `Dashboards` и кликаем по `Import`.
Используем уже готовый дашборд с номером 15757. Больше готовых дашбордов можно взять на сайте [графаны](https://grafana.com/grafana/dashboards/).
![image](img/Screenshot_23.png)

Установим prometheus в качестве источника данных для дашборда и нажмём `Import`.
![image](img/Screenshot_24.png)

В резьтате видим наш дашборд с различными метриками кластера:
![image](img/Screenshot_25.png)


## В заключение

Мы научились собирать метрики k8s с помощью prometheus и отображать в grafana.
