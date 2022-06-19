# Перенос системы на RAID-1 в одном скрипте mdadm
Для создания массива используется пакет mdadm и создается массив RAID-1 на ДВУХ диска(возможны добавления функционала: другие виды RAID и более гибкий выбор дисков для создания массива.
Скрипт выполняет следующие действия:
  1. Выбирает вид RAID массива (пока только RAID-1)
  2. Дает выбор использовать текущий диск для добавления(пока не работает) в массив или использовать два новых диска
  3. Форматирование дисков для RAID массива. Создается раздел на весь диск с флагов raid autodetect
  4. Создание массива md0 средствами mdadm (raid level=1, raid-devices=2) + добавление конфигурации массива в конфиг mdadm.conf
  5. Перенос разделов с текущей системы на md0 + форматирование разделов(фиксированные разделы, надо доработать)
  6. Перенос данных системы на md0 средствами rsync. Для мониторинга прогресса можно в параллельном окне запустить ```watch -n 1 df``` и смотреть диски /dev/sda1 и /dev/md0p1
  7. Замена uuid разделов в файле fstab на диске md0 для корректной загрузки системы
  8. Переход chroot в диск массива, смонтированный в /mnt
  9. Запуск в среде chroot скрипта chroot.sh:
    1. Обновление конфига grub для изменения menuentry с учетом запуска с рейдового диска (корректируются диски, uuid)
    2. Установка grub на два рейдовых диска
    3. Обновление образа файловой системы, который загружается вместе с ядром при старте компьютера для учета конфига mdadm.conf (поиска рейдовых разделов)
    4. Добавление в файл /etc/default/grub 
GRUB_RECORDFAIL_TIMEOUT=10 (добавляем, чтобы система могла загрузиться и в случае ошибки, не требуя интерактивного вмешательства; таймаут по вкусу);
GRUB_CMDLINE_LINUX_DEFAULT=«bootdegraded» (обязательно добавляем «bootdegraded», чтобы система могла загрузиться с неполноценного массива) - актуально при использовании текущего диска при построении массива. Добавлено "для следующих улучшений скрипта".

## Для работы необохдимо
  
  1. Диск с MBR разметкой (разделы Авто - /home и swap)
 Скрипт копирует разделы аналогично исходному диску, но форматирование разделов не автоматизировано и форматирует только / в ext4 и swap 
  3. Пакеты mdadm, parted, rsync
 Без этих пакетов не будет функционировать скрипт

!!! Пока работает только вариант с RAID-1 для двух новых дисков(без текущего).

## Использование
```bash
git clone https://github.com/lxg7/raid1builder.git
cd raid1builder
sudo ./raid1builder.sh
```
 Далее необходимо следовать инструкциям
 
 ## To-Do
 - Доделать вариант работы с (текущим диском + 1 новый) => raid
 - (???) Добавить RAID-0 
 - форматирование разделов в md0 для работы с разным количеством разделов
