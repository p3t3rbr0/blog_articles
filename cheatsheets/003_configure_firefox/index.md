[META]:

```
title: Настройка Firefox
revision: 1
tags: recipes, firefox
meta-desc: Шпаргалка по настройке Firefox. Общие рекомендации, отключение телеметрии, улучшение производительности.
```

---

На данный момент Firefox является единственным вменяемым браузером, в котором возможно произвести расширенную настройку обычному пользователю. Например, отключить телеметрию, изменить размер буферов, выставить/увеличить лимиты на сетевые соединения, определить количество процессов для парсинга DOM и т.д.

Такой потенциал, вкупе с ESR-веткой и блокировщиком рекламы, позволяют ожидаемо стабильно пользоваться благами WWW на протяжении длительного периода, экономя время и нервы.

Данная шпаргалка позволяет быстро настроить браузер для адекватного использования на любой поддерживаемой FF платформе, и актуальна для версий 115-118 включительно.

---

1. [Общие рекомендации](#1-общие-рекомендации)
2. [Базовая настройка](#2-базовая-настройка)
3. [Дополнительная настройка](#3-дополнительная-настройка)
4. [Отключение телеметрии](#4-отключение-телеметрии)
5. [Используемые источники](#5-используемые-источники)

# 1. Общие рекомендации.

Первым делом определим общие положения относительно того, как добиться приемлемой работы браузера Firefox.

1) Качаем с офф. сайта последнюю ESR-версию.

ESR-версия поддерживается год, ограждена от добавления новых возможностей, и включает в себя только багфиксы.

2) Выпиливаем все лишнее из тулбара, чтобы остались только кнопки "туда-сюда-релоад", address bar, кнопка меню дополнений и кнопка меню браузера.

3) Устанавливаем блокировщик рекламы [ublock-origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/).

4) [Опционально] устанавливаем плагин для проверки орфографии через внешний сервер - [languagetool](https://addons.mozilla.org/en-US/firefox/addon/languagetool/).

О настройке последнего я писал в заметке "[Проверка орфографии](https://peterbro.su/posts/languagetool)".

5) Настраиваем дополнения: отключаем автоапдейт, отключаем лишние фильтры, подсказки и т.п.

6) После проделанных выше манипуляций, переходим к настройке самого браузера: сначала через интерфейс `Settings`, а затем через интерфейс `about:config`.

На последних двух манипуляциях остановимся подробно.

# 2. Базовая настройка.

Вносим необходимые изменения через интерфейс `Settings`. Ниже рассматриваются основные из них, различная вкусовщина опущена.

## General

    :::
    Language
        > Use your operating system settings for “English (GB)” to format dates, times, numbers, and measurements.
            -> [ON]
        > Check your spelling as you type
            -> [OFF]
    Files and Applications
        > Downloads
            > Always ask you where to save files
                -> [ON]
        > Applications
            > Ask whether to open or save files
                -> [ON]
    Firefox Updates
        > Check for updates but let you choose to install them
            -> [ON]

## Privacy and Security

    :::
    Browser Privacy
        > Enhanced Tracking Protection
            > Strict
                -> [ON]
    Firefox Data Collection and Use
        > Allow Firefox to send technical and interaction data to Mozilla
            -> [OFF]
        > Allow Firefox to install and run studies
            -> [OFF]
        > Allow Firefox to make personalized extension recommendations
            -> [OFF]
        > View Firefox studies
            -> [OFF]
        > Allow Firefox to send backlogged crash reports on your behalf
            -> [OFF]
    > Security
        > Deceptive Content and Dangerous Software Protection
            > Block dangerous and deceptive content
                -> [ON]
            > Block dangerous downloads
                -> [ON]
            > Warn you about unwanted and uncommon software
                -> [ON]
        > Certificates
            > Query OCSP responder servers to confirm the current validity of certificates
                -> [ON]
    > HTTPS-Only Mode
        > Don’t enable HTTPS-Only Mode
            -> [ON]
    > DNS over HTTPS
        > Enable secure DNS using
            > Off
                -> [ON]

## Performance

    :::
    Use recommended performance settings
        -> [OFF]
    Use hardware acceleration when available
        -> [OFF]

# 3. Дополнительная настройка.

Для дальнейше настройки потребуется зайти по адресу `about:config`, поставить галочку и нажать на кнопочку).
Каждая настройка — есть ничто иное, как определение нужного ЗНАЧЕНИЯ для целевого КЛЮЧА. Кратко про затрагиваемые ключи:

- **reader.parse-on-load.enabled**

Отключаем дополнительный парсинг страницы для "reading mode".

- **reader.toolbar.vertical**

Отключаем отображение иконки "reading mode" из адресной строки.

- **extensions.pocket.enabled**

Отключаем встроенное дополнение "Pocket".

- **security.dialog_enable_delay**

Отключаем задержку (она связана с безопасностью, но будьте покойны оно вам не пригодится) перед показом диалоговых окон браузером (в том числе и диалога скачивания файлов).

- **dom.ipc.processCount**

Задаем нужное количество процессов для парсинга DOM. Хорошей идеей будет задать число, равное числу ядер (в том числе и виртуальных) процессора.

- **full-screen-api.warning.timeout**

Отключаем предупреждение о видео, отображаемом в полноэкранном режиме (чем меньше "всплывашек", тем лучше)

- **gfx.webrender.software**

Включаем или отключаем рендеринг содержимого через CPU. Лучше явным образом задать нужное значение. Если у вас дискретная видеокарта и настроено аппаратное ускорение, то опцию следует ОТКЛючить, дабы рендер осуществлялся на GPU.

- **browser.cache.memory.capacity**

Определяем объем кеша. На [wiki.archlinux](#5-используемые-источники) предлагают воспользоваться формулой:

    :::
    (41297 - (41606 / (1 + ((<RAM> / 1.16) ** 0.75))))

где `<RAM>` - объем (в гигабайтах) оперативной памяти на устройстве. К примеру, для 16GB RAM, получается:

    :::
    (41297 - (41606 / (1 + ((16 / 1.16) ** 0.75))))
    36196.5

Доверимся предположению, что данная магия улучшит отзывчивость или уменьшит отжираемую браузером память).

- **browser.sessionstore.resume_from_crash**

Отключаем возможность восстановления вкладок после аварийного завершения. Не вижу ценности в этом для себя, ибо работаю с ноута (что исключает сценарии внезапного шатдауна), а браузер сам по себе вроде как не должен падать).

Если для вас эта фича нужна/важна, то можете не отключать, но хотя бы увеличить интервал записи сессии на диск (тем самым сэкономив Disk IOPS) - `browser.sessionstore.interval`.

- **browser.preferences.defaultPerformanceSettings.enabled**

Отключаем стандартные настройки производительности (так как мы их зададим вручную).

Итоговая таблица с настройками:

| Название свойства                                      | Значение/Действие |
|--------------------------------------------------------|-------------------|
| reader.parse-on-load.enabled                           | false             |
| reader.toolbar.vertical                                | false             |
| extensions.pocket.enabled                              | false             |
| security.dialog_enable_delay                           | 0                 |
| dom.ipc.processCount                                   | 4                 |
| full-screen-api.warning.timeout                        | 0                 |
| gfx.webrender.software                                 | false             |
| browser.cache.memory.capacity                          | 36196.5           |
| browser.sessionstore.resume_from_crash                 | false             |
| browser.preferences.defaultPerformanceSettings.enabled | false             |

# 4. Отключение телеметрии.

Ну тут комментарии излишни - отрубаем все с тегами `%telemetry%` и `%report%`:

| Название свойства                                  | Значение/Действие |
|----------------------------------------------------|-------------------|
| browser.newtabpage.activity-stream.feeds.telemetry | false             |
| browser.newtabpage.activity-stream.telemetry       | false             |
| browser.ping-centre.telemetry                      | false             |
| datareporting.healthreport.service.enabled         | false             |
| datareporting.healthreport.uploadEnabled           | false             |
| datareporting.policy.dataSubmissionEnabled         | false             |
| datareporting.sessions.current.clean               | true              |
| devtools.onboarding.telemetry.logged               | false             |
| toolkit.telemetry.archive.enabled                  | false             |
| toolkit.telemetry.bhrPing.enabled                  | false             |
| toolkit.telemetry.enabled                          | false             |
| toolkit.telemetry.firstShutdownPing.enabled        | false             |
| toolkit.telemetry.hybridContent.enabled            | false             |
| toolkit.telemetry.newProfilePing.enabled           | false             |
| toolkit.telemetry.prompted                         | 2                 |
| toolkit.telemetry.rejected                         | true              |
| toolkit.telemetry.reportingpolicy.firstRun         | false             |
| toolkit.telemetry.server                           | Delete URL        |
| toolkit.telemetry.server_owner                     | Delete Value      |
| toolkit.telemetry.shutdownPingSender.enabled       | false             |
| toolkit.telemetry.unified                          | false             |
| toolkit.telemetry.unifiedIsOptIn                   | false             |
| toolkit.telemetry.updatePing.enabled               | false             |

# 5. Используемые источники.

1. <https://wiki.archlinux.org/title/Firefox/Tweaks>
2. <https://github.com/K3V1991/Disable-Firefox-Telemetry-and-Data-Collection>
