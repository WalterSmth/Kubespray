![](https://www.infracloud.io/assets/img/Blog/Blog-updated-creatives/index-image-1200x628/ha-kubernetes-with-kubespray-1200x628.png)

# Поднимаем kubernetes с помощью Kubespray

### Действия, выполняемые на всех серверах
На каждом сервере обновляем пакеты
```sh
sudo apt update
```

На каждом сервере отключаем файл подкачки (swap-файл). Со включенным swap-файлом kubernetes отказывается работать

```sh
sudo swapoff -a
```
Редактируем файл fstab. Делается это, чтобы после перезагрузки сервера файл подкачки не включался снова. Закомментируем, добавив # в начало строки, (или просто удалим) последнюю строку (в которой упоминается файл /swap.img).
```sh
nano /etc/fstab
```

На каждом сервере отключаем firewall
```sh
sudo firewall-cmd --state
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

На каждом сервере создаём ssh ключи и прокидываем их на каждый сервер. Также нужно прокинуть ssh-ключ с главной машины, где будете инициировать kubernetes, на неё саму же.
```sh
ssh-keygen
ssh-copy-id root@<host IP>
```

### Действия, выполняемые на главном сервере
Ставим pip
```sh
sudo apt install python3-pip
```

Клонируем репозиторий kubespray
```sh
git clone https://github.com/kubernetes-sigs/kubespray.git
```

Настраиваем окружение для Ansible
```sh
VENVDIR=kubespray-venv
KUBESPRAYDIR=kubespray
ANSIBLE_VERSION=2.12	#Версию Ansible выбирайте по требованиям kubespray
virtualenv --python=$(which python3) $VENVDIR
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements-$ANSIBLE_VERSION.txt
```

Клонируем базовую конфигурацию кластера. Вместо mycluster можно написать любое название
```sh
cp -rfp inventory/sample inventory/mycluster
```

Прописываем IP-адреса наших серверов. Здесь исползуются условные IP для примера
```sh
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

Запускаем установку нашего кластера. Установка может занять от нескольких минут до часа
```sh
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

После завершения установки проверяем наш кластер
```sh
kubectl get nodes
```

Вывод будет примерно такой
```sh
NAME    STATUS   ROLES           AGE   VERSION
node1   Ready    control-plane   26m   v1.26.5
node2   Ready    control-plane   25m   v1.26.5
node3   Ready    <none>          24m   v1.26.5
```

#### Источники
https://proglib.io/p/pervoe-znakomstvo-s-kubernetes-ustanovka-klastera-k8s-vruchnuyu-2021-05-21

https://github.com/kubernetes-sigs/kubespray/tree/master

https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible.md#installing-ansible
