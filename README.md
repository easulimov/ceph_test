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

#### Запуск сервиса

```bash
systemctl start ceph-mon@node-01
```

```bash
systemctl status ceph-mon@node-01
```