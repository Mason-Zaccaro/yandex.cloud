Руководство по работе с Yandex Cloud CLI (yc) — расширенная версия

---

1. Установка и настройка CLI

Установка  
Скачайте и установите утилиту yc для вашей ОС:  
https://cloud.yandex.ru/docs/cli/operations/install-cli

Настройка  
1. Инициализируйте профиль:
   yc init
2. Перейдите по ссылке для аутентификации  
3. Введите токен (действует 1 год)  
4. Выберите облако, каталог и зону по умолчанию  

---

2. Основные команды и структура  

Структура команд  
Сервис → Ресурс → Действие → Параметры  

---

3. Работа с сетями  

Создание сети и подсетей  
Создайте облачную сеть:  
   yc vpc network create --name my-network  

Добавьте подсети:  
   yc vpc subnet create \
     --name my-subnet-a \
     --zone ru-central1-a \
     --range 192.168.1.0/24 \
     --network-name my-network  

Публичные IP-адреса  
Назначение динамического IP:  
   --network-interface subnet-name=my-subnet-a,nat-ip-version=ipv4  

Для статического IP: преобразуйте через консоль управления.  

Статическая маршрутизация  
Создайте таблицу маршрутизации:  
   yc vpc route-table create --name my-route-table  

Добавьте маршрут:  
   yc vpc route-table add-route \
     --name my-route-table \
     --destination 0.0.0.0/0 \
     --next-hop 192.168.1.5  

---

4. Управление виртуальными машинами (ВМ)  

Создание ВМ с NGINX  
   yc compute instance create \
     --name web-server \
     --zone ru-central1-a \
     --create-boot-disk image-family=ubuntu-2204-lts,size=30 \
     --memory 4 --cores 2 \
     --network-interface subnet-name=my-subnet-a,nat-ip-version=ipv4 \
     --metadata-from-file user-data=startup.sh  

Управление дисками и снимками  

Создание снимка:  
   yc compute disk snapshot create \
     --disk-id <disk-id> \
     --name my-snapshot  

Восстановление из снимка:  
   yc compute instance create \
     --name restored-vm \
     --zone ru-central1-a \
     --create-boot-disk snapshot-name=my-snapshot  

---

5. Группы безопасности  

Создание группы  
   yc vpc security-group create --name yc-security  

Добавление правил  
   yc vpc security-group add-rules yc-security \
     --rule "direction=ingress,port=80,protocol=tcp,cidr=0.0.0.0/0" \
     --rule "direction=egress,port=80,protocol=tcp,cidr=0.0.0.0/0"  

---

6. Балансировка нагрузки  

Создание целевой группы  
   yc load-balancer target-group create \
     --name demo-web \
     --target subnet-name=my-subnet-a,address=192.168.1.10 \
     --target subnet-name=my-subnet-b,address=192.168.2.20  

Настройка балансировщика  

Создайте балансировщик:  
   yc load-balancer network-load-balancer create \
     --name lb-demo-web \
     --listener name=demo-web-listener,port=80,target-port=80  

Привяжите целевую группу:  
   yc load-balancer network-load-balancer attach-target-group \
     --name lb-demo-web \
     --target-group-id <target-group-id>  

---

7. Проверка состояния ресурсов  

Мониторинг операций  
   yc operation get <operation-id>  

Проверка статуса балансировщика  
   yc load-balancer network-load-balancer get <balancer-id>  

---

8. Удаление ресурсов  

Удаление ВМ и дисков  
   yc compute instance delete web-server  
   yc compute disk delete <disk-id>  

Удаление балансировщика  
   yc load-balancer network-load-balancer delete lb-demo-web  
