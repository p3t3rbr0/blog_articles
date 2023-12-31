[META]:

```
title: OpenBSD на 4 года
tags: openbsd
meta-desc: Запуск эксперимента по содержанию OpenBSD на VPS в течение четырех лет.
```

---

Мне нравится [OpenBSD](https://www.openbsd.org/). Есть [много](https://why-openbsd.rocks/fact/) годных вещей, реализованных в рамках и вокруг проекта OpenBSD, которые выглядят достаточно "вкусными", чтобы их захотелось попробовать.

На практике ОС работает стабильно, в настройке относительно проста, синтаксис некоторых конфигов схож и даже как-то лаконичен.

В сети можно найти много статей с резонами попробовать OpenBSD, останавливаться на них я не буду, пытливый читатель без труда найдет инфу об этом. Я лишь хочу высказаться по поводу своего выбора серверной ОС на ближайшие 4 года.

---

Итак, требования: заиметь серверную ОС под личные проекты, которая бы работала стабильно, предсказуемо, была проста в использовании, в достаточной степени безопасна, и не требовала частых вмешательств с точки зрения администрирования.

До ноября месяца вся инфра на моей VPS работала под управлением OpenBSD 7.0. Работала чуть более двух лет. Два года предельно стабильной и предсказуемой работы. Это ли не круто?

Вот только возможностей старых версий софта перестало хватать. Плюс поднакопился мусор в конфигах и ФС. Поэтому решил установить новую версию ОС на новую VPS. А заодно с этим — стартануть эксперимент по содержанию OpenBSD, в одной версии, на протяжении 4 лет (т.е. в два раза больше, чем предыдущий период).

Кратко про резоны:

1) Админство сервера (пусть и "игрушечного") - требует временнЫх и когнитивных усилий. Релизы ОС выходят раз в полгода, каждый раз апргрейдиться, попутно разруливая всплывшие косяки то там, то здесь — не охота.

Поставить, настроить и надолго забыть — вот чего хочется. Хочется использовать время на то, чтобы СОЗДАВАТЬ (говношлепать), а не НАСТРАИВАТЬ (уже кем-то наговношлепанное).

2) При обновлении раз в N лет, где N > 1, — можно более отчетливо наблюдать тенденции в развитии того или иного ПО, а также в железе. Т.е. интересно, порой, обернуться назад, чтоб вспомнить как тогда было, и как есть сейчас.

К примеру. Арендуя первую VPS как раз где-то 4-5 лет назад (несколько лет я юзал Debian, прежде чем поменять его на OpenBSD), SSD диски были еще дорогими. Сейчас же уже просто не найти VPS с HDD, везде только опция SSD || NVMe, и цены на диски заметно снизились. Изменилась и линейка серверных CPU. Появились более производительные AMD EPYC и новые линейки Xeon'ов. Буквально намедни, доплатив всего 13 рублей свыше (относительно стоимости старой VPS), я арендовал новую с SSD-диском (на тот же объем) и двухъядерным AMD EPYC CPU. При прочих равных, это явный буст в производительности вычислительных операций и операций ввода-вывода.

ПО же, в плане изменений, вообще дико рвет с места в карьер. Например, в новой версии OpenBSD, Full Drive Encryption (FDE) делается одним нажатием на Enter (ранее надо было создавать два блочных устройства, натягивать на них softraid0, шаманить с bioctl и т.д.). Сейчас же это все спрятано под капотом единственного пункта в инсталляторе, буквально `"Хотите FDE" (yes/no)? `.

Появились новые возможности в средствах виртуализации (мной еще НЕ опробовано, но тенденции хорошие — так, глядишь, можно будет юзать опенка и на рабочей машине, для докеров в том числе). Появились новые механизмы защиты и вообще — куча всего: [7.1](https://www.openbsd.org/plus71.html), [7.2](https://www.openbsd.org/plus72.html), [7.3](https://www.openbsd.org/plus73.html), [7.4](https://www.openbsd.org/plus74.html).

В пользовательском софте, разумеется, тоже много изменений. Самое главное — новые версии интерпретаторов и компиляторов ЯП.

И все это за ~два года. Страшно представить, что там накопится за четыре года).

3) Обновления с чистого листа (форматирование раздела, установка новой ОС и нового ПО) — не оставляет за собой мусора в виде устаревших *.old-конфигов, косяков, возникающих при обновлении пользовательского ПО, при накатывании миграций баз данных, и т.п.

Так как VPS личная, то мы можем себе позволить снести все к херам собачим, и установить/настроить заново. Проделывать такое раз в полгода я не готов, а вот раз в четыре — вполне можно).

---

Однако, у всего этого мероприятия есть один минус — патчи на подсистемы ОС выходят всего год после релиза. Дальше данная версия считается неподдерживаемой. Это значит, что последующие три года (из моего эксперимента), ОС и софт на VPS будут находиться в неактуальном состоянии. Отсюда и потенциальная подверженность уязвимостям, и отсутствие новых возможностей в пользовательском софте.

Но... если взглянуть под другим углом: слегка устаревший софт это не так уж и плохо, если он работает стабильно и выполняет требуемые от него задачи. Уязвимости же не сильно страшны, ибо это OpenBSD).

Что я имею в виду? Ну, во-первых, любой удаленный взлом (получение прав пользователя на сервере или выполнение кода от имени пользователя) — этого надо еще добиться). Жопой в тырнет торчит не так уж и много сервисов, все они OpenBSD'ишные (т.е. не очень популярные, более простые чем аналоги, имеют встроенные средства защиты от типовых косяков, и посему чуть более надежны, чем аналоги).

Чисто в теории, МАКСИМУМ на что может рассчитывать потенциальный хакер — выполннение кода от имени сильного урезанного в правах пользователя, от которого запущен тот или иной сетевой сервис (коих, повторюсь, совсем не много).

Ну и самая главная защита) — ломать мой игрушечный сервачек не выгодно чисто с экономической точки зрения: затраты на взлом будут большие, а профита это мероприятие не принесет никакого. Поэтому "дЫры в безопасности" [Хорошо, что хоть что-то у нас в безопасности (c)] — нас не сильно колышут.

Рассуждая так, прихожу к выводу, что продержать 4 года опенка на серваке — вполне норм затея.

Данным постом, задаю точку отсчета четырехлетнего эксперимента. По прошествии этого времени интересно будет провести ретроспективу, поделиться наблюдениями. По моим подсчетам, на дворе тогда будет осень 2027 года и релиз под номером 8.2.

А пока — счастливо, и до связи!
