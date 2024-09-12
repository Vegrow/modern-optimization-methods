# Лабораторная работа №1.

## Цели работы:
Настроить nginx по заданному тз:
- Должен работать по https c сертификатом
- Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
- Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
- Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.

### Пошагово для чайников:
1. Устанавливаем nginx
```bash
sudo apt install nginx
```

![screenshot](/img/Screenshot_1.png)

2. Нам нужен серфтикат безопасности. Посколько мы на локальной виртуалочке, то не будем заморачиваться и выпустим самоподписанный.
 Создаём папку для сертификата
```bash
mkdir /etc/nginx/ssl
```

далее создаём сертификат и ключ командой:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx-selfsigned.key -out /etc/nginx/ssl/nginx-selfsigned.crt
```

при выполнии команды потребуется ввести некоторые данные, там всё интуитивно понятно.
![screenshot](/img/Screenshot_2.png)

3. Открываем файл /etc/nginx/sites-available/default.  и добавляем следующие сточки:

```bash
listen 443 ssl;
ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
```

Командой `nginx -t` проверяем, что всё хорошо, после чего рестартуем nginx командой `service nginx restart`
![screenshot](/img/Screenshot_3.png)

4. Открываем браузер, поверяем работу сайта по https. Поскольку сертификат самоподписный, то браузер орёт на нас благим матом. Осознавая все последствия и соглашаясь на риски, видим желаемое:
![screenshot](/img/Screenshot_4.png)

5. Добавляем принудительный редирект с 80 порта на 443. Для этого добавляем в файл конфигурации проверку на схему:

```bash
if ($scheme = http) {
return 301 https://$host$request_uri;
}
```

`Return 301` говорит браузеру и поисковым системам, что это постоянное перенаправление
А `https://$host$request_uri` является которотким кодом для указания версии HTTPS того, что набрал пользователь.
Не забываем рестартовать nginx после каждого изменения конфигурции.

6. Перейдём к настройке виртуальных хостов. Предположим, мы хотим, чтобы у нас было два сайта c доменами hellow1.ru и hellow2.ru
Поскольку у нас всё на виртуалке и реально доменами мы не владеем, настроим редирект на localhost.
Для этого добавляем в /etc/host следующие строки

```bash
127.0.0.1 hellow1.ru
127.0.0.1 www.hellow1.ru
127.0.0.1 hellow2.ru
127.0.0.1 www.hellow1.ru
```

7. Создаём директории для сайтов:
```bash
sudo mkdir -p /var/www/hellow1.ru/html
sudo mkdir -p /var/www/hellow2.ru/html
```

8. Создаём индексные файлы. Индексный файл - первый файл, к которому обращается nginx после обработки запроса от пользователя

```bash
vim /var/www/hellow1.ru/html/index.html
```
В открывшийся текстовый файл поместим следующий код
```
<html>
    <head>
        <title>Hellow1!</title>
    </head>
    <body>
        <h1>It's alive! Hellow1 alive!!!</h1>
    </body>
</html>
```
Поскольку индексный файл для второго сайта будет почти таким же, скопируем его в папку второго сайта, чтобы не переписывать полностью:
```
cp /var/www/hellow1.ru/html/index.html /var/www/hellow2.ru/html/
```
Далее открываем его и вносим изменения на свой вкус. Должно получиться так:
![screenshot](/img/Screenshot_7.png)
![screenshot](/img/Screenshot_8.png)

9. Создание виртальных хостов.
По умолчанию в nginx создан один серверный блок, который можно использовать в качестве шаблона для создания собственных блоков. Скопируем его:
```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/hellow1.ru
```
Откроем командой `sudo vim /etc/nginx/sites-available/hellow.com`. Сносим все комментарии. Удаляем default_server в строках listen.
В качестве директории, где хранятся файлы сайт, укажем новый адрес, который мы создали: `root /var/www/hellow1.ru/html`
В строку `server_name` впишем наше доменное имя `hellow1.ru`
В итоге должен получиться следующий конфигурационный файл:
![screenshot](/img/Screenshot_9.png)

Теперь копируем конфигурационный файл для второго нашего сайта: `sudo cp /etc/nginx/sites-available/hellow1.ru /etc/nginx/sites-available/hellow2.ru`
И не забываем указать правильный адрес в `root` и доменное имя в `server_name`
![screenshot](/img/Screenshot_10.png)

10. Подключение виртуальных хостов
Виртуальные хосты созданы, теперь их нужно подключить. Создадим для этого символические ссылки для каждого из сайтов:
```
sudo ln -s /etc/nginx/sites-available/hellow1.ru /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/hellow2.ru /etc/nginx/sites-enabled/
```
И снова перезагружаем сервер.
После этого проверим, всё ли мы правильно сделали. Для этого открываем браузер и переходим по адресам http://hellow1.ru и http://hellow2.ru.
![screenshot](/img/Screenshot_14.png)
![screenshot](/img/Screenshot_15.png)

13. Добавим alias.
Скачаем катринку с кисой и положим её в папку `/var/www/hellow1.ru/html/media` под именем `kitty.png`
Добавим в конфигурационный файл `/etc/nginx/sites-available/hellow2.ru` следующие строки:
```
location /hellowkitty.png {
	alias /var/www/hellow1.ru/html/media/kitty.png
}
```
Сохраним, перезагрузим nginx.
Проверим работу алиаса:
![screenshot](/img/Screenshot_16.png)

## Итоги работы:
Познакомились с nginx, cократили количество нервных клеток.