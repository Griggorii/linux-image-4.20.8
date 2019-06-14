linux-image-4.20.8 download https://dropmefiles.com.ua/eWKHA4 сначала пере копировать библиотеки , для OS 8.0  на базе 18.10 , OS 11 на базе 19.04 дистрибутивы которые я создал наити у меня на гит в библиотеках уже не нуждается ставить сначала header kernel потом остальные деб пакеты.

Ядро проверено поддерживает 3G , 4G , LTE устройства всё работает

Поддерживает почти всё оборудование , но не забываем в отличие от массы чсв я прошел монах шаолинь линукс , так что это ядро и компилировалось на моём же созданном дистрибутиве OS 11 на базе 16.04 

Я знаю многие ищут такое ядро которое будет везде работать и посковик будет забит вопросами типа обновил ядро теперь нет интернета или плохо монтируется флешка или вообще теперь не монтируется ничего , в этой золотой версии ядра можно уже не боятся написать новость вот Griggorii выпустил ядро которое работает не то что у макак.

Что надо сделать пред установкой моего патентного ядра 1) Удалите все старые ядра и вычестите каталоги от их остатков в мною указанных местах от хозяина рута /usr/src удалите там старый кернел /lib/modules и отсюда его долой , посмотрите чтобы в корневом каталоге у меня допустим он /  не было ссылок по типу initrd.img.old vmlinuz.old все они должны быть обязательно удалены почему читать чуть выше. Не сделаете удаление того чего описал здесь то получите ошибки.

Что я ещё узнал что если vmlinuz если нормальный то тиринга возможно у вас вообще не будет, по скольку он каким то образом делает работу в системе и естественно компилировался либо в среде плохого wm (с тирингом) либо в среде wm (без тиринга) и видимо берет дампы при компиляции и ничего тут не поделаешь , как же я проверил скажут многие особенно с чсв , да просто подменил vmlinuz-4.20.8 в одном дистрибутиве и создал на него ссылку vmlinuz в грубе сразу появился тиринг так это ещё только в грубе.

После установки и перезагрузки на новое ядро создайте фаил с расширением 1.sh

Скачайте 1.sh или создайте скопируйте текст ниже и вставьте этот текст в 1.sh и сделайте исполняемым и в терминале от суперпользователя выполните его , так же ошметки от старого ядра сохраняются в /lib/modules естественно удаляем оттуда старые остатки ядра не трогая 4.20.8


#!/bin/bash
 
LOG="/root/Clean_system.log"
GREEN="\033[1;32m"
RED="\033[0;31m"
YELLOW="\033[1;33m"
ENDCOLOR="\033[0m"
 
OLDCONF=$(dpkg -l|grep "^rc"|awk '{print $2}')
CURKERNEL=$(uname -r|sed 's/-*[a-z]//g'|sed 's/-386//g')
LINUXPKG="linux-(image|headers|ubuntu-modules|restricted-modules)"
METALINUXPKG="linux-(image|headers|restricted-modules)-(generic|i386|server|common|rt|xen)"
OLDKERNELS=$(dpkg -l|awk '{print $2}'|grep -E $LINUXPKG |grep -vE $METALINUXPKG|grep -v $CURKERNEL)
 
if [ $USER != root ]; then
  echo -e $RED"Вы должны быть root"
  exit 0
fi
 
echo "------------------------------------------------------------------------------" >> $LOG
date  >> $LOG
echo "------------------------------------------------------------------------------" >> $LOG
 
echo -e $YELLOW"Чистка пакетов запущена\n"$ENDCOLOR
echo -e $YELLOW"Bleachbit - приложения"$ENDCOLOR
bleachbit_cli -do  bash.history \
                   deepscan.backup \
                   deepscan.ds_store \
                   deepscan.thumbs_db \
                   deepscan.tmp \
                   firefox.cache \
                   firefox.cookies \
                   firefox.download_history \
                   firefox.forms \
                   firefox.places \
                   firefox.session_restore \
                   firefox.url_history \
                   flash.cache \
                   flash.cookies \
                   gedit.recent_documents \
                   gimp.tmp \
                   gnome.run \
                   java.cache \
                   kde.cache \
                   kde.recent_documents \
                   kde.tmp \
                   nautilus.history \
                   openofficeorg.cache \
                   openofficeorg.recent_documents \
                   pidgin.cache \
                   pidgin.logs \
                   system.cache \
                   system.clipboard \
                   system.desktop_entry \
                   system.free_disk_space \
                   system.localizations \
                   system.memory \
                   system.recent_documents \
                   system.rotated_logs \
                   system.tmp \
                   system.trash \
                   thumbnails.cache \
                   wine.tmp \
                   winetricks.temporary_files \
                   x11.debug_logs 2>> $LOG
 
echo -e $YELLOW"\nУдаление кэша apt..."$ENDCOLOR
apt-get -y clean 2>> $LOG
 
echo -e $YELLOW"Удаление частичных пакетов..."$ENDCOLOR
apt-get -y autoclean 2>> $LOG
 
echo -e $YELLOW"Удаление старых конфигов..."$ENDCOLOR
apt-get -y purge $OLDCONF 2>> $LOG
 
echo -e $YELLOW"Удаление незадействованных пакетов..."$ENDCOLOR
while [ -n "`deborphan`" ]; do
    deborphan
    echo
    apt-get -y purge `deborphan` 2>>$LOG
done
 
echo -e $YELLOW"Удаление старых ядер..."$ENDCOLOR
apt-get -y purge $OLDKERNELS 2>> $LOG
 
echo -e $YELLOW"Удаление пакетов, установленных по зависимостям и которые больше не нужны..."$ENDCOLOR
apt-get -y autoremove --purge 2>> $LOG
 
echo -e $YELLOW"Чистим корзины пользователей..."$ENDCOLOR
rm -rf /home/*/.local/share/Trash/*/** 2>> $LOG
rm -rf /root/.local/share/Trash/*/** 2>> $LOG
 
echo -e $GREEN"------------------------------------------------------------------------------"$ENDCOLOR >> $LOG
echo -e $RED"Очистка завершена!"$ENDCOLOR >> $LOG
echo -e $GREEN"------------------------------------------------------------------------------"$ENDCOLOR >> $LOG
cat "$LOG"
echo -e $GREEN"------------------------------------------------------------------------------"$ENDCOLOR >> $LOG


