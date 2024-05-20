homework-RAID
Описание домашнего задания
---
1. добавить в Vagrantfile еще дисков;
2. собрать R0/R5/R10 на выбор;
3. прописать собранный рейд в конф, чтобы рейд собирался при загрузке;
4. сломать/починить raid;
5. создать GPT раздел и 5 партиций и смонтировать их на диск.

---
- Этап 1: Vagrantfile изменен, прописан код для добавления дисков в ВМ
   
![images2](./images/image_raid_1.png)
![images2](./images/image_raid_2.png)

```bash
sudo fdisk -l
```
![images2](./images/image_raid_3.png)

- Этап 2: Собран R10 
  
![images2](./images/image_raid_4.png)

Занулим на всякий случай суперблоки

![images2](./images/image_raid_5.png)

Ответ команды дает понять, что диски не использовались ранее для RAID. продолжаем

Собираем raid
![images2](./images/image_raid_6.png)

Проверяем

![images2](./images/image_raid_7.png)

Раз у нас остался пятый диск, можно попробовать добавить его в raid в качестве горячей замены
![images2](./images/image_raid_8.png)
![images2](./images/image_raid_9.png)

- Этап 3: Пропишем собранный рейд в конфиг, чтобы рейд собирался при загрузке
```bash
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
# Тут так просто не получилось, пришлось заходить в суперпользователя
```
![images2](./images/image_raid_10.png)

Убедимся, что файл записан
![images2](./images/image_raid_11.png)

- Этап 4: Сломаем, а затем починим raid
```bash
mdadm /dev/md0 --fail /dev/sde
```
![images2](./images/image_raid_12.png)

Ну, в моем случае подтянулся диск горячей замены)), сломаем еще немного, "выдернув" еще диск
![images2](./images/image_raid_13.png)
![images2](./images/image_raid_14.png)

Удалим "сломанный" диск из рейда и вернем обратно, будто новый
![images2](./images/image_raid_15.png)
```bash
#момент построения рейда я пропустил, слишком быстро было
```
![images2](./images/image_raid_16.png)

- Этап 5: Создадим GPT раздел и 5 партиций и смонтируем их на диск
```bash
parted -s /dev/md0 mklabel gpt #Создаем раздел GPT на RAID
parted /dev/md0 mkpart primary ext4 0% 20% #Создаем партиции (5 комманд с шагом в 20%) 
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done # Создаем на этих партициях ФС
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done # Монтируем их по каталогам
```
![images2](./images/image_raid_17.png)
![images2](./images/image_raid_18.png)
