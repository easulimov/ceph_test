### Установка ceph (выполнить на всех нодах)
```bash
sudo apt install ceph -y
```


### Настройка мониторов (MON)


#### Генерация UUID для кластера
```bash
uuidgen
```


02ecfc51-3db2-451f-8f1c-ef5f62c275fe


#### Создание конфигурационного файла
```bash
sudo vim /etc/ceph/ceph.conf 
```

```ini
[global]
fsid = 02ecfc51-3db2-451f-8f1c-ef5f62c275fe
public network = 192.168.122.0/24

osd pool default pg num = 128
osd pool default pgp num = 128
osd journal size = 1024
osd pool default size = 3

[mon]
mon initial members = node01
mon host = node01, node02, node03
mon addr = 192.168.122.198,192.168.122.77,192.168.122.103

[mon.node01]
host = node01
mon addr = 192.168.122.198
```

#### Подготовка первого монитора


#### Создание связок ключей

```bash
ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow +'
```

```bash
ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'
```

```bash
ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd' --cap mgr 'allow r'
```

```bash
ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
```

```bash
ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
```

```bash
chown ceph:ceph /tmp/ceph.mon.keyring
```


#### Создание карты мониторов

```bash
monmaptool --create --add node01 192.168.122.198 --fsid 02ecfc51-3db2-451f-8f1c-ef5f62c275fe /tmp/monmap
```

#### Создание рабочего каталога для монитора


```bash
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node01
```

#### Инициализация хранилища монитора, с ипользованием ранее созданной связки ключей и карты монитора


```bash
sudo -u ceph ceph-mon --mkfs -i node01 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```


#### Запуск сервиса

```bash
systemctl start ceph-mon@node01
```


```bash
systemctl status ceph-mon@node01
```

#### Подготовка второго и третьего монитора


#### Добавление в архив конфигурационного файла и ключа, которые должны совпадать на всех нодах


```bash
tar czvf /tmp/config.tar.gz /etc/ceph/
```

#### Копирование архива на другие ноды

```bash
scp /tmp/config.tar.gz node02:/tmp/
```

```bash
scp /tmp/config.tar.gz node03:/tmp/
```


#### Настройка второго монитора на node02
#### Распаковка конфига и ключа


```bash
tar xzvf /tmp/config.tar.gz
```

```bash
mkdir /etc/ceph
mv etc/ceph/ceph.conf /etc/ceph/
mv etc/ceph/ceph.client.admin.keyring /etc/ceph/
```


#### Проверка распаковки
```bash
ls -alF /etc/ceph/
```


#### Создание каталога для монитора
```bash
mkdir -p /var/lib/ceph/mon/ceph-node02
```

#### Смена владельца каталога
```bash
chown ceph:ceph /var/lib/ceph/mon/ceph-node02
```

#### Создание временной папки для ключей и карты монитора
```bash
mkdir /tmp/node02
```


#### Получение ключей монитора
```bash
ceph auth get mon. -o /tmp/node02/monkeyring
```

#### Получение карты монитора
```bash
ceph mon getmap -o /tmp/node02/monmap
```

#### Инициализация карты мониторов 
```bash
sudo -u ceph ceph-mon -i node02 --mkfs --monmap /tmp/node02/monmap --keyring /tmp/node02/monkeyring
```

#### Добавление монитора в карту
```bash
ceph mon add node02 192.168.122.77:6789
```

#### Редактирование конфига, добавление информации о 2м и 3м мониторе

```bash
vim /etc/ceph/ceph.conf
```

```ini
[global]
fsid = 02ecfc51-3db2-451f-8f1c-ef5f62c275fe
public network = 192.168.122.0/24

osd pool default pg num = 128
osd pool default pgp num = 128
osd journal size = 1024
osd pool default size = 3

[mon]
mon initial members = node01
mon host = node01, node02, node03
mon addr = 192.168.122.198,192.168.122.77,192.168.122.103

[mon.node01]
host = node01
mon addr = 192.168.122.198

[mon.node02]
host = node02
mon addr = 192.168.122.77

[mon.node03]
host = node03
mon addr = 192.168.122.103
```

#### Запуск сервиса 2го монитора

```bash
systemctl start ceph-mon@node02
```


```bash
systemctl status ceph-mon@node02
```

#### Проверка работы второго монитора

```bash
ceph -s
```

#### Переход на node01 и правка конфига
#### Редактирвоание конфига, добавление информации о 2м и 3м мониторе

```bash
vim /etc/ceph/ceph.conf
```

```ini
[global]
fsid = 02ecfc51-3db2-451f-8f1c-ef5f62c275fe
public network = 192.168.122.0/24

osd pool default pg num = 128
osd pool default pgp num = 128
osd journal size = 1024
osd pool default size = 3

[mon]
mon initial members = node01
mon host = node01, node02, node03
mon addr = 192.168.122.198,192.168.122.77,192.168.122.103

[mon.node01]
host = node01
mon addr = 192.168.122.198

[mon.node02]
host = node02
mon addr = 192.168.122.77

[mon.node03]
host = node03
mon addr = 192.168.122.103
```

#### Настройка третьего монитора (node03)
#### Распаковка конфига и ключа


```bash
tar xzvf /tmp/config.tar.gz
```

```bash
mkdir /etc/ceph
mv etc/ceph/ceph.conf /etc/ceph/
mv etc/ceph/ceph.client.admin.keyring /etc/ceph/
```


#### Проверка распаковки
```bash
ls -alF /etc/ceph/
```


#### Создание каталога для монитора
```bash
mkdir -p /var/lib/ceph/mon/ceph-node03
```

#### Смена владельца каталога
```bash
chown ceph:ceph /var/lib/ceph/mon/ceph-node03
```

#### Создание временной папки для ключей и карты монитора
```bash
mkdir /tmp/node03
```


#### Получение ключей монитора
```bash
ceph auth get mon. -o /tmp/node03/monkeyring
```

#### Получение карты монитора
```bash
ceph mon getmap -o /tmp/node03/monmap
```

#### Инициализация карты мониторов 
```bash
sudo -u ceph ceph-mon -i node03 --mkfs --monmap /tmp/node03/monmap --keyring /tmp/node03/monkeyring
```

#### Добавление монитора в карту
```bash
ceph mon add node03 192.168.122.103:6789
```

#### Редактирование конфига, добавление информации о 2м и 3м мониторе

```bash
vim /etc/ceph/ceph.conf
```

```ini
[global]
fsid = 02ecfc51-3db2-451f-8f1c-ef5f62c275fe
public network = 192.168.122.0/24

osd pool default pg num = 128
osd pool default pgp num = 128
osd journal size = 1024
osd pool default size = 3

[mon]
mon initial members = node01
mon host = node01, node02, node03
mon addr = 192.168.122.198,192.168.122.77,192.168.122.103

[mon.node01]
host = node01
mon addr = 192.168.122.198

[mon.node02]
host = node02
mon addr = 192.168.122.77

[mon.node03]
host = node03
mon addr = 192.168.122.103
```

#### Запуск сервиса 3го монитора

```bash
systemctl start ceph-mon@node03
```


```bash
systemctl status ceph-mon@node03
```

#### Проверка работы третьего монитора
```bash
ceph -s
```

### Настройка менеджеров (MGR)


#### Выполнение на node01
#### Генерация ключа пользователя и выдача прав на профиль менеджера и доступ к сервисам osd\mds
```bash
ceph auth get-or-create mgr.node01 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
```

Вывод:
```bash
[mgr.node01]
	key = AQBmDo1mJmdaOhAAoftwU4nyd82F+Nhim8d9Cg==
```

#### Создание каталога для сервиса менеджера
```bash
mkdir -p /var/lib/ceph/mgr/ceph-node01
```

#### Вставка вывода полученного ранее в файл:
```bash
vim /var/lib/ceph/mgr/ceph-node01/keyring
```

#### Исправление прав на файл
```bash
chown -R ceph:ceph /var/lib/ceph/mgr/ceph-node01/keyring
```

#### Запуск сервиса менеджера 
```bash
systemctl start ceph-mgr@node01
```

#### Проверка статуса сервиса менеджера 
```bash
systemctl status ceph-mgr@node01
```

#### Выполнение на node02
#### Генерация ключа пользователя и выдача прав на профиль менеджера и доступ к сервисам osd\mds
```bash
ceph auth get-or-create mgr.node02 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
```

Вывод:
```bash
[mgr.node02]
	key = AQDbEI1mw0IEMxAA/MHBCB0HP9BIGtUDYALRQA==
```

#### Создание каталога для сервиса менеджера
```bash
mkdir -p /var/lib/ceph/mgr/ceph-node02
```

#### Вставка вывода полученного ранее в файл:
```bash
vim /var/lib/ceph/mgr/ceph-node02/keyring
```

#### Исправление прав на файл
```bash
chown -R ceph:ceph /var/lib/ceph/mgr/ceph-node02/keyring
```

#### Запуск сервиса менеджера 
```bash
systemctl start ceph-mgr@node02
```

#### Проверка статуса сервиса менеджера 
```bash
systemctl status ceph-mgr@node02
```

#### Выполнение на node03
#### Генерация ключа пользователя и выдача прав на профиль менеджера и доступ к сервисам osd\mds
```bash
ceph auth get-or-create mgr.node03 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
```

Вывод:
```bash
[mgr.node03]
	key = AQD1EI1mpywmNhAAnlF0zcB42NdiUxuhfVmwHw==
```

#### Создание каталога для сервиса менеджера
```bash
mkdir -p /var/lib/ceph/mgr/ceph-node03
```

#### Вставка вывода полученного ранее в файл:
```bash
vim /var/lib/ceph/mgr/ceph-node03/keyring
```

#### Исправление прав на файл
```bash
chown -R ceph:ceph /var/lib/ceph/mgr/ceph-node03/keyring
```

#### Запуск сервиса менеджера 
```bash
systemctl start ceph-mgr@node03
```

#### Проверка статуса сервиса менеджера 
```bash
systemctl status ceph-mgr@node03
```

### Настройка OSD


> Важно! В данном примере установка OSD выполняется на те же ноды. В случе необходиомсти установки на новые ноды, нужно будет скопировать конфигурационный файл и ключ, как это было при настройке мониторов на noed02/node03. Также, требуется скопировать ключ для bootstrap, иначе ceph выдаст ошибку при попытке создания блочного устройства osd.

#### Проверка ранее созданного ключа bootstrap на node01
```bash
ls -alF /var/lib/ceph/bootstrap-osd/
```


```bash
cat /var/lib/ceph/bootstrap-osd/ceph.keyring 
```

Вывод:
```bash
[client.bootstrap-osd]
	key = AQClCY1mjIAMEBAAOgctoXdX2mWhcoAvusnnng==
	caps mgr = "allow r"
	caps mon = "profile bootstrap-osd"
```

#### Архивирование bootstrap ключа на node01
```bash
tar cvzf /tmp/osd.tar.gz /var/lib/ceph/bootstrap-osd/
```

#### Копирование архива с bootstrap ключом на node02 и node03
```bash
scp /tmp/osd.tar.gz root@node02:/tmp
```

```bash
scp /tmp/osd.tar.gz root@node03:/tmp
```

#### Распаковка архива на node02
```bash
tar xzvf /tmp/osd.tar.gz
```


#### Перемещение ключа на node02
```bash
mv var/lib/ceph/bootstrap-osd/ceph.keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
```

```bash
chown ceph:ceph /var/lib/ceph/bootstrap-osd/ceph.keyring
```


#### Распаковка архива на node03
```bash
tar xzvf /tmp/osd.tar.gz
```

#### Перемещение ключа на node03
```bash
mv var/lib/ceph/bootstrap-osd/ceph.keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
```

```bash
chown ceph:ceph /var/lib/ceph/bootstrap-osd/ceph.keyring
```

#### Создание блочного устройства для osd на node01


> К виртуальной машине подключен отдельный диск  /dev/vdb. Список доступных дисков можно получить с помощью команды lsblk


```bash
ceph-volume lvm create --data /dev/vdb

```

#### Создание блочного устройства для osd на node02


> К виртуальной машине подключен отдельный диск  /dev/vdb. Список доступных дисков можно получить с помощью команды lsblk


```bash
ceph-volume lvm create --data /dev/vdb
```

#### Создание блочного устройства для osd на node03


> К виртуальной машине подключен отдельный диск /dev/vdb. Список доступных дисков можно получить с помощью команды lsblk


```bash
ceph-volume lvm create --data /dev/vdb
```

### Проверка статуса ceph
```bash
ceph -s
```

> В процессе сборки кластера ceph могут возникнуть проблемы в виде *_WARN и остаться даже после полной сборки



#### Архифирование всех отчетов об ошибках
```bash
ceph crash archive-all
```


#### Отключение модуля restful
```bash
ceph mgr module disable restful
```

#### Переключение протокол обмена сообщениями между мониторами на 2-ю версию
```bash
ceph mon enable-msgr2
```

#### Отключение insecure global_id
```bash
ceph config set mon mon_warn_on_insecure_global_id_reclaim false
```


```bash
ceph config set mon mon_warn_on_insecure_global_id_reclaim_allowed false
```

#### Повторная проверка статуса ceph
```bash
ceph -s
```


### Подключение клиента к ceph


#### Создание osd pool подключившись к node01
```bash
ceph osd pool create rbd 32
```


#### Просмотр созданного osd pool
```bash
ceph osd lspools
```


#### Создание пользователя и предоставление ему доступа к pool
```bash
ceph auth get-or-create client.rbd1 mon 'profile rbd' osd 'profile rbd pool=rbd' mgr 'profile rbd pool=rbd'
```

Вывод:
```bash
[client.rbd1]
	key = AQCTIY1m7Su3DhAAvRbqILMwYczEgGohld/m7Q==
```

#### Копирование конфигурационного файла ceph на клиентский хост (client)
```bash
scp /tmp/config.tar.gz root@client:/tmp
```

#### Установка ключа архива на хосте client (добавить ключ из ранее полученного вывода команды ceph auth get-or-create)
```bash
mkdir /etc/ceph
```

```bash
vim /etc/ceph/ceph.client.rbd1.keyring
```


#### Распаковка архива на хосте client
```bash
tar xzvf /tmp/config.tar.gz
```

```bash
mv etc/ceph/ceph.conf /etc/ceph/
```

#### Получение вывода rbd с идентификатором пользователя и проверка, что нет image
```bash
rbd --id rbd1 ls
```

#### Создание image
```bash
rbd create --id rbd1 --size 1024 image1
```


#### Проверка image
```bash
rbd --id rbd1 ls
```

#### Выполнение маппинга диска на client (подключить как блочное устройство)
```bash
rbd map image1 --id rbd1
```


#### Создание файловой системы xfs на блочном устройстве
```bash
mkfs.xfs /dev/rbd0
```

#### Монтирование в директорию
```bash
mkdir /mnt/rbd0
```

```bash
mount /dev/rbd0 /mnt/rbd0
```

#### Проверка смонтированного каталога
```bash
ls -alF /mnt/rbd0
```

```bash
df -hT
```