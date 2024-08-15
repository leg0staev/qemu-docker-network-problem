# При запуске Docker пропадает сеть на гостевых машинах QEMU (сеть настроена через мост, дистрибутив - Manjaro Linux)

Проблема в том, что docker изолирует свои сети и меняет политики цепочек в iptables с ACCEPT на DROP. Чтобы сеть виртуальных машинах снова заработала, нужно добавить правило в цепочку FORWARD для Вашего моста:

``sudo iptables -A FORWARD -p all -i br0 -j ACCEPT``

где ``br0`` - имя Вашего моста

При таком подходе после перезагрузки хоста правило пропадет. Но есть выход. Подготовим окружение перед запуском docker, изменив docker.service:

> docker.service — это файл, который используется системной службой для управления Docker Daemon на операционных системах, основанных на systemd. В этом файле содержатся параметры и конфигурации для запуска Docker Daemon.

``EDITOR=nano sudo -E systemctl edit docker``

далее нужно вставить эти строчки:

``[Service]``  
``ExecStartPre=/usr/bin/iptables -A FORWARD -p all -i br0 -j ACCEPT``  
``ExecStop=/usr/bin/iptables -D FORWARD -p all -i br0 -j ACCEPT``

где ``br0`` - имя Вашего моста

- ``ExecStartPre`` добавляет правило в iptables перед запуском приложения.
- ``ExecStop`` удаляет правило из iptables при остановке приложения.

после манипуляций выполните следующие команды для перезапуска службы Docker:

``sudo systemctl daemon-reload``  
``sudo systemctl restart docker``

Заключение.  
В теории можно настроить свои правила iptables и сохранить их перед установкой и запуском докер. Тогда docker будет добавлять свои праила к Вашим и ничего не должно ломаться.
Подробнее про iptables можно посмотреть у Лавлинского в [видео](https://youtu.be/Q0EC8kJlB64?si=A43zjV8teMnOSz5D)
