# Добавление диска в Vagrantfile
Для добавление диска нужно добавить следующий блок в Vagrantfile.
```
:sata5 => {
 :dfile => './sata5.vdi', # Путь, по которому будет создан файл диска
 :size => 250, # Размер диска в мегабайтах
 :port => 5 # Номер порта на который будет зацеплен диск
},
```
# Сборка RAID 6
Занулим суперблок:
```
 mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
 
```
Далее создаем рейд следующей командой:

```
mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
```
В данном примере мы собираем RAID 6, указываем после ключа -l. Ключ -n указывает на колличество дисков.
# Создание конфигурационного файла mdadm.conf.
```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```
# Создание GPT раздела, пяти партиций и монтирование их на диск
Создание раздаела GPT на RAID
```
 parted -s /dev/md0 mklabel gpt
```
Создаем партиции 
```
 parted /dev/md0 mkpart primary ext4 0% 20%
 parted /dev/md0 mkpart primary ext4 20% 40%
 parted /dev/md0 mkpart primary ext4 40% 60%
 parted /dev/md0 mkpart primary ext4 60% 80%
 parted /dev/md0 mkpart primary ext4 80% 100%
```
Далее создаем на этих партициях файловую систему.
```
 for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
```
И монтируем их по каталогам
```
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```
# Скрипт создания RAID 6
```
sudo -i
mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
mkdir  /etc/mdadm
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
parted -s /dev/md0 mklabel gpt
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```
# Добаление скрипта в Vagrantfile
Для добавления скрипта в Vagrantfile необходимо добавить модуль :
```
box.vm.provision "shell", path: "build_raid6.sh"
```


