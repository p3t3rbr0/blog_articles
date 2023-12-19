[META]:

```
title: Проверка орфографии
tags: recipes, emacs
meta-desc: Проверка орфографии в браузере, Libreoffice и Emacs. Настройка и использование LanguageTool.
```

---

Часто приходится набирать текст в браузере: при заполнении web-формы, google-документа, email'а и т.п. Печатаю я торопливо, и, к сожалению, заимел привычку не перечитывать написанное. Но даже перечитав текст, порой в упор можно не заметить какой-либо опечатки или ошибки, особенно под вечер, когда голова уже не соображает.

К счастью, для таких балбесов как я, есть spellchecker'ы. Я об одном из них как раз хочу рассказать. Софтина называется [LanguageTool](https://languagetool.org/) (далее просто LT). Она хорошо справляется с задачей на проверку орфографии (и не только) в браузере (и не только).

---

1. [Общие сведения](#1-общие-сведения)
2. [Запуск и проверка](#2-запуск-и-проверка)
3. [Проверка орфографии в браузере](#3-проверка-орфографии-в-браузере)
4. [Проверка орфографии в LibreOffice](#4-проверка-орфографии-в-libreoffice)
5. [Проверка орфографии в Emacs](#5-проверка-орфографии-в-emacs)
6. [Итог](#5-итог)

# 1. Общие сведения

Программа "подсвечивает" не только опечатки/ошибки, но еще и забытые/лишние скобки, криво расставленную или забытую пунктуацию, помогает подобрать синонимы. Подробнее про весь доступный функционал см. на офф. сайте.

Писана программа на Java, а сам алгоритм проверки текстов использует под капотом широко известные библиотеки: `Hunspell` и `Apache Lucene`.

LT может работать и как CLI-программа, и как клиент-серверное приложение. Нас прежде всего будет интересовать второй вариант использования.

Клиент — интегрируется в целевую программу (например, в браузер) и при заполнении форм — отправляет http-запрос на сервер, который, в свою очередь, занимается уже непосредственной проверкой присланного в запросе текса.

По умолчанию, браузерное дополнение используется сервер разрабов LT. Но юзать его небезопасно (в формах могут быть и явки/пароли, код, который вы только что написали под NDA, и т.д.) В общем, отправлять личный текст к какому-то "дядюшке на деревню" — зашквар для любого уважающего себя "ковбоя клавиатуры").

Но LT не удостоился бы чести быть описанным в данном блоге), если бы не позволял запускать свой сервер проверки орфографии, на локальном или удалённом хосте (см. [https://dev.languagetool.org/http-server](https://dev.languagetool.org/http-server)). И это ли не круто?

# 2. Запуск и проверка

Для линуксов и макоси есть готовые пакеты, которые включают в себя сценарии запуска приложения в виде системного сервиса/демона.

Но для того чтобы быстро попробовать утилиту, не привязываясь к особенностям конкретной ОС, поднимем ванильный сервер на локалхосте:

    :::shell-session
    $ wget https://languagetool.org/download/LanguageTool-stable.zip
    $ unzip LanguageTool-stable.zip && rm -f LanguageTool-stable.zip
    $ cd LanguageTool-<VERSION>/
    $ java -cp languagetool-server.jar org.languagetool.server.HTTPServer --port 8081

Проверка:

    :::shell-session
    $ curl -s -d "language=ru-RU" -d "text=текст с ашибкой" http://localhost:8081/v2/check \
        | jq ".matches[].replacements"

Результат:

    :::json
    [{"value":"Текст"}]
    [{"value":"ошибкой"},{"value":"ошибкою"},{"value":"сшибкой"},{"value":"ушивкой"},{"value":"шибкой"},{"value":"а шибкой"}]

Работает.

# 3. Проверка орфографии в браузере

Далее, качаем и устанавливаем [браузерное расширение](https://addons.mozilla.org/en-US/firefox/addon/languagetool), после чего в тулбаре появится кнопка:

![](/media/posts/languagetool/toolbar_btn.png "Контекстное меню браузерного расширения LanguageTool")

В настройках дополнения переключаем способ проверки — на собственный сервера:

![](/media/posts/languagetool/settings_window.png "Окно настроек LanguageTool")

# 4. Проверка орфографии в LibreOffice

Далее настраиваем spell checking на базе LT для [LibreOffice](https://languagetool.org/insights/post/product-libreoffice/#how-to-enable-languagetool-on-libreoffice):

![](/media/posts/languagetool/libreoffice_settings.png "Настройка spell checking через LanguageTool в LibreOffice")

и

![](/media/posts/languagetool/libreoffice_settings_aids.png "Настройка spell checking через LanguageTool в LibreOffice")

> Убедитесь, что в настройках spell checker'a у вас добавлен Русский язык как один из целевых языков при проверки орфографии.

Проверяем:

![](/media/posts/languagetool/libreoffice_usage.png "Пример проверки текста через LanguageTool в LibreOffice")

Готово!

# 5. Проверка орфографии в Emacs

Для Emacs, конечно же, тоже есть плагины:

- [https://github.com/PillFall/languagetool.el](https://github.com/PillFall/languagetool.el)
- [https://github.com/mhayashi1120/Emacs-langtool](https://github.com/mhayashi1120/Emacs-langtool)

Рассмотрим подключение и использование первого пакета — `languagetool.el`:

    :::lisp
    ;; LanguageTool
    (use-package languagetool
        :ensure t
        :defer t
        :commands (languagetool-check
                   languagetool-clear-suggestions
                   languagetool-correct-at-point
                   languagetool-correct-buffer
                   languagetool-set-language
                   languagetool-server-mode)
        :config
        (setq languagetool-java-arguments '("-Dfile.encoding=UTF-8")
              languagetool-server-url "http://localhost"
              languagetool-server-port 8081)
        :bind (("<f7>" . languagetool-server-mode)
               ("<f8>" . languagetool-correct-buffer)))

Проверяем:

![](/media/posts/languagetool/emacs.png "Пример проверки текста через LanguageTool в Emacs")

Вроде бы работает, но... У меня несколько раз Emacs зависал наглухо, при использовании LT через http. Подозреваю, что это связано с блокирующей операцией во встроенном http-клиенте Emacs. Поэтому я перешел на использование LT в Emacs через CLI-интерфейс. И это работает без проблем. Меняем код выше на:

    :::lisp
    ;; LanguageTool
    (use-package languagetool
        :ensure t
        :defer t
        :commands (languagetool-check
                   languagetool-clear-suggestions
                   languagetool-correct-at-point
                   languagetool-correct-buffer
                   languagetool-set-language)
        :config
        (setq languagetool-java-arguments '("-Dfile.encoding=UTF-8")
              languagetool-console-command "~/<path to lt>/languagetool-commandline.jar")
        :bind (("<f7>" . languagetool-check)
               ("<f8>" . languagetool-correct-buffer)))

где `<path to lt>` — полный или относительный путь к каталогу с дистрибутивом LT.

# 6. Итог

LanguageTool — годный и шустрый инструмент для проверки орфографии (и не только) в браузере (и не только). Opensource, self-hosted, — все как мы любим. Энджой, мазаХака!
