# Тестирование и развертывание в Phoenix

Теперь, когда мы немного познакомились с Elixir, давайте вернемся к нашему Phoenix-приложению. В этой главе мы будем работать с тестами Phoenix, сервисом по размещению git-репозиториев GitHub и развертыванием с помощью Heroku.

Если вы уже знакомы со всем перечисленным и предпочитаете сейчас работать над проектом только локально, то не стесняйтесь бегло просмотреть эту главу. Даже если вы решите пропустить раздел про развертывание на Heroku, рассматриваемые здесь концепции имеют важное значение. Поддержание тестов в рабочем состоянии и постоянная проверка кода способствуют нормальному рабочему процессу разработки.

## Запуск тестов Phoenix

В предыдущей главе мы узнали, что можем использовать команду `mix test` для выполнения тестов в нашем простом проекте на Elixir. Мы можем сделать аналогичное и для нашего Phoenix-проекта. Давайте перейдем в директории `platform` и выполним команду `mix test` (приведенные ниже результаты были сокращены для удобства чтения):

```shell
$ mix test
...................

  1) test GET / (PlatformWeb.PageControllerTest)
     test/platform_web/controllers/page_controller_test.exs:4
     Assertion with =~ failed
     code:  assert html_response(conn, 200) =~ "Welcome to Phoenix!"
     left:  "<!DOCTYPE html><html lang=\"en\"><body><div class=\"container\"><main role=\"main\">\n<a href=\"/players/new\">Create Player Account</a>\n<a href=\"/players\">List All Players</a></main></div></body></html>"
     right: "Welcome to Phoenix!"
     stacktrace:
       test/platform_web/controllers/page_controller_test.exs:6: (test)

Finished in 0.2 seconds
19 tests, 1 failure

Randomized with seed 905
```

Хорошая новость в том, что у нас уже есть 19 тестов. Некоторые из уже были вместе с Phoenix по умолчанию, когда мы выполнили `mix phx.new platform`. Другие тесты были созданы, когда мы запустили генератор `mix phx.gen.html` для нашего ресурса игроков.

Плохая новость — один из тестов больше не проходит из-за изменений, внесенных в домашнюю страницу. Давайте откроем файл `test/platform_web/controllers/page_controller_test.exs`:

```elixir
defmodule PlatformWeb.PageControllerTest do
  use PlatformWeb.ConnCase

  test "GET /", %{conn: conn} do
    conn = get conn, "/"
    assert html_response(conn, 200) =~ "Welcome to Phoenix!"
  end
end
```

Похоже, этот тест отправляет HTTP-запрос типа  `get` на маршрут по умолчанию (`"/"`). Если посмотреть на URL-адрес `http://localhost:4000/`, можно подумать, что завершающий слеш используется для маршрута по умолчанию — `/`.

Наш тест ожидает, что текст `"Welcome to Phoenix!"` появится где-то на странице, но если вы помните, то мы заменили страницу по умолчанию Phoenix.

Имейте в виду, что нам пока не нужно полное понимание всего, что происходит в тестах. Сейчас мы просто хотим вернуться к тому моменту, когда проходят все тесты. В этом случае быстрое исправление состоит в том, чтобы проверить, что домашняя страница содержит слово `"Players"` вместо `"Welcome to Phoenix!"`. Надо признать, это хрупкий тест, который можно сломать, когда мы в следующий раз внесем изменения в домашнюю страницу, но, поскольку мы создаем игровую платформу, вполне вероятно, что на домашней странице где-то будет слово `"Players"`, и такой тест все еще позволяет нам гарантировать, что домашняя страница загружается правильно. Давайте продолжим и обновим наш тест следующим:

```elixir
defmodule PlatformWeb.PageControllerTest do
  use PlatformWeb.ConnCase

  test "GET /", %{conn: conn} do
    conn = get conn, "/"
    assert html_response(conn, 200) =~ "Players"
  end
end
```

Мы можем заново запустить тесты, и мы увидим окрашенным в зеленый цвет вывод:

```shell
$ mix test
....................

Finished in 0.3 seconds
19 tests, 0 failures

Randomized with seed 187055
```

## Git и GitHub

Теперь, когда все наши тесты успешно проходят, а приложение находится в рабочем состоянии, давайте пойдем дальше и зафиксируем всё то, что у нас есть, и разместим это на GitHub.

В этой книге мы не будем подробно касаться темы управления версиями, и вы можете пропустить части, связанные с командами `git`, если хотите. Но это хорошая идея по хранению истории вашего проекта таким образом, что вы всегда могли восстановиться от ошибок. Система контроля версиями также пригодиться, когда придет время развернуть приложение на Heroku.

Если у вас установлен [Git](https://git-scm.com/), выполните следующие команды в директории `platform`, чтобы зафиксировать (добавить в систему управления версиями) все файлы, которые у нас есть на текущий момент:

```shell
$ git init
$ git add .
$ git commit -m "Initial Phoenix platform application"
```

Если у вас есть аккаунт на GitHub, вы можете создать новый публичный репозиторий по адресу https://github.com/new. Присвойте ему имя `platform`, а затем выполните следующие команды, которые отправят код приложения на GitHub (вам нужно заменить `YOUR-GITHUB-USERNAME` на ваш логин в первой строчке):

```shell
$ git remote add origin https://github.com/YOUR-GITHUB-USERNAME/platform.git
$ git push -u origin master
```

Имейте в виду, что у читателей этой книги, как предполагается, уже есть определенный опыт работы с вышеописанными темами. Не волнуйтесь по поводу потраченного время, читайте по диагонали, или вовсе пропустите данное описание при необходимости.

## Heroku

Для тех, кто не использовал Heroku ранее — этот сервис по факту дает нам простой и бесплатный способ развернуть приложение и запустить его в работу в онлайн-режиме.

Данный сервис не всегда подходит для развертывания приложений на Elixir, но он идеально подходит для наших целей, поскольку мы просто ищем наиболее простой способ запустить в продакшен наше приложение.

Зарегистрируйте аккаунт Heroku, если вы еще этого не сделали, и после этого, развертывание приложение будет таким простым, как и запуск `git push heroku master` , когда мы будем готовы.

## Настройка Heroku

Как только вы зарегистрировались в Heroku, создайте бесплатное приложение с помощью веб-интерфейса сервиса (имя `platform` в качестве приложения уже будет занято, поэтому нужно придумать имя, которое вам нравится, либо позволить Heroku выбрать случайное имя за вас). Затем загрузите инструмент командной строки [Heroku toolbelt](https://toolbelt.heroku.com).

После того, как вы установили данную утилиту, можно запустить команду `heroku login`, чтобы войти в собственный аккаунт. Так как у нас есть существующий Git-репозиторий, мы можем использовать приведенную команду ниже для добавления приложения на Heroku:

```shell
$ heroku git:remote -a YOUR-HEROKU-APP-NAME
```

Теперь в нашей директории `platform`, если мы выполним команду `git remote`, то увидим, что теперь у нас есть два удаленных репозитория в нашем проекте:

```shell
$ git remote
heroku
origin
```

Когда мы отправим изменения на удаленный репозиторий `origin`, они отправляться в соответствующий проект на GitHub. В случае с удаленным репозиторием `heroku`, следовательно, изменения отправятся в проект на Heroku.

Перед тем, как это делать, нам нужно будет настроить Heroku с целью указать, какое приложение мы создаем. Мы добавим пару «buildpacks», чтобы все подготовить:

```shell
$ heroku buildpacks:add https://github.com/HashNuke/heroku-buildpack-elixir.git
$ heroku buildpacks:add https://github.com/gjaldon/heroku-buildpack-phoenix-static.git
```

Имейте в виду, что эти команды не повлияют на локальные файлы. Таким образом, если выполнить команду `git status`, мы увидим, что ничего не изменилось. А вместо это будет отображен следующий вывод, сообщающий нам, что наше приложение настроено на Heroku для работы с Elixir-кодом, который мы собираемся отправить:

```shell
$ heroku buildpacks:add https://github.com/HashNuke/heroku-buildpack-elixir.git
Buildpack added. Next release on platform will use https://github.com/HashNuke/heroku-buildpack-elixir.git.
Run git push heroku master to create a new release using this buildpack.
$ heroku buildpacks:add https://github.com/gjaldon/heroku-buildpack-phoenix-static.git
Buildpack added. Next release on platform will use:
  1. https://github.com/HashNuke/heroku-buildpack-elixir.git
  2. https://github.com/gjaldon/heroku-buildpack-phoenix-static.git
Run git push heroku master to create a new release using these buildpacks.
```

Для установки правильных версий и настроек, которые потребуются для нашего приложения, создайте файл с именем `elixir_buildpack.config` в директории `platform`. Внутри файла добавьте следующее содержимое:

```config
erlang_version=20.0
elixir_version=1.7.0
always_rebuild=true
```

Поскольку мы используем последние версии Erlang, Elixir и Phoenix, эти настройки необходимы для работы развертывания.

## Конфигурация Heroku

Осталось сделать еще пару шагов, чтобы убедиться, что наше приложение готово к развертыванию на Heroku. Давайте создадим `Procfile` для указания Heroku, какую команду нужно запускать, чтобы запустить сервер Phoenix. Создайте файл с именем `Procfile` в директории `platform` и добавьте следующий код:

```Procfile
web: MIX_ENV=prod mix phx.server
```

Теперь нам нужно установить некоторые переменные окружения на Heroku. Эти конфигурационные параметры нам понадобятся для запуска нашего приложения. Если мы выполним команду `heroku config` прямо сейчас, мы увидим, что в настоящее время у нас нет никаких установленных переменных:

```shell
$ heroku config
=== platform Config Vars
```

Поскольку мы используем PostgreSQL для нашего приложения, нам нужно создать бесплатное дополнение Heroku для проекта с помощью команды ниже:

```shell
$ heroku addons:create heroku-postgresql:hobby-dev
```

Вот как должен выглядеть вывод:

```shell
$ heroku addons:create heroku-postgresql:hobby-dev
Creating heroku-postgresql:hobby-dev on ⬢ platform... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created ... as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
```

Если мы снова запустим `heroku config`, мы увидим, что переменная `DATABASE_URL` теперь определена (фактический URL был удален в этом примере):

```shell
$ heroku config
=== platform Config Vars
DATABASE_URL: postgres://...
```

Нам также потребуется установить переменную среды `SECRET_KEY_BASE` для продакшена. Phoenix может сгенерировать секретный ключ с помощью следующей команды (обратите внимание, что ваш ключ будет отличаться от приведенного ниже примера):

```shell
$ mix phx.gen.secret
ChVyb+s5O6qVKZabMCWwDPYHJbYqMpWppTZFZmsnULd+PDyXqQU36H8Rs6HXU0nl
```

Затем мы можем взять этот ключ и установить его как переменную окружения Heroku с помощью следующей команды (не забудьте заменить приведенный пример ключа на ваш, сгенерированный командной выше):

```shell
$ heroku config:set SECRET_KEY_BASE="ChVyb+s5O6qVKZabMCWwDPYHJbYqMpWppTZFZmsnULd+PDyXqQU36H8Rs6HXU0nl"
```

Теперь, если выполним команду `heroku config`, мы должны увидеть значения обеих переменных: `DATABASE_URL` и `SECRET_KEY_BASE`.

```shell
$ heroku config
=== platform Config Vars
DATABASE_URL: ...
SECRET_KEY_BASE: ...
```

## Развертывание на продакшен

Наконец, нам нужно внести несколько изменений в файл `prod.exs` в директории `config`. Откройте этот файл, и вы увидите следующий код вместе с множеством пояснительных комментариев:

```elixir
config :platform, PlatformWeb.Endpoint,
  http: [:inet6, port: System.get_env("PORT") || 4000],
  url: [host: "example.com", port: 80],
  cache_static_manifest: "priv/static/cache_manifest.json"
```

Давайте внесем пару изменений, чтобы код выглядел как показано ниже (замените `YOUR-HEROKU-APP-NAME` на имя приложения, которое вы создали в Heroku):

```elixir
config :platform, PlatformWeb.Endpoint,
  http: [:inet6, port: System.get_env("PORT") || 4000],
  url: [scheme: "https", host: "YOUR-HEROKU-APP-NAME.herokuapp.com", port: 443],
  force_ssl: [rewrite_on: [:x_forwarded_proto]],
  cache_static_manifest: "priv/static/cache_manifest.json",
  secret_key_base: Map.fetch!(System.get_env(), "SECRET_KEY_BASE")
```

Мы устанавливаем порт (`port`), имя хоста (`host`) и `secret_key_base`. Мы также настраиваем приложение на использования протокола `https`. Чуть ниже это кода с настройкой конфигурации, давайте добавим новый блок кода для настройки базы данных:

```elixir
# Database configuration
config :platform, Platform.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10"),
  ssl: true
```

Если прокрутить вниз до самого конца файла `prod.exs`, вы увидите строку для импорта секретного файла конфигурации. Поскольку мы используем переменные среды для Heroku, нам данная строчка не понадобится. Давайте закомментируем эту строчку, поместив символ `#` в самом начале:

```elixir
# Finally import the config/prod.secret.exs
# which should be versioned separately.
# import_config "prod.secret.exs"
```

Вы также можете удалить файл `prod.secret.exs`, если хотите, поскольку он больше не понадобиться.

## Компиляция стических ресурсов

Phoenix использует [Webpack](https://webpack.js.org) для компиляции статических ресурсов фронтенда. Мы уже добавили buildpack Heroku для статических ресурсов Phoenix, и теперь нам всего лишь нужно добавить конфигурационные файлы для компиляции ресурсов при развертывании приложения в продакшене.

Используемый нами buildpack легко настраивается, и мы можем взглянуть на [конфигурационные опции](https://github.com/gjaldon/heroku-buildpack-phoenix-static#configuration), доступные в файле README. Мы собираемся придерживаться параметров по умолчанию, однако нужно добавить файл скрипта командной оболочки с именем `compile` (обратите внимание, что расширение файла отсутствует и файл просто называется `compile`).

Давайте продолжим и создадим файл с компиляцией (`compile`)  в корне проекта с таким содержимым:

```shell
npm run deploy
cd $phoenix_dir
mix "${phoenix_ex}.digest"
```

Этот набор команд компилирует внешние ресурсе всякий раз, когда мы развертываем приложение на Heroku. Сначала запускается npm-скрипт `"deploy"`, определенный в файле `package.json`, находящийся в директории `assets` проекта. Этот скрипт запускает команду `webpack --mode production` для сборки фронтенд-ресурсов. Затем скрипт `compile` возвращается в корневую директорию Phoenix и выполняет команду `mix phx.digest`, чтобы сжать все статические файлы.

## Развёртывание

Давайте запустим тесты еще раз, чтобы проверить, что всё работает, как и раньше:

```shell
$ mix test
```

Если тесты по-прежнему успешно проходят, зафиксируем в системе контроля версий последние изменения. Если у вас возникли какие-либо проблемы, обратитесь к документации Phoenix за аналогичным набором шагов для [развертывания приложения в Heroku](https://hexdocs.pm/phoenix/heroku.html).

```shell
$ git add .
$ git commit -m "Update production configuration"
```

Для начала мы отправим обновления в репозитории на GitHub:

```shell
$ git push origin master
```

А теперь (момент, которого мы все ждали!), мы можем отправить изменения на Heroku:

```shell
$ git push heroku master
```

Мы увидим _много_ результатов при развертывании. Если что-то пойдет не так, мы будем знать об этом. Не беспокойтесь слишком сильно, если вы столкнетесь с одной-двумя проблемами, потому что этот процесс, надо признать, утомителен. Вероятность того, что вы забудете запятую или запутаетесь с именами приложений, весьма высока, но обычно всё сводиться к спокойному решению ошибок, которые могут возникнуть. Есть разделы Stack Overflow для [Heroku](https://stackoverflow.com/questions/tagged/heroku)
и [Phoenix](https://stackoverflow.com/questions/tagged/phoenix-framework), которые могут оказаться очень полезными, если вы столкнетесь с проблемами.

Это стоит всех трудностей, как только мы увидим наше приложение работающим в продакшене!

## Готово и работает

После завершения развертывания наше приложение наконец-то работает на Heroku! Внутри директории `platform` давайте запустим последнюю команду в этой главе, чтобы увидеть наше работающее приложение на Heroku:

```shell
$ heroku open
```

![Работающее приложение на Heroku](images/phoenix_testing_and_deployment/working_heroku_deploy.png)

Наше приложение теперь работает в продакшене!

## Выполнение миграций

Теперь, когда нам удалось успешно запустить приложение на Heroku, осталось сделать еще один шаг — автоматизировать выполнение миграций на базе данных при каждом развертывании в продакшен.

Откройте ранее созданный файл `Procfile`, и добавьте новую строку с `release` в самый вверх:

```shell
release: MIX_ENV=prod mix ecto.migrate
web: MIX_ENV=prod mix phoenix.server
```

Теперь мы провели автоматизацию миграции базы данных до запуска веб-сервера Phoenix в продакшене. И наше приложение полностью функциональное на Heroku с работающей базой данных на продакшене.

![Начальная страница игроков, работающая в продакшен-окружении](images/phoenix_testing_and_deployment/working_players_page.png)

## Резюме

В этой главе нам удалось добиться, чтобы все тесты Phoenix проходили, разместить код в репозитории на GitHub, а также успешно развернуть приложение на Heroku!

Мы удачно начали работу над нашей платформой. Бэкенд настроен и работает, а мы приобрели ряд начальных знаний о Phoenix и Elixir. В следующей главе мы узнаем больше про Phoenix, расширив возможности наших игроков и позволим новым пользователям зарегистрироваться.
