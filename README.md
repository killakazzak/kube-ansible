# Ansible playbook для установки  кластера k8s


| k8s ver         | Distributive    | CRI             | Notes           |
|-----------------|-----------------|-----------------|-----------------|
| 1.30            | Rocky Linux 8.10 | containerd 1.6.32 | **OK**               |

Поддерживает:

- Kubernetes v1.30.
- Установку одной или несколько control nodes.
- HA доступ к API kubernetes.
- calico.
- В KubeProxyConfiguration установлены параметры для работы Metallb.
- nodelocaldns - кеширующий DNS сервер на каждой ноде кластера.

## Установка ansible

```shell
yum install ansible -y
```

Генерируем ssh ключ:

```shell
ssh-keygen
```

Копируем ключики в виртуальные машины из [hosts.yaml]([hosts.yml](https://github.com/killakazzak/kube-ansible/blob/main/hosts.yaml)):

 ```shell
ssh-copy-id root@ismail-ha101p.oblako.local
ssh-copy-id root@ismail-ha102p.oblako.local
ssh-copy-id root@ismail-ctr101p.oblako.local
ssh-copy-id root@ismail-ctr102p.oblako.local
ssh-copy-id root@ismail-ctr103p.oblako.local
ssh-copy-id root@ismail-data101p.oblako.local
ssh-copy-id root@ismail-data102p.oblako.local
ssh-copy-id root@ismail-data103p.oblako.local
```

## Конфигурационные параметры

* [Инвентори](https://github.com/killakazzak/kube-ansible/blob/main/hosts.yaml).
* [Общая конфигурация](group_vars/k8s_cluster).

## Установка

### k8s с одной control node

В [инвентори](hosts.yaml) в группе `k8s_masters` необходимо указать только один хост.

```shell
ansible-playbook install-cluster.yaml
```

### k8s с несколькими control nodes

В [инвентори](hosts.yaml) в группе `k8s_masters` необходимо указать **нечётное количество
control nodes**.

```shell
ansible-playbook install-cluster.yaml
```

### k8s c HA

Используются haproxy и keepalived.

![ha cluster](images/ha_cluster.jpg)

В конфигурационном файле определите параметры доступа к API :

* `ha_cluster_virtual_ip` - виртуальный IP адрес.
* `ha_cluster_virtual_port` - порт. Не должен быть равен 6443.

## Удалить кластер

```shell
ansible-playbook reset.yaml
```

**Внимание!!!** Скрипт удаляет **все** нестандартные цепочки и чистит все стандартные цепочки.

## Апдейт кластера

Изменяете версию кластера в `group_vars\k8s_cluster` и запускаете апдейт.

```shell
ansible-playbook upgrade.yaml
```

## Сервисные функции

Сервисные функции находятся в директории `services`
