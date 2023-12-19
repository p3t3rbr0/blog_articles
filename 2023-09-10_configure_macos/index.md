[META]:

```
title: Настройка MacOS
tags: recipes, macos
meta-desc: Настройка MacOS. Первичная настройка. Удаление неиспользуемых приложений. Отключение избыточных сервисов. Отключение IPv6. Отключение atime-аттрибута.
```

---

MacOS — операционная система для состоятельных американских домохозяек, которая, по какой-то причине, пользуется популярностью у разработчиков. Есть подозрение, что паттерны поведения первых и вторых схожи). Но не будем унижать достопочтенных домохозяек таким вульгарным сравнением, лучше поговорим о самой ОС.

Тормозная в повседневном использовании, прожорливая к ресурсам, изобилующая совершенно лишними аудиовизуальными эффектами. С тонной предустановленного софта, по типу: Stocks, TV, Podcasts, FaceTime, FindMy<del>Fuck</del>Mac и т.п. С расставленной на каждом углу телеметрией, и принудительной синхронизацией всего и вся с apple-облаком.

Плюс ко всему, она не шибко располагает средствами для настройки. Только штатная утилита "System Settings", с которой особо не разгуляешься.

Однако кое-какой потенциал для тюнинга все еще остается, даром что ли это полноценная POSIX-система? Попробуем воспользоваться имеющимся потенциалом, чтобы снизить градус отвращения от пользования данной ОС.

---

> Приведенные в статье "рецепты" актуальны для MacOS: Big Sur, Monterey и Ventura. Для других версий также должно сработать, но с некоторыми правками.

1. [Первичная настройка](#1-первичная-настройка)
2. [Удаление неиспользуемых приложений](#2-удаление-неиспользуемых-приложений)
3. [Отключение избыточных сервисов](#3-отключение-избыточных-сервисов)
4. [Отключение IPv6](#4-отключение-ipv6)
5. [Отключение atime-аттрибута](#5-отключение-atime-аттрибута)
6. [Итог](#6-итог)

# 1. Первичная настройка

Выполняем минимально необходимую настройку через штатные средства "System Settings". Отрубаем все лишнее.

## 1.1 Отключение пользовательской телеметрии

> Есть еще "системная" телеметрия, которую через System Settings отключить никак не получится. Ее мы подрежем в разделее про отключение избыточных сервисов.

`System Settings` > `Security & Privacy` > `Analytics & Improvements`

## 1.2 Отключение Focus

`System Settings` > `Focus`

## 1.3 Отключение Screen Time

`System Settings` > `Screen Time`

## 1.4 Отключение автообновления

`System Settings` > `General` > `Software Update`

## 1.5 Удаление элементов из автозапуска

`System Settings` > `General` > `Login Items`

## 1.6 Отключение синхронизации AirDrop с iCloud

`System Settings` > `General` > `AirDrop с iCloud`

## 1.7 Отключение всего в Accessibility

`System Settings` > `Accessibility`

## 1.8 Отключение лишнего в Control Center

`System Settings` > `Control Center`

Я вырубил все, кроме отображения статусов: Wi-Fi, Battery и Clock в статусбаре. Остальное все отключил.

## 1.9 Отключение голосового помощника и фоновый индексации

`System Settings` > `Siri & Spotlight`

Вырубаем все галочки/переключатели полностью, везде.

## 1.10 Настройка полномочий для программ

Запрещаем программам определять наше местоположение (переводим все в ноль):

`System Settings` > `Privacy & Security` > `Location Services`

`System Settings` > `Privacy & Security` > `Contacts` — в ноль.

Далее проходимся по каждому из подразделов и выдаем/запрещаем те или иные полномочия (камера, микрофон и т.п.) для целевых приложений.

Отбирайте все полномочия для программ, которыми вы не пользуетесь на систематической основе. Если что — включить можно будет всегда (а вот отключать потом... вы вряд ли полезете в настройки).

## 1.11 Отключение Game Center

`System Settings` > `Game Center`

...

Дальнейшие настройки рассматривать не буду, основной принцип — отключить все, что не является вами активно используемым.

# 2. Удаление неиспользуемых приложений

Определяем список из приложений и утилит, которые нам не нужны и удаляем их. Однако просто так, из "Finder > Applications" их удалить не получится. Благо, на уровне ФС, встроенные приложения представляют собой просто директории в `/System/Applications`. И, кажется, что можно просто взять и удалить целевые каталоги программ, посредством `rm -rf`. Однако сделать это сходу не получится, так как корневой раздел (в который и установлена OS со всеми своими приложениями) примонтирован с флагами `ro` (Read Only). Придется его перемонтировать в кастомный каталог, предварительно отключив "защиту от дурака". Делаем!

1. Отключаем в настройках шифрование системного раздела (если включено) — `System Settings` > `Privacy & Security` > `FileVault`.

2. Перезагружаемся в "режим восстановления" (при загрузке зажимаем и удерживаем клавиши `Command+R`), запускаем `Terminal`.

3. Отключаем систему `SIP` (System Integrity Protection) и защиту небезопасных действий от root.

В терминале выполняем:

    :::shell-session
    $ csrutil disable
    $ csrutil authenticated-root disable

4. Перезагружаемся в обычный режим, авторизуемся.

Окончательно определяемся со списком ПО, которое планируем удалить. Свой набор ненужного софта я запечатлел в shell-скрипте:

    :::shell
    #!/bin/sh

    #
    # Remove unused MacOS native applications.
    #
    # Usage: [MNT=<mount point>] [DISK=<block device filename>] ./remove_macos_apps.sh
    #
    # Example: MNT=~/my/mnt DISK=/dev/disk1s5 ./remove_macos_apps.sh
    #

    set -eu

    MNT=${MNT:-$HOME/opt/mnt}
    DISK=${DISK:-/dev/disk1s5}

    APPLICATIONS_PATH=$MNT/System/Applications
    UTILITIES_PATH=$APPLICATIONS_PATH/Utilities

    mount -o nobrowse -t apfs $DISK $MNT

    size=0

    for application in \
        "Automator" \
        "App Store" \
        "Books" \
        "Contacts" \
        "Dictionary" \
        "FaceTime" \
        "FindMy" \
        "Freeform" \
        "Home" \
        "Mail" \
        "Maps" \
        "Messages" \
        "Music" \
        "News" \
        "Notes" \
        "Photos" \
        "Podcasts" \
        "Reminders" \
        "Siri" \
        "Stickies" \
        "Stocks" \
        "Shortcuts" \
        "Time Machine" \
        "TV" \
        "TextEdit" \
        "VoiceMemos" \
        "Weather";
    do
        echo "Deleting application: ${application} ..."
        size=$(($size + $(du -sm "${APPLICATIONS_PATH}/${application}.app" |awk '{ print $1 }')))
        rm -rf "${APPLICATIONS_PATH}/${application}.app"
    done

    for utility in \
        "Activity Monitor" \
        "Bluetooth File Exchange" \
        "Boot Camp Assistant" \
        "ColorSync Utility" \
        "Console" \
        "Digital Color Meter" \
        "Grapher" \
        "Keychain Access" \
        "Migration Assistant" \
        "Screenshot" \
        "Script Editor" \
        "VoiceOver Utility";
    do
        echo "Deleting utility: ${utility} ..."
        size=$(($size + $(du -sm "${UTILITIES_PATH}/${utility}.app" |awk '{ print $1 }')))
        rm -rf "${UTILITIES_PATH}/${utility}.app"
    done

    echo
    echo "Released ${size} Mib."

    echo
    echo "Creating fs snapshot and unmounting ..."
    bless --folder $MNT/System/Library/CoreServices --bootefi --create-snapshot
    diskutil unmount $MNT

    echo
    echo "Done"

Скрипт выполняется от рута (sudo) и делает следующее: монтирует системный раздел с поддержкой записи файлов, удаляет набор перечисленных приложений и утилит, создает новый снимок файловой системы (после внесенных нами изменений), размонтирует раздел.

В моем случае, после работы скрипта, высвобождается ~500 Mib диска, а в меню приложений (Launchpad) — чистота и порядок.

> Стоит иметь в виду, что программы удаляются до следующего обновления системы. Т.е. скрипт придется запускать всякий раз после обновления/переустановки.

# 3. Отключение избыточных сервисов

По умолчанию ОС запускает невероятное количество фоновых сервисов: синхронизаторы с облаком, репортеры, индексаторы, телеметраторы, ассистенты, игровые сервисы, сервисы телефонии и монетизации, и тонны другого силоса. Все это можно и нужно отключить.

В MacOS сервисы могут быть представлены двумя сущностями: демонами и агентами. И то, и другое, на уровне ОС — просто программы, запущенные в фоне. Однако демоны запускаются от привилегированного пользователя, при старте ОС, а агенты — после авторизации обычного пользователя. Для каждого пользователя может быть различное количество агентов. Кроме того, агенты могут быть запущены другими агентами при определенных условиях или же запускаться периодически, как крон-скрипты.

И демоны, и агенты, могут быть представлены как бинарными, исполняемыми файлами, так и текстовыми xml-сценариями.

Демоны располагаются в каталогах:

- `/System/Library/LaunchDaemons` — системные демоны;
- `/Library/LaunchDaemons` — не системные демоны, общие для всех пользователей;

Пользовательские агенты располагаются в каталогах:

- `/System/Library/LaunchAgents` — системные агенты (входят в состав macOS), запущенные от имени пользователя;
- `/Library/LaunchAgents` — общие (но не системные) агенты для всех пользователей;
- `~/Library/LaunchAgents` — агенты текущего пользователя;

Для управления демонами и агентами используем утилиту `launchctl(1)`.

Задумка простая — определяем (посредством просмотра содержимого каталогов по путям, описанным выше) какие из сервисов мы считаем лишними (по названию сервиса + гугл для непонятных слов), и отключаем их.

Сам процесс отключения я запечатлел в shell-скрипте:

> Отключение сервисов также требует отключение SIP. Если вы этого еще не сделали, то перейдите раздел про удаление программ и следуйте инструкции по отключению "защиты от дурака".

    :::shell
    #!/bin/sh

    #
    # Disable redundant macos services (daemons and agents).
    #
    # Usage: [ID=<user id>] ./disable_macos_services.sh
    #
    # Example: ID=502 ./disable_macos_services.sh
    #

    set -eu

    ID=${ID:-501}

    AGENTS=$(cat <<-END
    com.apple.accessibility.MotionTrackingAgent
    com.apple.AddressBook.ContactsAccountsService
    com.apple.AMPArtworkAgent
    com.apple.AMPDeviceDiscoveryAgent
    com.apple.AMPLibraryAgent
    com.apple.ap.adprivacyd
    com.apple.ap.adservicesd
    com.apple.ap.promotedcontentd
    com.apple.assistant_service
    com.apple.assistantd
    com.apple.avconferenced
    com.apple.BiomeAgent
    com.apple.biomesyncd
    com.apple.bird
    com.apple.CalendarAgent
    com.apple.cloudd
    com.apple.cloudpaird
    com.apple.cloudphotod
    com.apple.CloudPhotosConfiguration
    com.apple.CommCenter-osx
    com.apple.ContactsAgent
    com.apple.CoreLocationAgent
    com.apple.familycircled
    com.apple.familycontrols.useragent
    com.apple.familynotificationd
    com.apple.followupd
    com.apple.gamed
    com.apple.geod
    com.apple.homed
    com.apple.icloud.findmydeviced
    com.apple.icloud.findmydeviced.aps-demo
    com.apple.icloud.findmydeviced.aps-development
    com.apple.icloud.findmydeviced.aps-production
    com.apple.icloud.findmydeviced.findmydevice-user-agent
    com.apple.icloud.findmydeviced.ua-services
    com.apple.icloud.fmfd
    com.apple.icloud.searchpartyd
    com.apple.icloud.searchpartyd.accessorydiscoverymanager
    com.apple.icloud.searchpartyd.advertisementcache
    com.apple.icloud.searchpartyd.beaconmanager
    com.apple.icloud.searchpartyd.beaconmanager.agentdaemoninternal
    com.apple.icloud.searchpartyd.finderstatemanager
    com.apple.icloud.searchpartyd.pairingmanager
    com.apple.icloud.searchpartyd.scheduler
    com.apple.icloud.searchpartyuseragent
    com.apple.iCloudNotificationAgent
    com.apple.iCloudUserNotifications
    com.apple.imagent
    com.apple.imautomatichistorydeletionagent
    com.apple.imtransferagent
    com.apple.itunescloudd
    com.apple.knowledge-agent
    com.apple.ManagedClient.cloudconfigurationd
    com.apple.ManagedClientAgent.enrollagent
    com.apple.Maps.mapspushd
    com.apple.Maps.pushdaemon
    com.apple.mediaanalysisd
    com.apple.mediastream.mstreamd
    com.apple.newsd
    com.apple.nsurlsessiond
    com.apple.parsec-fbf
    com.apple.parsecd
    com.apple.passd
    com.apple.photoanalysisd
    com.apple.photolibraryd
    com.apple.progressd
    com.apple.protectedcloudstorage.protectedcloudkeysyncing
    com.apple.quicklook
    com.apple.quicklook.ui.helper
    com.apple.quicklook.ThumbnailsAgent
    com.apple.rapportd-user
    com.apple.remindd
    com.apple.routined
    com.apple.SafariCloudHistoryPushAgent
    com.apple.SafeEjectGPUAgent
    com.apple.screensharing.agent
    com.apple.screensharing.menuextra
    com.apple.screensharing.MessagesAgent
    com.apple.ScreenTimeAgent
    com.apple.security.cloudkeychainproxy3
    com.apple.sidecar-hid-relay
    com.apple.sidecar-relay
    com.apple.Siri.agent
    com.apple.siri.context.service
    com.apple.siriknowledged
    com.apple.suggestd
    com.apple.Spotlight
    com.apple.spotlightknowledged
    com.apple.speech.synthesisserver
    com.apple.telephonyutilities.callservicesd
    com.apple.TMHelperAgent
    com.apple.TMHelperAgent.SetupOffer
    com.apple.UsageTrackingAgent
    com.apple.videosubscriptionsd
    END
    )

    DAEMONS=$(cat <<-END
    com.apple.bootpd
    com.apple.backupd
    com.apple.backupd-helper
    com.apple.cloudd
    com.apple.cloudpaird
    com.apple.cloudphotod
    com.apple.CloudPhotosConfiguration
    com.apple.CoreLocationAgent
    com.apple.coreduetd
    com.apple.dhcp6d
    com.apple.familycontrols
    com.apple.findmymacmessenger
    com.apple.followupd
    com.apple.FollowUpUI
    com.apple.ftp-proxy
    com.apple.ftpd
    com.apple.GameController.gamecontrollerd
    com.apple.geod
    com.apple.icloud.findmydeviced
    com.apple.icloud.findmydeviced.aps-demo
    com.apple.icloud.findmydeviced.aps-development
    com.apple.icloud.findmydeviced.aps-production
    com.apple.icloud.findmydeviced.findmydevice-user-agent
    com.apple.icloud.findmydeviced.ua-services
    com.apple.icloud.fmfd
    com.apple.icloud.searchpartyd
    com.apple.icloud.searchpartyd.accessorydiscoverymanager
    com.apple.icloud.searchpartyd.advertisementcache
    com.apple.icloud.searchpartyd.beaconmanager
    com.apple.icloud.searchpartyd.beaconmanager.agentdaemoninternal
    com.apple.icloud.searchpartyd.finderstatemanager
    com.apple.icloud.searchpartyd.pairingmanager
    com.apple.icloud.searchpartyd.scheduler
    com.apple.icloud.searchpartyuseragent
    com.apple.iCloudHelper
    com.apple.iCloudNotificationAgent
    com.apple.iCloudUserNotificationsd
    com.apple.itunescloudd
    com.apple.ManagedClient.cloudconfigurationd
    com.apple.netbiosd
    com.apple.nsurlsessiond
    com.apple.protectedcloudstorage.protectedcloudkeysyncing
    com.apple.rapportd
    com.apple.screensharing
    com.apple.security.cloudkeychainproxy3
    com.apple.siri.morphunassetsupdaterd
    com.apple.siriinferenced
    END
    )

    for agent in $AGENTS; do
        echo "Disable agent: $agent ..."
        launchctl bootout gui/$ID/$agent || true
        launchctl disable gui/$ID/$agent || true
    done
     for daemon in $DAEMONS; do
        echo "Disable daemon: $daemon ..."
        sudo launchctl bootout system/$daemon || true
        sudo launchctl disable system/$daemon || true
    done

> Если на устройстве не используется WiFi, то желательно отключить соотвествующие агенты: `com.apple.wifi.WiFiAgent` и демоны: `com.apple.airportd`, `com.apple.wifi.WiFiAgent`, `com.apple.diagnosticextensions.osx.wifi.helper`, `com.apple.wifianalyticsd`, `com.apple.wifiFirmwareLoader`, `com.apple.wifip2pd`, `com.apple.wifivelocityd`.

Перезагружаемся. Смотрим список запущенных сервисов:

    :::shell-session
    $ sudo launchctl list

Если в списке запущенных осталось что-то неугодное — отключаем по аналогии с сервисами из скрипта (не забывая при этом модифицировать и сам скрипт, для повторных запусков в будущем).

Стоит также иметь в виду, что при отключении сервисов, автоматически формируются списки отключенных: `/private/var/db/com.apple.xpc.launchd/disabled.plist` и `disabled.501.plist` (где `501` — UID конкретного пользователя). Это может нас выручить, если при отключении "что-то пошло не так". Вы всегда можете отредактировать соответствующие файлы, либо просто удалить целиком, перезагрузиться и начать процедуру по отключению заново.

> Как и с удалением встроенных приложений, удаление сервисов работает ровно до следующего обновления/переустановки ОС.

# 4. Отключение IPv6

`ipv6` имеет приоритет над `ipv4`, и трафик может начать в любой момент курсировать по этому протоколу. Есть ряд потенциальных проблем с этим связанных (подробнее см. в сети способ атаки на хосты с использованием ipv6), но в это мы углубляться не будем. А будем руководствоваться простым правилом безопасности: все, что нам явным образом не нужнО — должно быть отключено.

Отключение ipv6 для Ethernet-интерфейсов:

    :::shell-session
    $ sudo networksetup -setv6off Ethernet

Отключение ipv6 для Wlan-интерфейсов:

    :::shell-session
    $ sudo networksetup -setv6off Wi-Fi

# 5. Отключение atime-атрибута

Чтобы снизить количество обращений к диску, следует отключить atime-атрибут (дата-время последнего доступа к файлу) у монтируемых файловых систем.
Чтобы автоматизировать отключение atime-метки, пишем сценарий демона, который будет выполняться на этапе загрузки ОС.

`/Library/LaunchDaemons/local.noatime.plist`:

    :::xml
    <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
     <!-- Install: copy to /Library/LaunchDaemons/, chown root:wheel and chmod 644 -->
     <plist version="1.0">
       <dict>
         <key>Label</key>
         <string>local.noatime</string>
         <key>ProgramArguments</key>
         <array>
           <string>/bin/bash</string>
           <string>-c</string>
           <string>mnt=/System/Volumes/Data; mount | grep -F " $mnt "; mount -vuwo nobrowse,noatime "$mnt"</string>
         </array>
         <!-- launchd appends the output to the log file so write it to /tmp that is removed on each reboot -->
         <key>StandardOutPath</key>
         <string>/tmp/noatime.log</string>
         <key>StandardErrorPath</key>
         <string>/tmp/noatime.log</string>
         <key>RunAtLoad</key>
         <true/>
         <key>LaunchOnlyOnce</key>
         <true/>
         <key>KeepAlive</key>
         <false/>
       </dict>
     </plist>

Данный сценарий получает текущие опции монтирования разделов (в моем случае — единственного раздела), и перемонтирует его с отключением atime-атрибута. Все сообщения из потоков STD{OUT,ERR} логгируем во временный файл — `/tmp/noatime.log`.

Загружаем и выполняем сценарий как системный демон:

    :::shell-session
    $ sudo launchctl bootstrap system /Library/LaunchDaemons/local.noatime.plist

Проверяем лог смонтированных ФС (первая строчка — опции монтирования ДО, вторая — ПОСЛЕ):

    :::shell-session
    $ cat /tmp/noatime.log

    /dev/disk1s1 on /System/Volumes/Data (apfs, local, journaled, nobrowse)
    /dev/disk1s1 on /System/Volumes/Data (apfs, local, journaled, noatime, nobrowse)

Готово. С этого момента (и при последующих перезагрузках) мой единственный раздел монтируется без поддержки atime-атрибута, что (в теории) должно сказаться на общем количестве дисковых операций, и, следовательно, физическом ресурсе диска.

# 6. Итог

Беглая настройка MacOS завершена: удалены неиспользуемые встроенные приложения, отключены лишние сервисы, подкручены настройки сети и аттрибуты монтирования ФС. Система для "состоятельных американских домохозяек" стала потреблять чуть меньше ресурсов, и в целом работать чуть более предсказуемо.

Чисто утилитарно, по ресурсам, до настройки БЫЛО:

- ~4GB RAM на старте
- ~500 запущенных процессов
- ~1.4 CPU LA

После проделанных работ СТАЛО:

- 3GB RAM на старте
- ~250 процессов
- ~0.8 CPU LA

Не густо, но тем не менее. В более ранних версиях макоси была возможность отключения теней для окон (что заметно ускоряло GUI в целом). Но с какой-то версии MacOS такую возможность убрали, и тени теперь являются захардкоженной частью графической подсистемы.

Неровен час, у пользователей отберут даже такие скромные средства для кастомизации, которыми мы воспользовались в данном руководстве. Так что — настраивайте, пока еще есть такая возможность).
