### Установка ceph
```bash
sudo apt install ceph -y
```

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


#### Настройка второго монитора
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

#### Настройка третьего монитора
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

