# Мониторинг ssh содинения на Linux серверах на предмет подбора пароля (Brute force)
 
Данный репозиторий содержит роль которая устанавливает на удаленный сервер prometheus-node-exporter и скрипт ssh_faild_connect.sh. Также эта роль добавляет правила запуска скрипта ssh_faild_connect.sh в crontab
1. Создается пользователь prometheus и устанавливается node-exporter из tar.gz архива. ( tag: install_node_exporter ) использует переменные 
* node_exporter_version: 1.1.2 - Описывает номер версии prometheus-node-exporter 
* node_exporter_bin_version: "node_exporter-{{ node_exporter_version }}.linux-{{ '386' if ansible_architecture == 'i386' else 'amd64' }}" - формирует название tar.gz архива с учетом номера версии и архитектуры текущего сервера. ( node_exporter-1.1.2.linux-amd64 ) 
* node_exporter_bin_url: "http://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/{{ node_exporter_bin_version }}.tar.gz" - формирует ссылку для скачивания tar.gz архива из репозитория github.
2. устанавливаем и настраиваем запуск скрипта ssh_faild_connect.ssh ( tag: install_ssh_faild_connect ) 
* создается каталог /var/lib/node_exporter/textfile_collector/ для хранения *prom файлов с метриками 
* скрипт ssh_faild_connect.sh копируется в каталог /opt/ 
* создается правило для запуска ssh_faild_connect.ssh в crontab 
* первый запуск скрипта
   
Описание:
***
Роль была создана для установки prometheus-node-exporter на станции работающие на старых OS был выбран выбран метод установки из архива для того чтоб избежать произвольной установки (обновления) пакетов. node-exporter после установки будет находиться в каталоге /opt/$node_exporter_bin_version и настроен на обработку текстовых файлов из каталога /var/lib/node_exporter/textfile_collector/
***
Скрипт ssh_faild_connect.sh запускается согласно правилам crontаb. Просматривает 1000 последних строк из файла /var/log/auth.log. После чего ищет наличие сбойных соединений по ssh протоколу по ключевым словам ( "Failed password" "ssh" ) и считает полученные строки. Эти вычисления производятся трижды, для пользователя root, для всех пользователей описанных в OS, также считается общее количество всех сбойных соединений.
***
Пример ssh_faild_connect.prom
```
ssh_faild_connect{user="root"} 6
ssh_faild_connect{user="alluser"} 6
ssh_faild_connect{user="homeusers"} 0
```
Также в репозитории находиться  файл ssh_faild_connections.yml описывающий алерты для данных метирк.
* Перевый алерт настроен на появление скорости изменения метрики ssh_faild_connect{user="alluser"} 
* Второй алерт настроен на превышение определенного количества сбойных соединений для метрики ssh_faild_connect{user="alluser"}

Также при составлении файла inventory_host  нужно будет учесть что некоторые сервера могут работать с версией python ниже 2.6 
с такими серверами ansible  работать не сможет.
