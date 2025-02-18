Конфигурирование приложений на Rails
====================================

Это руководство раскрывает особенности конфигурирования и инициализации, доступные приложениям на Rails.

После прочтения этого руководства, вы узнаете:

* Как конфигурировать поведение ваших приложений на Rails.
* Как добавить дополнительный код, запускаемый при старте приложения.

Расположение инициализационного кода
------------------------------------

Rails предлагает четыре стандартных места для размещения инициализационного кода:

* config/application.rb
* Конфигурационные файлы конкретных сред
* Инициализаторы
* Пост-инициализаторы

Запуск кода до Rails
--------------------

В тех редких случаях, когда вашему приложению необходимо запустить некоторый код до того, как сам Rails загрузится, поместите его до вызова `require 'rails/all'` в `config/application.rb`.

Конфигурирование компонентов Rails
----------------------------------

В целом, работа по конфигурированию Rails означает как настройку компонентов Rails, так и настройку самого Rails. Конфигурационный файл `config/application.rb` и конфигурационные файлы конкретных сред (такие как `config/environments/production.rb`) позволяют определить различные настройки, которые можно придать всем компонентам.

Например, можно добавить эту настройку в файл `config/application.rb`:

```ruby
config.time_zone = 'Central Time (US & Canada)'
```

Это настройка для самого Rails. Если хотите передать настройки для отдельных компонентов Rails, это также осуществляется через объект `config` в `config/application.rb`:

```ruby
config.active_record.schema_format = :ruby
```

Rails будет использовать эту конкретную настройку для конфигурирования Active Record.

### (rails-general-configuration) Общие настройки Rails

Эти конфигурационные методы вызываются на объекте `Rails::Railtie`, таком как подкласс `Rails::Engine` или `Rails::Application`.

* `config.after_initialize` принимает блок, который будет запущен _после того_, как Rails закончит инициализацию приложения. Это включает инициализацию самого фреймворка, engine-ов и всех инициализаторов приложения из `config/initializers`. Отметьте, что этот блок _будет_ запущен для Rake задач. Полезно для конфигурирования настроек, установленных другими инициализаторами:

    ```ruby
    config.after_initialize do
      ActionView::Base.sanitized_allowed_tags.delete 'div'
    end
    ```

* `config.asset_host` устанавливает хост для ассетов. Полезна, когда для хостинга ассетов используются CDN, или когда необходимо обойти встроенные в браузеры конкурентные ограничения, используя различные псевдонимы доменов. Укороченная версия `config.action_controller.asset_host`.

* `config.autoload_once_paths` принимает массив путей, по которым Rails будет загружать константы, не стирающиеся между запросами. Уместна, если `config.cache_classes` является `false`, что является в режиме development по умолчанию. В противном случае все автозагрузки происходят только раз. Все элементы этого массива также должны быть в `autoload_paths`. По умолчанию пустой массив.

* `config.autoload_paths` принимает массив путей, по которым Rails будет автоматически загружать константы.По умолчанию все директории в `app`. Больше не рекомендуется настраивать это. Подробнее смотрите в руководстве [Автозагрузка и перезагрузка констант](/constant_autoloading_and_reloading#autoload-paths-and-eager-load-paths)

* `config.cache_classes` контролирует, будут ли классы и модули приложения перезагружены при каждом запросе. По умолчанию `false` в режиме development и true в режимах test и production.

* `config.beginning_of_week` устанавливает начало недели по умолчанию для приложения. Принимает символ валидного дня недели (например, `:monday`).

* `config.cache_store` конфигурирует, какое хранилище кэша использовать для кэширования Rails. Опции включают один из символов `:memory_store`, `:file_store`, `:mem_cache_store`, `:null_store`, `:redis_cache_store` или объект, реализующий API кэша. По умолчанию `:file_store`.

* `config.colorize_logging` определяет, использовать ли коды цвета ANSI при логировании информации. По умолчанию `true`.

* `config.consider_all_requests_local` это флажок. Если `true`, тогда любая ошибка вызовет детальную отладочную информацию, которая будет выгружена в отклик HTTP, и контроллер `Rails::Info` покажет контекст выполнения приложения в `/rails/info/properties`. По умолчанию `true` в режимах development и test, и `false` в режиме production. Для более детального контроля, установите ее в `false` и примените `local_request?` в контроллерах для определения, какие запросы должны предоставлять отладочную информацию при ошибках.

* `config.console` позволит установить класс, который будет использован как консоль при вызове `rails console`. Лучше всего запускать его в блоке `console`:

    ```ruby
    console do
      # этот блок вызывается только при запуске консоли,
      # поэтому можно безопасно поместить тут pry
      require "pry"
      config.console = Pry
    end
    ```

* `config.eager_load` когда `true`, лениво загружает все зарегистрированные `config.eager_load_namespaces`. Они включают ваше приложение, engine-ы, фреймворки Rails и любые другие зарегистрированные пространства имен.

* `config.eager_load_namespaces` регистрирует пространства имен, которые лениво загружаются, когда `config.eager_load` равно `true`. Все пространства имен в этом списке должны отвечать на метод `eager_load!`.

* `config.eager_load_paths` принимает массив путей, из которых Rails будет нетерпеливо загружать при загрузке, если включено кэширование классов. По умолчанию каждая папка в директории `app` приложения.

* `config.enable_dependency_loading`: когда true, включает автозагрузку, даже если приложение нетерпеливо загружено и `config.cache_classes` установлена как true. По умолчанию false.

* `config.encoding` настраивает кодировку приложения. По умолчанию UTF-8.

* `config.exceptions_app` устанавливает приложение по обработке исключений, вызываемое промежуточной программой ShowException, когда происходит исключение. По умолчанию `ActionDispatch::PublicExceptions.new(Rails.public_path)`.

* `config.debug_exception_response_format` устанавливает формат, используемый в откликах, когда возникают ошибки в режиме development. По умолчанию `:api` для только API приложений и `:default` для нормальных приложений.

* `config.file_watcher` это класс, используемый для обнаружения обновлений файлов в файловой системе, когда `config.reload_classes_only_on_change` равно `true`. Rails поставляется с `ActiveSupport::FileUpdateChecker` (по умолчанию) и `ActiveSupport::EventedFileUpdateChecker` (этот зависит от гема [listen](https://github.com/guard/listen)). Пользовательские классы должны соответствовать `ActiveSupport::FileUpdateChecker` API.

* `config.filter_parameters` используется для фильтрации параметров, которые не должны быть показаны в логах, такие как пароли или номера кредитных карт. Он также фильтрует чувствительные параметры в столбцах базы данных при вызове `#inspect` на объектах Active Record. По умолчанию Rails фильтрует пароли, добавляя `Rails.application.config.filter_parameters += [:password]` в `config/initializers/filter_parameter_logging.rb`. Фильтр параметров работает как частично соответствующее регулярное выражение.

* `config.force_ssl` принуждает все запросы обслуживаться протоколом HTTPS, используя промежуточную программу `ActionDispatch::SSL`, и устанавливает `config.action_mailer.default_url_options` равным `{ protocol: 'https' }`. Это может быть настроено, установив `config.ssl_options` - подробнее смотрите в [документации ActionDispatch::SSL](http://api.rubyonrails.org/classes/ActionDispatch/SSL.html).

* `config.log_formatter` определяет форматер для логгера Rails. Эта опция по умолчанию равна экземпляру `ActiveSupport::Logger::SimpleFormatter` для всех режимов. Если установите значение для `config.logger`, вы должны вручную передать значение вашего форматера для вашего логгера до того, как он будет обернут в экземпляр `ActiveSupport::TaggedLogging`, Rails не сделает это за вас.

* `config.log_level` определяет многословность логгера Rails. Эта опция по умолчанию `:debug` для всех сред. Доступные уровни лога: `:debug`, `:info`, `:warn`, `:error`, `:fatal`, and `:unknown`.

* `config.log_tags` принимает список: методов, на которые отвечает объект `request`, объект `Proc`, который принимает `request` объект, или что-то, отвечающее на `to_s`. С помощью этого становится просто тегировать строчки лога отладочной информацией, такой как поддомен и id запроса - очень полезно для отладки многопользовательского приложения.

* `config.logger` это логгер, который будет использован для `Rails.logger` и любого логирования, относящегося к Rails, такого как `ActiveRecord::Base.logger`. По умолчанию это экземпляр `ActiveSupport::TaggedLogging`, оборачивающий экземпляр `ActiveSupport::Logger`, который пишет лог в директорию `log/`. Можно предоставить произвольный логгер, чтобы получить полную совместимость, нужно следовать следующим рекомендациям:

  * Чтобы поддерживался форматер, необходимо в логгере вручную назначить форматер из значения `config.log_formatter`.
  * Чтобы поддерживались тегированные логи, экземпляр лога должен быть обернут в `ActiveSupport::TaggedLogging`.
  * Чтобы поддерживалось глушение, логгер должен включать модуль `ActiveSupport::LoggerSilence`. Класс `ActiveSupport::Logger` уже включает эти модули.

    ```ruby
    class MyLogger < ::Logger
      include ActiveSupport::LoggerSilence
    end

    mylogger           = MyLogger.new(STDOUT)
    mylogger.formatter = config.log_formatter
    config.logger      = ActiveSupport::TaggedLogging.new(mylogger)
    ```

* `config.middleware` позволяет настроить промежуточные программы приложения. Это подробнее раскрывается в разделе [Конфигурирование промежуточных программ](#configuring-middleware) ниже.

* `config.reload_classes_only_on_change` включает или отключает перезагрузку классов только при изменении отслеживаемых файлов. По умолчанию отслеживает все по путям автозагрузки и установлена `true`. Если `config.cache_classes` установлена `true`, эта опция игнорируется.

* `secret_key_base` используется для определения ключа, позволяющего сессиям приложения быть верифицированными по известному ключу безопасности, чтобы избежать подделки. Приложения получают случайно сгенерированный ключ в test и development средах, другие среды должны устанавливать это в `config/credentials.yml.enc`.

* `config.public_file_server.enabled` конфигурирует Rails на обслуживание статичных файлов из директории public. Эта опция по умолчанию `true`, но в среде production устанавливается `false`, так как серверные программы (например, NGINX или Apache), используемые для запуска приложения, должны обслуживать статичные ресурсы вместо Rails. Если запускаете или тестируете приложение в среде production с помощью WEBrick (не рекомендуется использовать WEBrick в production), установите эту опцию в `true`. В противном случае нельзя воспользоваться кэшированием страниц и запросами файлов, существующих в директории public.

* `config.session_store` определяет, какой класс использовать для хранения сессии. Возможные значения `:cookie_store`, которое по умолчанию, `:mem_cache_store` и `:disabled`. Последнее говорит Rails не связываться с сессиями. По умолчанию равно хранилищу куки с именем приложения в качестве ключа сессии. Произвольные хранилища сессии также могут быть определены:

    ```ruby
    config.session_store :my_custom_store
    ```

    Это произвольное хранилище должно быть определено как `ActionDispatch::Session::MyCustomStore`.

* `config.time_zone` устанавливает временную зону по умолчанию для приложения и включает понимание временных зон для Active Record.

### Настройка ассетов

* `config.assets.enabled` это флажок, контролирующий, будет ли включен файлопровод (asset pipeline). По умолчанию он устанавливается `true`.

* `config.assets.css_compressor` определяет используемый компрессор CSS. По умолчанию установлен `sass-rails`. Единственное альтернативное значение в настоящий момент это `:yui`, использующее гем `yui-compressor`.

* `config.assets.js_compressor` определяет используемый компрессор JavaScript. Возможные варианты `:closure`, `:uglifier` и `:yui` требуют использование гемов `closure-compiler`, `uglifier` или `yui-compressor` соответственно.

* `config.assets.gzip` флажок, включающий создание сжатых версий скомпилированных ассетов вместе с несжатыми ассетами. По умолчанию установлено `true`.

* `config.assets.paths` содержит пути, используемые для поиска ассетов. Присоединение путей к этой конфигурационной опции приведет к тому, что эти пути будут использованы в поиске ассетов.

* `config.assets.precompile` позволяет определить дополнительные ассеты (иные, чем `application.css` и `application.js`), которые будут предварительно компилированы при запуске `rake assets:precompile`.

* `config.assets.unknown_asset_fallback` позволяет модифицировать поведение файлопровода, когда ассет не в нем, если вы используете sprockets-rails 3.2.0 или новее. По умолчанию `false`.

* `config.assets.prefix` определяет префикс из которого будут обслуживаться ассеты. По умолчанию `/assets`.

* `config.assets.manifest` определяет полный путь для использования файлом манифеста прекомпилятора ассетов. По умолчанию файл называется `manifest-<random>.json` в директории `config.assets.prefix` в папке public.

* `config.assets.digest` включает использование меток SHA256 в именах ассетов. Установлено по умолчанию `true`.

* `config.assets.debug` отключает объединение и сжатие ассетов. Установлено по умолчанию `true` в `development.rb`.

* `config.assets.version` опция, используемая в генерации хэша SHA256. Ее можно использовать чтобы принудительно перекомпилировать все файлы.

* `config.assets.compile` - булево значение, используемое для включения компиляции Sprockets на лету в production.

* `config.assets.logger` принимает логгер, соответствующий интерфейсу Log4r, или дефолтный Ruby класс `Logger`. По умолчанию такой же, как указан в `config.logger`. Установка `config.assets.logger` в `false` отключает логирование сжатых ассетов.

* `config.assets.quiet` отключает логирование запросов к ассетам. Установлено `true` по умолчанию в `development.rb`.

### Конфигурирование генераторов

Rails позволяет изменить, какие генераторы следует использовать, с помощью метода `config.generators`. Этот метод принимает блок:

```ruby
config.generators do |g|
  g.orm :active_record
  g.test_framework :test_unit
end
```

Полный перечень методов, которые можно использовать в этом блоке, следующий:

* `assets` позволяет создавать ассеты при генерации скаффолда. По умолчанию `true`.

* `force_plural` позволяет имена моделей во множественном числе. По умолчанию `false`.

* `helper` определяет, генерировать ли хелперы. По умолчанию `true`.

* `integration_tool` определяет интеграционный инструмент, используемый для генерации интеграционных тестов. По умолчанию `:test_unit`.

* `system_tests` определяет интеграционный инструмент, используемый для генерации системных тестов. По умолчанию `:test_unit`.

* `orm` определяет используемую orm. По умолчанию `false` и используется Active Record.

* `resource_controller` определяет используемый генератор для генерация контроллера при использовании `rails generate resource`. По умолчанию `:controller`.

* `resource_route` определяет нужно ли генерировать определение ресурсного маршрута или нет. По умолчанию `true`.

* `scaffold_controller`, отличающийся от `resource_controller`, определяет используемый генератор для генерации контроллера _скаффолда_ при использовании `rails generate scaffold`. По умолчанию `:scaffold_controller`.

* `stylesheets` включает в генераторах хук для таблиц стилей. Используется в Rails при запуске генератора `scaffold` , но этот хук также может использоваться в других генераторах. По умолчанию `true`.

* `stylesheet_engine` конфигурирует используемый при генерации ассетов движок CSS (например, sass). По умолчанию `:css`.

* `scaffold_stylesheet` создает `scaffold.css` при генерации ресурса скаффолда. По умолчанию `true`.

* `test_framework` определяет используемый тестовый фреймворк. По умолчанию `false`, и используется minitest.

* `template_engine` определяет используемый движок шаблонов, такой как ERB или Haml. По умолчанию `:erb`.

### (configuring-middleware) Конфигурирование промежуточных программ (middleware)

Каждое приложение Rails имеет стандартный набор промежуточных программ, используемых в следующем порядке в среде development:

* `ActionDispatch::SSL` принуждает каждый запрос быть обслуженным с помощью HTTPS. Включен, если `config.force_ssl` установлена `true`. Передаваемые сюда опции могут быть настроены с помощью `config.ssl_options`.

* `ActionDispatch::Static` используется для обслуживания статичных ассетов. Отключено, если `config.public_file_server.enabled` равна `false`. Установите `config.public_file_server.index_name` если вам нужно обслуживать индексный файл статичной директории, который называется не `index`. Например, для обслуживания `main.html` вместо `index.html` для запросов, установите `config.public_file_server.index_name` в `"main"`.

* `ActionDispatch::Executor` позволяет тредобезопасную перезагрузку кода. Отключено, если `config.allow_concurrency` установлена `false`, что загружает `Rack::Lock`. `Rack::Lock` оборачивает приложение в мьютекс, таким образом оно может быть вызвано только в одном треде одновременно.

* `ActiveSupport::Cache::Strategy::LocalCache` служит простым кэшем в памяти. Этот кэш не является тредобезопасным и предназначен только как временное хранилище кэша для отдельного треда.

* `Rack::Runtime` устанавливает заголовок `X-Runtime`, содержащий время (в секундах), затраченное на выполнение запроса.

* `Rails::Rack::Logger` пишет в лог, что начался запрос. После выполнения запроса сбрасывает логи.

* `ActionDispatch::ShowExceptions` ловит исключения, возвращаемые приложением, и рендерит прекрасные страницы исключения, если запрос локальный или если `config.consider_all_requests_local` установлена `true`. Если `config.action_dispatch.show_exceptions` установлена `false`, исключения будут вызваны несмотря ни на что.

* `ActionDispatch::RequestId` создает уникальный заголовок X-Request-Id, доступный для отклика, и включает метод `ActionDispatch::Request#uuid`.

* `ActionDispatch::RemoteIp` проверяет на атаки с ложных IP и получает валидный `client_ip` из заголовков запроса. Конфигурируется с помощью опций `config.action_dispatch.ip_spoofing_check` и `config.action_dispatch.trusted_proxies`.

* `Rack::Sendfile` перехватывает отклики, чьи тела были обслужены файлом, и заменяет их специфичным для сервером заголовком X-Sendfile. Конфигурируется с помощью `config.action_dispatch.x_sendfile_header`.

* `ActionDispatch::Callbacks` запускает подготовленные колбэки до обслуживания запроса.

* `ActionDispatch::Cookies` устанавливает куки для каждого запроса.

* `ActionDispatch::Session::CookieStore` ответственно за хранение сессии в куки. Для этого может использоваться альтернативная промежуточная программа, при изменении `config.action_controller.session_store` на альтернативное значение. Кроме того, переданные туда опции могут быть сконфигурированы `config.action_controller.session_options`.

* `ActionDispatch::Flash` настраивает ключи `flash`. Доступно, только если у `config.action_controller.session_store` установлено значение.

* `Rack::MethodOverride` позволяет методу быть переопределенным, если установлен `params[:_method]`. Это промежуточная программа, поддерживающая типы методов HTTP PATCH, PUT и DELETE.

* `Rack::Head` преобразует запросы HEAD в запросы GET и обслуживает их соответствующим образом.

Кроме этих полезных промежуточных программ можно добавить свои, используя метод `config.middleware.use`:

```ruby
config.middleware.use Magical::Unicorns
```

Это поместит промежуточную программу `Magical::Unicorns` в конец стека. Можно использовать `insert_before`, если желаете добавить промежуточную программу перед другой.

```ruby
config.middleware.insert_before Rack::Head, Magical::Unicorns
```

Или можно вставить промежуточную программу на конкретное место с помощью индексов. Например, если хотите вставить промежуточную программу `Magical::Unicorns` наверх стека, это можно сделать так:

```ruby
config.middleware.insert_before 0, Magical::Unicorns
```

Также есть `insert_after`, который вставляет промежуточную программу после другой:

```ruby
config.middleware.insert_after Rack::Head, Magical::Unicorns
```

Промежуточные программы также могут быть полностью переставлены и заменены другими:

```ruby
config.middleware.swap ActionController::Failsafe, Lifo::Failsafe
```

Они также могут быть убраны из стека полностью:

```ruby
config.middleware.delete Rack::MethodOverride
```

### Конфигурирование i18n

Все эти конфигурационные опции делегируются в библиотеку `I18n`.

* `config.i18n.available_locales` определяет разрешенные доступные локали приложения. По умолчанию все ключи локалей, обнаруженные в файлах локалей, обычно только `:en` для нового приложения.

* `config.i18n.default_locale` устанавливает локаль по умолчанию для приложения, используемого для интернационализации. По умолчанию `:en`.

* `config.i18n.enforce_available_locales` обеспечивает, что все локали, переданные из i18n, должны быть объявлены в списке `available_locales`, вызывая исключение `I18n::InvalidLocale` при установке недоступной локали. По умолчанию `true`. Рекомендуется не отключать эту опцию, если этого не сильно требуется, так как она работает в качестве меры безопасности от установки неверной локали на основе пользовательских данных.

* `config.i18n.load_path` устанавливает путь, используемый Rails для поиска файлов локали. По умолчанию `config/locales/*.{yml,rb}`.

* `config.i18n.fallbacks` устанавливает поведение фолбэка для отсутствующих переводов. Вот 3 примера использования этой опции:

  * Можно установить опции `true` для использования локали по умолчанию в качестве фолбэка следующим образом:

    ```ruby
    config.i18n.fallbacks = true
    ```

  * Или можно установить массив локалей в качестве фолбэка так:

    ```ruby
    config.i18n.fallbacks = [:tr, :en]
    ```

  * Или можно установить различные фолбэки для различных локалей. Например, если хотите использовать `:tr` для `:az` и `:de`, `:en` для `:da` в качестве фолбэков, можно сделать так:

    ```ruby
    config.i18n.fallbacks = { az: :tr, da: [:de, :en] }
    # или
    config.i18n.fallbacks.map = { az: :tr, da: [:de, :en] }
    ```

### Конфигурирование Active Model

* `config.active_model.i18n_full_message` это булево значение, управляющее, может ли формат ошибки `full_message` быть переопределен на уровне атрибута или модели в файлах локали. По умолчанию `false`.

### Конфигурирование Active Record

`config.active_record` включает ряд конфигурационных опций:

* `config.active_record.logger` принимает логгер, соответствующий интерфейсу Log4r или дефолтного класса Ruby Logger, который затем передается на любые новые сделанные соединения с базой данных. Можете получить этот логгер, вызвав `logger` или на любом классе модели Active Record, или на экземпляре модели Active Record. Установите его в nil, чтобы отключить логирование.

* `config.active_record.primary_key_prefix_type` позволяет настроить именование столбцов первичного ключа. По умолчанию Rails полагает, что столбцы первичного ключа именуются `id` (и эта конфигурационная опция не нуждается в установке). Есть два возможных варианта:
    * `:table_name` сделает первичный ключ для класса Customer как `customerid`
    * `:table_name_with_underscore` сделает первичный ключ для класса Customer как `customer_id`

* `config.active_record.table_name_prefix` позволяет установить глобальную строку, добавляемую в начало имен таблиц. Если установить ее равным `northwest_`, то класс Customer будет искать таблицу `northwest_customers`. По умолчанию это пустая строка.

* `config.active_record.table_name_suffix` позволяет установить глобальную строку, добавляемую в конец имен таблиц. Если установить ее равным `_northwest`, то класс Customer будет искать таблицу `customers_northwest`. По умолчанию это пустая строка.

* `config.active_record.schema_migrations_table_name` позволяет установить строку, которая будет использоваться как имя таблицы для миграций схемы.

* `config.active_record.internal_metadata_table_name` позволяет установить строку, которая будет использоваться как имя таблицы для внутренних метаданных.

* `config.active_record.protected_environments` позволяет установить массив имен сред, где деструктивные экшны должны быть запрещены.

* `config.active_record.pluralize_table_names` определяет, должен Rails искать имена таблиц базы данных в единственном или множественном числе. Если установлено `true` (по умолчанию), то класс Customer будет использовать таблицу `customers`. Если установить `false`, то класс Customers будет использовать таблицу `customer`.

* `config.active_record.default_timezone` определяет, использовать `Time.local` (если установлено `:local`) или `Time.utc` (если установлено `:utc`) для считывания даты и времени из базы данных. По умолчанию `:utc`.

* `config.active_record.schema_format` регулирует формат для выгрузки схемы базы данных в файл. Опции следующие: `:ruby` (по умолчанию) для независимой от типа базы данных версии, зависимой от миграций, или `:sql` для набора (потенциально зависимого от типа БД) выражений SQL.

* `config.active_record.error_on_ignored_order` определяет, должна ли быть вызвана ошибка, если во время порционного (batch) запроса была проигнорирована сортировка или лимит. Опцией может быть либо `true` (вызывается ошибка), либо `false` (предупреждение). По умолчанию `false`.

* `config.active_record.timestamped_migrations` регулирует, должны ли миграции нумероваться серийными номерами или временными метками. По умолчанию `true` для использования временных меток, которые более предпочтительны, если над одним проектом работают несколько разработчиков.

* `config.active_record.lock_optimistically` регулирует, должен ли Active Record использовать оптимистическую блокировку. По умолчанию `true`.

* `config.active_record.cache_timestamp_format` управляет форматом значения временной метки в ключе кэширования. По умолчанию `:nsec`.

* `config.active_record.record_timestamps` это булево значение, управляющее, должна ли происходить временная метка операций модели `create` и `update`. Значение по умолчанию `true`.

* `config.active_record.partial_writes` это булево значение, управляющее, должны ли использоваться частичные записи (т.е. обновления только тех атрибутов, которые помечены dirty). Отметьте, что при использовании частичной записи также можно использовать оптимистическую блокировку `config.active_record.lock_optimistically`, так как конкурентные обновления могут записывать атрибуты, основываясь на возможном устаревшем статусе чтения. Значение по умолчанию `true`.

* `config.active_record.maintain_test_schema` это булево значение, управляющее, должен ли Active Record пытаться сохранять вашу тестовую базу данных актуальной с `db/schema.rb` (или `db/structure.sql`) при запуске тестов. По умолчанию `true`.

* `config.active_record.dump_schema_after_migration` это флажок, который контролирует, должна ли происходить выгрузка схемы (`db/schema.rb` или `db/structure.sql`) при запуске миграций. Он установлен `false` в `config/environments/production.rb`, генерируемом Rails. Значение по умолчанию `true`, если эта конфигурация не установлена.

* `config.active_record.dump_schemas` управляет, какие схемы баз данных будут выгружаться при вызове `db:structure:dump`. Опции: `:schema_search_path` (по умолчанию), при которой выгружается любая схема, перечисленная в `schema_search_path`, `:all`, при которой выгружаются все схемы, независимо от `schema_search_path`, или строки со схемами, разделенными через запятую.

* `config.active_record.belongs_to_required_by_default` это булево значение и управляет, будет ли валидация записи падать, если отсутствует связь `belongs_to`.

* `config.active_record.warn_on_records_fetched_greater_than` позволяет установить порог для предупреждения для итогового размера запроса. Если количество возвращаемых записей в запросе будет превышать пороговое значение, запишется предупреждение. Это может быть полезным для выявления запросов, которые могут быть причиной увеличения требуемой памяти.

* `config.active_record.index_nested_attribute_errors` позволяет ошибкам для вложенных отношений `has_many` также быть отраженными с индексом. По умолчанию `false`.

* `config.active_record.use_schema_cache_dump` позволяет пользователям получить информацию о кэше схемы из `db/schema_cache.yml` (сгенерированного с помощью `rails db:schema:cache:dump`), вместо отправления запроса в базу данных для получения этой информации. По умолчанию `true`.

Адаптер MySQL добавляет дополнительную конфигурационную опцию:

* `ActiveRecord::ConnectionAdapters::Mysql2Adapter.emulate_booleans` регулирует, должен ли Active Record рассматривать все столбцы `tinyint(1)` как boolean. По умолчанию `true`.

Адаптер SQLite3Adapter добавляет еще одну опцию конфигурации:

* `ActiveRecord::ConnectionAdapters::SQLite3Adapter.represent_boolean_as_integer` указывает, хранятся ли булевые значения в базах данных sqlite3 как 1 и 0 или 't' и 'f'. Выходное значение `ActiveRecord::ConnectionAdapters::SQLite3Adapter.represent_boolean_as_integer`, установленное в false устарело. Базы данных SQLite использовали 't' и 'f' для сериализации булевых значений, и следует преобразовать старые данные в 1 и 0 (их нативную булевую сериализацию), прежде чем устанавливать этот флаг в true. Преобразование может быть выполнено с помощью настройки Rake задачи, которая запускает

    ```ruby
    ExampleModel.where("boolean_column = 't'").update_all(boolean_column: 1)
    ExampleModel.where("boolean_column = 'f'").update_all(boolean_column: 0)
    ```

  для всех моделей и всех булевых столбцов, после чего флаг должен быть установлен в true с помощью добавления следующего в файл application.rb:

    ```ruby
    Rails.application.config.active_record.sqlite3.represent_boolean_as_integer = true
    ```

Выгрузчик схемы добавляет дополнительную конфигурационную опцию:

* `ActiveRecord::SchemaDumper.ignore_tables` принимает массив таблиц, которые _не_ должны быть включены в любой генерируемый файл схемы.

* `ActiveRecord::SchemaDumper.fk_ignore_pattern` позволяет настроить другое регулярное выражение, которое будет использоваться для определения того, следует ли выгружать имя внешнего ключа из db/schema.rb или нет. По умолчанию имена внешних ключей, начинающиеся с `fk_rails_`, не экспортируются в выгрузку схемы базы данных. По умолчанию используется `/^fk_rails_[0-9a-f]{10}$/`.

### Конфигурирование Action Controller

`config.action_controller` включает несколько конфигурационных настроек:

* `config.action_controller.asset_host` устанавливает хост для ассетов. Полезна, когда для хостинга ассетов используются CDN, или когда вы хотите обойти встроенную в браузеры политику ограничения домена при использовании различных псевдонимов доменов.

* `config.action_controller.perform_caching` конфигурирует, должно ли приложение выполнять возможность кэширования, предоставленную компонентом Action Controller. Установлено `false` в режиме development, `true` в production.

* `config.action_controller.default_static_extension` конфигурирует расширение, используемое для кэшированных страниц. По умолчанию `.html`.

* `config.action_controller.include_all_helpers` устанавливает, должны ли быть все хелперы вьюх доступны везде или только в соответствующем контроллере. Если установлен `false`, методы `UsersHelper` будут доступны только во вьюхах, рендерящихся как часть `UsersController`. Если `true`, методы `UsersHelper` будут доступны везде. Поведением настройки по умолчанию (когда этой опции явно не установлено `true` или `false`) является то, что все хелперы вьюх доступны в каждом контроллере.

* `config.action_controller.logger` принимает логгер, соответствующий интерфейсу Log4r или дефолтного класса Ruby Logger, который затем используется для логирования информации от Action Controller. Установите его в `nil`, чтобы отключить логирование.

* `config.action_controller.request_forgery_protection_token` устанавливает имя параметра токена для RequestForgery. Вызов `protect_from_forgery` по умолчанию устанавливает его в `:authenticity_token`.

* `config.action_controller.allow_forgery_protection` включает или отключает защиту от CSRF. По умолчанию `false` в режиме тестирования и `true` в остальных режимах.

* `config.action_controller.forgery_protection_origin_check` настраивает,должен ли сверяться заголовок HTTP `Origin` с доменом сайта в качестве дополнительной защиты от межсайтовой подделки запроса.

* `config.action_controller.per_form_csrf_tokens` настраивает, должны ли токены CSRF быть валидными только для метода/экшна, для которого они сгенерированы.

* `config.action_controller.default_protect_from_forgery` определяет, будет ли добавлена защита от подделки в `ActionController:Base`. Значением по умолчанию является false. Стоит отметить, что это включено по умолчанию при загрузке значений для Rails 5.2.

* `config.action_controller.relative_url_root` может использоваться, что бы сообщить Rails, [деплой происходит в поддиректорию](#deploy-to-a-subdirectory-relative-url-root). По умолчанию `ENV['RAILS_RELATIVE_URL_ROOT']`.

* `config.action_controller.permit_all_parameters` устанавливает все параметры для массового назначения как разрешенные по умолчанию. Значение по умолчанию `false`.

* `config.action_controller.action_on_unpermitted_parameters` включает логирование или вызов исключения, если обнаружены параметры, которые не разрешены явно. Чтобы включить, установите `:log` или `:raise`. По умолчанию `:log` в средах development и test, и `false` во всех остальных средах.

* `config.action_controller.always_permitted_parameters` устанавливает список разрешенных параметров, которые разрешены по умолчанию. Значениями по умолчанию являются `['controller', 'action']`.

* `config.action_controller.enable_fragment_cache_logging` определяет, нужно ли логировать чтение и запись в кэш фрагментов в следующем расширенном формате:

    ```
    Read fragment views/v1/2914079/v1/2914079/recordings/70182313-20160225015037000000/d0bdf2974e1ef6d31685c3b392ad0b74 (0.6ms)
    Rendered messages/_message.html.erb in 1.2 ms [cache hit]
    Write fragment views/v1/2914079/v1/2914079/recordings/70182313-20160225015037000000/3b4e249ac9d168c617e32e84b99218b5 (1.1ms)
    Rendered recordings/threads/_thread.html.erb in 1.5 ms [cache miss]
    ```

  По умолчанию установлено `false`, что выводит результаты следующим образом:

    ```
    Rendered messages/_message.html.erb in 1.2 ms [cache hit]
    Rendered recordings/threads/_thread.html.erb in 1.5 ms [cache miss]
    ```

### Конфигурирование Action Dispatch

* `config.action_dispatch.session_store` устанавливает имя хранилища данных сессии. По умолчанию `:cookie_store`; другие валидные опции включают `:active_record_store`, `:mem_cache_store` или имя вашего собственного класса.

* `config.action_dispatch.default_headers` это хэш с заголовками HTTP, которые по умолчанию устанавливаются для каждого отклика. По умолчанию определены как:

    ```ruby
    config.action_dispatch.default_headers = {
      'X-Frame-Options' => 'SAMEORIGIN',
      'X-XSS-Protection' => '1; mode=block',
      'X-Content-Type-Options' => 'nosniff',
      'X-Download-Options' => 'noopen',
      'X-Permitted-Cross-Domain-Policies' => 'none',
      'Referrer-Policy' => 'strict-origin-when-cross-origin'
    }
    ```

* `config.action_dispatch.default_charset` указывает кодировку по умолчанию для всех рендеров. По умолчанию `nil`.

* `config.action_dispatch.tld_length` устанавливает длину TLD (домена верхнего уровня) для приложения. По умолчанию `1`.

* `config.action_dispatch.ignore_accept_header` используется для определения, нужно ли игнорировать заголовки accept запроса. По умолчанию `false`.

* `config.action_dispatch.x_sendfile_header` определяет специфичный для сервера заголовок X-Sendfile.Это полезно для ускоренной отдачи файлов с сервера. Например, можно установить 'X-Sendfile' для Apache.

* `config.action_dispatch.http_auth_salt` устанавливает значение соли HTTP Auth. По умолчанию `'http authentication'`.

* `config.action_dispatch.signed_cookie_salt` устанавливает значение соли для подписанных куки. По умолчанию `'signed cookie'`.

* `config.action_dispatch.encrypted_cookie_salt` устанавливает значение соли для зашифрованных куки. По умолчанию `'encrypted cookie'`.

* `config.action_dispatch.encrypted_signed_cookie_salt` устанавливает значение соли для подписанных зашифрованных куки. По умолчанию `'signed encrypted cookie'`.

* `config.action_dispatch.authenticated_encrypted_cookie_salt` устанавливает значение соли для аутентификационных зашифрованных куки. По умолчанию `'authenticated encrypted cookie'`.

* `config.action_dispatch.encrypted_cookie_cipher` устанавливает алгоритм шифрования, который будет использоваться для зашифрованных куки. По умолчанию `"aes-256-gcm"`.

* `config.action_dispatch.signed_cookie_digest` устанавливает дайджест, который будет использоваться для подписанных куки. По умолчанию `"SHA1"`.

* `config.action_dispatch.cookies_rotations` позволяет чередовать секреты, шифры и дайджесты для зашифрованных и подписанных куки.

* `config.action_dispatch.use_authenticated_cookie_encryption` определяет, используют подписанные и зашифрованные куки шифр AES-256-GCM или более старый шифр AES-256-CBC. По умолчанию `true`.

* `config.action_dispatch.use_cookies_with_metadata` включает запись куки с включенными метаданными о назначении и сроке действия. По умолчанию `true`.

* `config.action_dispatch.perform_deep_munge` конфигурирует, должен ли применяться метод `deep_munge` на параметрах. Подробнее смотрите в руководстве [Безопасность приложений на Rails](/ruby-on-rails-security-guide#unsafe-query-generation). По умолчанию `true`.

* `config.action_dispatch.rescue_responses` конфигурирует, какие исключения назначаются статусу HTTP. Он принимает хэш и можно указать пары исключение/статус. По умолчанию он определен как:

  ```ruby
  config.action_dispatch.rescue_responses = {
    'ActionController::RoutingError'               => :not_found,
    'AbstractController::ActionNotFound'           => :not_found,
    'ActionController::MethodNotAllowed'           => :method_not_allowed,
    'ActionController::UnknownHttpMethod'          => :method_not_allowed,
    'ActionController::NotImplemented'             => :not_implemented,
    'ActionController::UnknownFormat'              => :not_acceptable,
    'ActionController::InvalidAuthenticityToken'   => :unprocessable_entity,
    'ActionController::InvalidCrossOriginRequest'  => :unprocessable_entity,
    'ActionDispatch::Http::Parameters::ParseError' => :bad_request,
    'ActionController::BadRequest'                 => :bad_request,
    'ActionController::ParameterMissing'           => :bad_request,
    'Rack::QueryParser::ParameterTypeError'        => :bad_request,
    'Rack::QueryParser::InvalidParameterError'     => :bad_request,
    'ActiveRecord::RecordNotFound'                 => :not_found,
    'ActiveRecord::StaleObjectError'               => :conflict,
    'ActiveRecord::RecordInvalid'                  => :unprocessable_entity,
    'ActiveRecord::RecordNotSaved'                 => :unprocessable_entity
  }
  ```

  Любое ненастроенное исключение приведет к 500 Internal Server Error.

* `ActionDispatch::Callbacks.before` принимает блок кода для запуска до запроса.

* `ActionDispatch::Callbacks.after` принимает блок кода для запуска после запроса.

### Конфигурирование Action View

`config.action_view` включает несколько конфигурационных настроек:

* `config.action_view.cache_template_loading` контролирует, будут ли шаблоны перезагружены при каждом запросе. Значение по умолчанию устанавливается для `config.cache_classes`.

* `config.action_view.field_error_proc` предоставляет генератор HTML для отображения ошибок, приходящих от Active Model. По умолчанию:

    ```ruby
    Proc.new do |html_tag, instance|
      %Q(<div class="field_with_errors">#{html_tag}</div>).html_safe
    end
    ```

* `config.action_view.default_form_builder` говорит Rails, какой form builder использовать по умолчанию. По умолчанию это `ActionView::Helpers::FormBuilder`. Если хотите, чтобы после инициализации загружался ваш класс form builder (и, таким образом, перезагружался с каждым запросом в development), можно передать его как строку.

* `config.action_view.logger` принимает логгер, соответствующий интерфейсу Log4r или классу Ruby по умолчанию Logger, который затем используется для логирования информации от Action View. Установите `nil` для отключения логирования.

* `config.action_view.erb_trim_mode` задает режим обрезки, который будет использоваться ERB. По умолчанию `'-'`, которая включает обрезку висячих пробелов и новых строчек при использовании `<%= -%>` или `<%= =%>`. Подробнее смотрите в [документации по Erubis](http://www.kuwata-lab.com/erubis/users-guide.06.html#topics-trimspaces).

* `config.action_view.embed_authenticity_token_in_remote_forms` позволяет установить поведение по умолчанию для `authenticity_token` в формах с `remote: true`. По умолчанию установлен `false`, что означает, что remote формы не включают `authenticity_token`, что полезно при фрагментарном кэшировании формы. Remote формы получают аутентификацию из тега `meta`, поэтому встраивание бесполезно, если, конечно, вы не поддерживаете браузеры без JavaScript. В противном случае можно либо передать `authenticity_token: true` как опцию для формы, либо установить эту настройку в `true`.

* `config.action_view.prefix_partial_path_with_controller_namespace` определяет должны ли партиалы искаться в поддиректории шаблонов для контроллеров в пространстве имен, или нет. Например, рассмотрим контроллер с именем `Admin::ArticlesController`, который рендерит этот шаблон:

    ```erb
    <%= render @article %>
    ```

Настройка по умолчанию `true`, что использует партиал в `/admin/articles/_article.erb`. Установка значение в `false` будет рендерить `/articles/_article.erb`, что является тем же поведением, что и рендеринг из контроллера не в пространстве имен, такого как `ArticlesController`.

* `config.action_view.raise_on_missing_translations` определяет, должно ли быть вызвано исключение для отсутствующих переводов.

* `config.action_view.automatically_disable_submit_tag` определяет, должен ли `submit_tag` автоматически отключаться при клике, это по умолчанию `true`.

* `config.action_view.debug_missing_translation` определяет, должны ли ключи отсутствующих переводов оборачиваться в тег `<span>`. Это по умолчанию `true`.

* `config.action_view.form_with_generates_remote_forms` определяет, должны ли `form_with` генерировать remote формы или нет. Это по умолчанию `true`.

* `config.action_view.form_with_generates_ids` определяет, должны ли `form_with` генерировать ids на inputs. Это по умолчанию `true`.

* `config.action_view.default_enforce_utf8` определяет, генерируются ли формы со скрытым тегом, который заставляет старые версии Internet Explorer отправлять формы, закодированные в UTF-8. Это по умолчанию `false`.

* `config.action_view.finalize_compiled_template_methods` определяет, должны ли методы в `ActionView::CompiledTemplates` сами удалять собранные шаблоны, когда экземпляры шаблонов уничтожаются сборщиком мусора. Это помогает предотвратить утечку памяти в режиме разработки, но для больших тестовых наборов отключение этой опции в тестовой среде может повысить производительность. Это по умолчанию `true`.

### (configuring-action-mailer) Конфигурирование Action Mailer

Имеется несколько доступных настроек `ActionMailer::Base`:

* `config.action_mailer.logger` принимает логгер, соответствующий интерфейсу Log4r или класса Ruby по умолчанию Logger, который затем используется для логирования информации от Action Mailer. Установите его в `nil`, чтобы отключить логирование.

* `config.action_mailer.smtp_settings` позволяет детально сконфигурировать метод доставки `:smtp`. Она принимает хэш опций, который может включать любые из следующих опций:
    * `:address` - Позволяет использовать удаленный почтовый сервер. Просто измените его значение по умолчанию "localhost".
    * `:port` - В случае, если почтовый сервер не работает с портом 25, можно изменить это.
    * `:domain` - Если нужно определить домен HELO, это делается здесь.
    * `:user_name` - Если почтовый сервер требует аутентификацию, установите имя пользователя этой настройкой.
    * `:password` - Если почтовый сервер требует аутентификацию, установите пароль этой настройкой.
    * `:authentication` - Если почтовый сервер требует аутентификацию, здесь необходимо установить тип аутентификации. Это должен быть один из символов `:plain`, `:login`, `:cram_md5`.
    * `:enable_starttls_auto` - Определяет, включен ли STARTTLS на вашем сервере SMTP и начинает его использовать. По умолчанию `true`.
    * `:openssl_verify_mode` - При использовании TLS, можно установить, как OpenSSL проверяет сертификат. Это полезно, если необходимо валидировать самоподписанный и/или wildcard сертификат. Это может быть одна из констант проверки OpenSSL, `:none` или `:peer` -- или сама константа `OpenSSL::SSL::VERIFY_NONE` или `OpenSSL::SSL::VERIFY_PEER`, соответственно.
    * `:ssl/:tls` - Позволяет соединению SMTP использовать SMTP/TLS (SMTPS: SMTP поверх прямого соединения TLS).

* `config.action_mailer.sendmail_settings` Позволяет детально сконфигурировать метод доставки `sendmail`. Она принимает хэш опций, который может включать любые из этих опций:
    * `:location` - Место расположения исполняемого файла sendmail. По умолчанию `/usr/sbin/sendmail`.
    * `:arguments` - Аргументы командной строки. По умолчанию `-i`.

* `config.action_mailer.raise_delivery_errors` определяет, должна ли вызываться ошибка, если доставка письма не может быть завершена. По умолчанию `true`.

* `config.action_mailer.delivery_method` определяет метод доставки, по умолчанию `:smtp`. За подробностями обращайтесь к разделу по настройке в руководстве [Основы Action Mailer](/action_mailer_basics#action-mailer-configuration)

* `config.action_mailer.perform_deliveries` определяет, должна ли почта фактически доставляться. По умолчанию `true`; удобно установить ее `false` при тестировании.

* `config.action_mailer.default_options` конфигурирует значения по умолчанию Action Mailer. Используется для установки таких опций, как `from` или`reply_to` для каждого рассыльщика. Эти значения по умолчанию следующие:

    ```ruby
    mime_version:  "1.0",
    charset:       "UTF-8",
    content_type: "text/plain",
    parts_order:  ["text/plain", "text/enriched", "text/html"]
    ```

    Присвойте хэш для установки дополнительных опций:

    ```ruby
    config.action_mailer.default_options = {
      from: "noreply@example.com"
    }
    ```

* `config.action_mailer.observers` регистрирует обсерверы, которые будут уведомлены при доставке почты.

    ```ruby
    config.action_mailer.observers = ["MailObserver"]
    ```

* `config.action_mailer.interceptors` регистрирует перехватчики, которые будут вызваны до того, как почта будет отослана.

    ```ruby
    config.action_mailer.interceptors = ["MailInterceptor"]
    ```

* `config.action_mailer.preview_interceptors` регистрирует перехватчики, которые будут вызваны до того, как почта будет предварительно просмотрена.

    ```ruby
    config.action_mailer.preview_interceptors = ["MyPreviewMailInterceptor"]
    ```

* `config.action_mailer.preview_path` определяет место расположения превью рассыльщика.

    ```ruby
    config.action_mailer.preview_path = "#{Rails.root}/lib/mailer_previews"
    ```

* `config.action_mailer.show_previews` включает или отключает превью рассыльщика. По умолчанию `true` в development.

    ```ruby
    config.action_mailer.show_previews = false
    ```

* `config.action_mailer.deliver_later_queue_name` указывает название очереди для рассыльщиков. По умолчанию `mailers`.

* `config.action_mailer.perform_caching` указывает, должно ли выполняться кэширование фрагментов для шаблонов рассыльщиков. Это по умолчанию `false` во всех средах.

### (configuring-active-support) Конфигурирование Active Support

Имеется несколько конфигурационных настроек для Active Support:

* `config.active_support.bare` включает или отключает загрузку `active_support/all` при загрузке Rails. По умолчанию `nil`, что означает, что `active_support/all` загружается.

* `config.active_support.test_order` устанавливает порядок, в котором выполняются тестовые случаи. Возможные значения `:random` и `:sorted`. По умолчанию `:random`.

* `config.active_support.escape_html_entities_in_json` включает или отключает экранирование сущностей HTML в сериализации JSON. По умолчанию `true`.

* `config.active_support.use_standard_json_time_format` включает или отключает сериализацию дат в формат ISO 8601. По умолчанию `true`.

* `config.active_support.time_precision` устанавливает точность значений времени, кодируемого в JSON. По умолчанию `3`.

* `config.active_support.use_sha1_digests` указывает, следует ли использовать SHA-1 вместо MD5 для генерации дайджестов для не конфиденциальных (non-sensitive) данных, таких как заголовок ETag. По умолчанию false.

* `config.active_support.use_authenticated_message_encryption` указывает, следует ли использовать аутентификационное шифрование AES-256-GCM в качестве шифра по умолчанию для шифрования сообщений вместо AES-256-CBC. По умолчанию false, но включено при загрузке умолчаний для Rails 5.2.

* `ActiveSupport::Logger.silencer` устанавливают `false`, чтобы отключить возможность silence logging в блоке. По умолчанию `true`.

* `ActiveSupport::Cache::Store.logger` определяет логгер, используемый в операциях хранения кэша.

* `ActiveSupport::Deprecation.behavior` альтернативный сеттер для `config.active_support.deprecation`, конфигурирующий поведение предупреждений об устаревании в Rails.

* `ActiveSupport::Deprecation.silence` принимает блок, в котором все предупреждения об устаревании умалчиваются.

* `ActiveSupport::Deprecation.silenced` устанавливает, отображать ли предупреждения об устаревании.

### Конфигурирование Active Job

`config.active_job` предоставляет следующие конфигурационные опции:

* `config.active_job.queue_adapter` устанавливает адаптер для бэкенда очередей. По умолчанию адаптер `:async`. Актуальный список встроенных адаптеров смотрите в [документации ActiveJob::QueueAdapters API](http://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html).

    ```ruby
    # Убедитесь, что гем адаптера есть в вашем Gemfile
    # и следуйте определенным инструкция по установке
    # и деплою.
    config.active_job.queue_adapter = :sidekiq
    ```

* `config.active_job.default_queue_name` может быть использована для того, чтобы изменить название очереди по умолчанию. По умолчанию это `"default"`.

    ```ruby
    config.active_job.default_queue_name = :medium_priority
    ```

* `config.active_job.queue_name_prefix` позволяет установить опциональный непустой префикс к названию очереди для всех заданий. По умолчанию пустой и не используется.

    Со следующей настройкой задания будут добавляться в очередь `production_high_priority`, при запуске в production:
    ```ruby
    config.active_job.queue_name_prefix = Rails.env
    ```

    ```ruby
    class GuestsCleanupJob < ActiveJob::Base
      queue_as :high_priority
      #....
    end
    ```

* `config.active_job.queue_name_delimiter` имеет значение по умолчанию `'_'`. Если `queue_name_prefix` установлена, тогда `queue_name_delimiter` соединяет префикс и название очереди без префикса.

    Со следующей настройкой задания будут добавлять в очередь `video_server.low_priority`:

    ```ruby
    # префикс должен быть установлен для использования разделителя
    config.active_job.queue_name_prefix = 'video_server'
    config.active_job.queue_name_delimiter = '.'
    ```

    ```ruby
    class EncoderJob < ActiveJob::Base
      queue_as :low_priority
      #....
    end
    ```

* `config.active_job.logger` принимает логгер, соответствующий интерфейсу Log4r или дефолтного класса Ruby Logger, который затем используется для логирования информации от Action Job. Вы можете получить этот логгер вызвав `logger` в классе Active Job или экземпляре Active Job. Установите его в `nil`, чтобы отключить логирование.

* `config.active_job.custom_serializers` позволяет устанавливать собственные сериализаторы аргументов. По умолчанию используется `[]`.

### Конфигурация Action Cable

* `config.action_cable.url` принимает строку с URL, на котором размещается ваш сервер Action Cable. Следует использовать эту опцию, если вы запускаете серверы Action Cable отдельно от основного приложения.

* `config.action_cable.mount_path` принимает строку, куда монтировать Action Cable, как часть процесса основного сервера. По умолчанию `/cable`. Ей можно указать nil, чтобы не монтировать Action Cable как часть вашего обычного сервера Rails.

### (configuring-active-storage) Конфигурирование Active Storage

`config.active_storage` предоставляет следующие опции конфигурации:

* `config.active_storage.variant_processor` принимает символ `:mini_magick` или `:vips`, указывая, будут ли варианты преобразования выполняться с помощью MiniMagick или ruby-vips. По умолчанию это `:mini_magick`.

* `config.active_storage.analyzers` принимает массив классов, указывающий анализаторы, доступные для blobs в Active Storage. По умолчанию используется `[ActiveStorage::Analyzer::ImageAnalyzer, ActiveStorage::Analyzer::VideoAnalyzer]`. Первый может извлекать ширину и высоту blob изображения; последний может извлекать ширину, высоту, длительность, угол и соотношение сторон blob видео.

* `config.active_storage.previewers` принимает массив классов, указывающий на средства предварительного просмотра изображений, доступные для blobs в Active Storage. По умолчанию используется `[ActiveStorage::Previewer::PDFPreviewer, ActiveStorage::Previewer::VideoPreviewer]`. Первый может генерировать миниатюру из первой страницы blob PDF; последний из соответствующего кадра blob видео.

* `config.active_storage.paths` принимает хэш опций, с указанием мест расположения команд средств предварительного просмотра/анализатора. По умолчанию используется `{}`, что означает, что команды будут искать по дефолтному пути. Можно включить любую из следующих опций:
    * `:ffprobe` - Место расположения исполняемого ffprobe.
    * `:mutool` - Место расположения исполняемого mutool.
    * `:ffmpeg` - Место расположения исполняемого ffmpeg.

   ```ruby
   config.active_storage.paths[:ffprobe] = '/usr/local/bin/ffprobe'
   ```

* `config.active_storage.variable_content_types` принимает массив строк, указывающий типы содержимого, которые Active Storage может преобразовывать через ImageMagick. По умолчанию используется `%w(image/png image/gif image/jpg image/jpeg image/vnd.adobe.photoshop image/vnd.microsoft.icon)`.

* `config.active_storage.content_types_to_serve_as_binary` принимает массив строк, указывающий типы содержимого, которые Active Storage всегда будет отдавать в качестве прикрепленного файла, а не встроенного. По умолчанию используется `%w(text/html text/javascript image/svg+xml application/postscript application/x-shockwave-flash text/xml application/xml application/xhtml+xml)`.

* `config.active_storage.queue` может быть использован для установки имени очереди Active Job, используемой для выполнения заданий, таких как анализ содержимого blob или очистки (purging) блога.

  ```ruby
  config.active_storage.queue = :low_priority
  ```

* `config.active_storage.logger` может быть использован для установки логгера, используемого Active Storage. Принимает логгер, соответствующий интерфейсу Log4r или дефолтному классу Logger в Ruby.

  ```ruby
  config.active_storage.logger = ActiveSupport::Logger.new(STDOUT)
  ```

* `config.active_storage.service_urls_expire_in` определяет срок действия по умолчанию для URL, генерируемых с помощью:
  * `ActiveStorage::Blob#service_url`
  * `ActiveStorage::Blob#service_url_for_direct_upload`
  * `ActiveStorage::Variant#service_url`

  По умолчанию 5 минут.

* `config.active_storage.routes_prefix` может быть использована для установки префикса маршрута для маршрутов, обслуживаемых Active Storage. Принимает строку, с которой будут начинаться генерируемые маршруты.

  ```ruby
  config.active_storage.routes_prefix = '/files'
  ```

  По умолчанию `/rails/active_storage`

### Конфигурирование базы данных

Почти каждое приложение на Rails взаимодействует с базой данных. Можно подключаться к базе данных с помощью установки переменной окружения `ENV['DATABASE_URL']` или с помощью использования файла `config/database.yml`.

При использовании файла `config/database.yml` можно указать всю информацию, необходимую для доступа к базе данных:

```yaml
development:
  adapter: postgresql
  database: blog_development
  pool: 5
```

Это будет подключаться к базе данных по имени `blog_development` при помощи адаптера `postgresql`. Та же самая информация может быть сохранена в URL и предоставлена с помощью переменной среды следующем образом:

```ruby
> puts ENV['DATABASE_URL']
postgresql://localhost/blog_development?pool=5
```

Файл `config/database.yml`содержит разделы для трех различных сред, в которых по умолчанию может быть запущен Rails:

* Среда `development` используется на вашем компьютере для разработки или локальном компьютере для того, чтобы вы могли взаимодействовать с приложением.
* Среда `test` используется при запуске автоматических тестов.
* Среда `production` используется, когда вы развертываете свое приложения во всемирной сети для использования.

Если хотите, можно указать URL внутри `config/database.yml`

```
development:
  url: postgresql://localhost/blog_development?pool=5
```

Файл `config/database.yml` может содержать теги ERB `<%= %>`. Все внутри тегов будет вычислено как код Ruby. Это можно использовать для вставки данных из переменных среды или для выполнения вычислений для генерации необходимой информации о соединении.

TIP: Вам не нужно обновлять конфигурации баз данных вручную. Если взглянете на опции генератора приложения, то увидите, что одна из опций называется `--database`. Эта опция позволяет выбрать адаптер из списка наиболее часто используемых реляционных баз данных. Можно даже запускать генератор неоднократно: `cd .. && rails new blog —database=mysql`. После того, как подтвердите перезапись `config/database.yml`, ваше приложение станет использовать MySQL вместо SQLite. Подробные примеры распространенных соединений с базой данных указаны ниже.

### Предпочтение соединения

Так как существует два способа настройки соединения (с помощью `config/database.yml` или с помощью переменной среды), важно понять, как они могут взаимодействовать.

Если имеется пустой файл `config/database.yml`, но существует `ENV['DATABASE_URL']`, Rails соединится с базой данных с помощью переменной среды:

```
$ cat config/database.yml

$ echo $DATABASE_URL
postgresql://localhost/my_database
```

Если имеется `config/database.yml`, но нет `ENV['DATABASE_URL']`, тогда для соединения с базой данных будет использован этот файл:

```
$ cat config/database.yml
development:
  adapter: postgresql
  database: my_database
  host: localhost

$ echo $DATABASE_URL
```

Если имеется и `config/database.yml`, и `ENV['DATABASE_URL']`, Rails будет объединять конфигурации вместе. Чтобы лучше понять, обратимся к примерам.

При дублирующей информации о соединении, приоритет имеет переменная среды:

```
$ cat config/database.yml
development:
  adapter: sqlite3
  database: NOT_my_database
  host: localhost

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"postgresql", "database"=>"my_database", "host"=>"localhost"}
    @url="postgresql://localhost/my_database">
  ]
```

Здесь адаптер, хост и база данных соответствуют информации в `ENV['DATABASE_URL']`.

Если предоставлена недублирующая информация, вы получите все уникальные значения, в случае любых конфликтов переменная среды также имеет приоритет.

```
$ cat config/database.yml
development:
  adapter: sqlite3
  pool: 5

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"postgresql", "database"=>"my_database", "host"=>"localhost", "pool"=>5}
    @url="postgresql://localhost/my_database">
  ]
```

Поскольку pool не содержится в предоставленной информации о соединении в `ENV['DATABASE_URL']`, его информация объединяется. Так как `adapter` дублирован, информация о соединении взята из `ENV['DATABASE_URL']`.

Единственных способ явно не использовать информацию о соединении из `ENV['DATABASE_URL']`, это определить явный URL соединения с использованием ключа `"url"`:

```
$ cat config/database.yml
development:
  url: sqlite3:NOT_my_database

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"sqlite3", "database"=>"NOT_my_database"}
    @url="sqlite3:NOT_my_database">
  ]
```

Тут игнорируется информация о соединении из `ENV['DATABASE_URL']`.

Так как возможно встроить ERB в `config/database.yml`, хорошей практикой является явно показать, что вы используете `ENV['DATABASE_URL']` для соединения с вашей базой данных. Это особенно полезно в production, так как вы не должны показывать секреты, такие как пароль от базы данных, в системе управления версиями (такой как Git).

```
$ cat config/database.yml
production:
  url: <%= ENV['DATABASE_URL'] %>
```

Теперь поведение понятное, что мы используем только информацию о соединении из `ENV['DATABASE_URL']`.

#### Конфигурирование базы данных SQLite3

В Rails есть встроенная поддержка [SQLite3](http://www.sqlite.org), являющейся легким несерверным приложением по управлению базами данных. Хотя нагруженная среда production может перегрузить SQLite, она хорошо работает для разработки и тестирования. Rails при создании нового проекта использует базу данных SQLite, но вы всегда можете изменить это позже.

Вот раздел дефолтного конфигурационного файла (`config/database.yml`) с информацией о соединении для среды development:

```yaml
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
```

NOTE: В этом руководстве мы используем базу данных SQLite3 для хранения данных, поскольку эта база данных работает с нулевыми настройками. Rails также поддерживает MySQL (включая MariaDB) и PostgreSQL "из коробки", и имеет плагины для многих СУБД. Если вы уже используете базу данных в работе, в Rails скорее всего есть адаптер для нее.

#### Конфигурирование базы данных MySQL или MariaDB

Если вы выбрали MySQL или MariaDB вместо SQLite3, ваш `config/database.yml` будет выглядеть немного по-другому. Вот раздел development:

```yaml
development:
  adapter: mysql2
  database: blog_development
  pool: 5
  username: root
  password:
  socket: /tmp/mysql.sock
```

Если в вашей базе для разработки есть пользователь root с пустым паролем, эта конфигурация у вас заработает. В противном случае измените username и password в разделе `development` на правильные.

NOTE: Если версия MySQL 5.5 или 5.6, и вы хотите использовать кодировку `utf8mb4` по умолчанию, настройте ваш сервер MySQL, чтобы он поддерживал более длинные префиксы ключей, включив системную переменную `innodb_large_prefix`.

Advisory Locks в MySQL по умолчанию включены и используются, чтобы сделать миграции базы данных безопасными. Их можно отключить, установив `advisory_locks` в `false`:

```yaml
production:
  adapter: mysql2
  advisory_locks: false
```

#### Конфигурирование базы данных PostgreSQL

Если вы выбрали PostgreSQL, ваш `config/database.yml` будет модифицирован для использования базы данных PostgreSQL:

```yaml
development:
  adapter: postgresql
  encoding: unicode
  database: blog_development
  pool: 5
```

По умолчанию Active Record использует особенности базы данных, такие как prepared statements и advisory locks. Вам может потребоваться отключить эти особенности, если вы используете внешний пул соединения, такой как PgBouncer:

```yaml
production:
  adapter: postgresql
  prepared_statements: false
  advisory_locks: false
```

Если включены, Active Record по умолчанию создаст до `1000` prepared statements на соединение с базой данных. Чтобы модифицировать это поведение, можно установить `statement_limit` в другое значение:

```
production:
  adapter: postgresql
  statement_limit: 200
```

Чем больше используется prepared statements, тем больше нужно памяти вашей базе данных. Если ваша база данных PostgreSQL достигает лимитов памяти, попробуйте снизить `statement_limit` или отключить prepared statements.

#### Конфигурирование базы данных SQLite3 для платформы JRuby

Если вы выбрали SQLite3 и используете JRuby, ваш `config/database.yml` будет выглядеть немного по-другому. Вот раздел development:

```yaml
development:
  adapter: jdbcsqlite3
  database: db/development.sqlite3
```

#### Конфигурирование базы данных MySQL или MariaDB для платформы JRuby

Если вы выбрали MySQL или MariaDB и используете JRuby, ваш `config/database.yml` будет выглядеть немного по-другому. Вот раздел development:

```yaml
development:
  adapter: jdbcmysql
  database: blog_development
  username: root
  password:
```

#### Конфигурирование базы данных PostgreSQL для платформы JRuby

Если вы выбрали PostgreSQL и используете JRuby, ваш `config/database.yml` будет выглядеть немного по-другому. Вот раздел development:

```yaml
development:
  adapter: jdbcpostgresql
  encoding: unicode
  database: blog_development
  username: blog
  password:
```

Измените username и password в разделе `development` на правильные.

### (creating-rails-environments) Создание сред Rails

По умолчанию Rails поставляется с тремя средами: "development", "test" и "production". Хотя в большинстве случаев их достаточно, бывают условия, когда нужно больше сред.

Представим, что у вас есть сервер, отражающий среду production, но используемый только для тестирования. Такой сервер обычно называется "staging server". Для определения среды с именем "staging" для этого сервера, просто создайте файл с именем `config/environments/staging.rb`. В качестве исходного содержимого используйте любой файл, существующий в `config/environments`, а затем сделайте в нем необходимые изменения.

Эта среда ничем не отличается от одной из стандартных, сервер запускается с помощью `rails server -e staging`, консоль с помощью `rails console -e staging`, работает `Rails.env.staging?`, и т.д.


### (Deploy to a subdirectory relative url root) Деплой в поддиректорию (относительно корневого url)

По умолчанию Rails ожидает, что ваше приложение запускается в корне (т.е. `/`). Этот раздел объяснит, как запустить ваше приложение внутри директории.

Допустим, мы хотим задеплоить наше приложение в "/app1". Rails необходимо знать эту директорию для генерации подходящих маршрутов:

```ruby
config.relative_url_root = "/app1"
```

альтернативно можно установить переменную среды `RAILS_RELATIVE_URL_ROOT`.

Теперь Rails будет добавлять "/app1" в начало каждой сгенерированной ссылки.

#### Использование Passenger

В Passenger запустить приложение в поддиректории просто. Подходящую конфигурацию можно найти в [руководстве по Passenger](https://www.phusionpassenger.com/library/deploy/apache/deploy/ruby/#deploying-an-app-to-a-sub-uri-or-subdirectory).

#### Использование обратного прокси

Размещение вашего приложения с использованием обратного прокси имеет определенные преимущества перед традиционным размещением. Они позволяют больше контролировать ваш сервер, располагая по слоям компоненты, требуемые вашему приложению.

Многие веб-серверы могут быть использованы в качестве прокси сервера для балансировки сторонних элементов, таких как кэширующие сервера или сервера приложений.

Одним из таких серверов приложений является [Unicorn](https://unicorn.bogomips.org/), запущенный за обратным прокси.

В этом случае необходимо настроить прокси сервер (NGINX, Apache и т.д.) принимать соединения из вашего сервера приложения (Unicorn). По умолчанию Unicorn будет слушать соединения TCP на 8080 порту, но можно изменить порт, или настроить использование сокетов.

Можно найти подробности в [Unicorn readme](https://unicorn.bogomips.org/README.html) и понять лежащую в основе [философию](https://unicorn.bogomips.org/PHILOSOPHY.html).

Как только вы настроили сервер приложения, необходимо проксировать запросы к нему, настроив надлежащим образом веб-сервер. Например, ваш конфиг NGINX может включать:

```
upstream application_server {
  server 0.0.0.0:8080;
}

server {
  listen 80;
  server_name localhost;

  root /root/path/to/your_app/public;

  try_files $uri/index.html $uri.html @app;

  location @app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://application_server;
  }

  # прочая конфигурация
}
```

Прочитайте актуальную информацию в [документации NGINX](https://nginx.org/en/docs/).

Настройка среды Rails
---------------------

Некоторые части Rails также могут быть сконфигурированы извне, предоставив переменные среды. Следующие переменные среды распознаются различными частями Rails:

* `ENV["RAILS_ENV"]` определяет среду Rails (production, development, test и так далее), под которой будет запущен Rails.

* `ENV["RAILS_RELATIVE_URL_ROOT"]` используется кодом роутинга для распознания URL при [деплое вашего приложение в поддиректории](#deploy-to-a-subdirectory-relative-url-root).

* `ENV["RAILS_CACHE_ID"]` и `ENV["RAILS_APP_VERSION"]` используются для генерация расширенных ключей кэша в коде кэширования Rails. Это позволит иметь несколько отдельных кэшей в одном и том же приложении.

(initialization) Использование файлов инициализаторов
-----------------------------------------------------

После загрузки фреймворка и любых гемов в вашем приложении, Rails приступает к загрузке инициализаторов. Инициализатор это любой файл с кодом ruby, хранящийся в `/config/initializers` вашего приложения. Инициализаторы могут использоваться для хранения конфигурационных настроек, которые должны быть выполнены после загрузки фреймворков и гемов, таких как опции для конфигурирования настроек для этих частей.

NOTE: Можно использовать подпапки для организации ваших инициализаторов, если нужно, так как Rails смотрит файловую иерархию в целом в папке `initializers` и ниже.

TIP: Хотя Rails поддерживает нумерацию имен файлов инициализации для целей порядка загрузки, лучше всего разместить любой код, который необходимо загрузить в определенном порядке в том же файле. Это уменьшает количество отказов имени файла, делает зависимости более явными и помогает выявить новые концепции в приложении.

События инициализации
---------------------

В Rails имеется 5 событий инициализации, которые могут быть встроены в разные моменты (отображено в порядке запуска):

* `before_configuration`: Это запустится как только константа приложения унаследуется от `Rails::Application`. Вызовы `config` будут вычислены до того, как это произойдет.

* `before_initialize`: Это запустится непосредственно перед процессом инициализации с помощью инициализатора `:bootstrap_hook`, расположенного рядом с началом процесса инициализации Rails.

* `to_prepare`: Запустится после того, как инициализаторы будут запущены для всех Railties (включая само приложение), но до нетерпеливой загрузки и построения стека промежуточных программ. Что еще более важно, запустится после каждого запроса в `development`, но только раз (при загрузке) в `production` и `test`.

* `before_eager_load`: Это запустится непосредственно после нетерпеливой загрузки, что является поведением по умолчанию для среды `production`, но не `development`.

* `after_initialize`: Запустится сразу после инициализации приложения, после запуска инициализаторов приложения из `config/initializers`.

Чтобы определить событие для них, используйте блочный синтаксис в подклассе `Rails::Application`, `Rails::Railtie` или `Rails::Engine`:

```ruby
module YourApp
  class Application < Rails::Application
    config.before_initialize do
      # тут идет инициализационный код
    end
  end
end
```

Это можно сделать также с помощью метода `config` на объекте `Rails.application`:

```ruby
Rails.application.config.before_initialize do
  # тут идет инициализационный код
end
```

WARNING: Некоторые части вашего приложения, в частности роутинг, пока еще не настроены в месте, где вызывается блок `after_initialize`.

### `Rails::Railtie#initializer`

В Rails имеется несколько инициализаторов, выполняющихся при запуске, все они определены с использованием метода `initializer` из `Rails::Railtie`. Вот пример инициализатора `initialize_whiny_nils` из Action Controller:

```ruby
initializer "action_controller.set_helpers_path" do |app|
  ActionController::Helpers.helpers_path = app.helpers_paths
end
```

Метод `initializer` принимает три аргумента: имя инициализатора, хэш опций (здесь не показан) и блок. В хэше опций может быть определен ключ `:before` для указания, перед каким инициализатором должен быть запущен новый инициализатор, и ключ `:after` определяет, после какого инициализатора запустить этот.

Инициализаторы, определенные методом `initializer`, будут запущены в порядке, в котором они определены, за исключением тех, в которых использованы методы `:before` или `:after`.

WARNING: Можно помещать свои инициализаторы до или после других инициализаторов в цепочки, пока это логично. Скажем, имеется 4 инициализатора, названные от "one" до "four" (определены в этом порядке), и вы определяете "four" идти _before_ "four", но _after_ "three", это не логично, и Rails не сможет установить ваш порядок инициализаторов.

Блочный аргумент метода `initializer` это экземпляр самого приложение, таким образом, можно получить доступ к его конфигурации, используя метод `config`, как это сделано в примере.

Поскольку `Rails::Application` унаследован от `Rails::Railtie` (опосредованно), можно использовать метод `initializer` в `config/application.rb` для определения инициализаторов для приложения.

### (initializers) Инициализаторы

Ниже приведен полный список всех инициализаторов, присутствующих в Rails в порядке, в котором они определены (и, следовательно, запущены, если не указано иное).

* `load_environment_hook`: Служит в качестве местозаполнителя, так что `:load_environment_config` может быть определено для запуска до него.

* `load_active_support`: Требует `active_support/dependencies`, настраивающий основу для Active Support. Опционально требует `active_support/all`, если `config.active_support.bare` не истинно, что является значением по умолчанию.

* `initialize_logger`: Инициализирует логгер (объект `ActiveSupport::Logger`) для приложения и делает его доступным как `Rails.logger`, если до него другой инициализатор не определит `Rails.logger`.

* `initialize_cache`: Если `Rails.cache` еще не установлен, инициализирует кэш, обращаясь к значению `config.cache_store` и сохраняя результат как `Rails.cache`. Если этот объект отвечает на метод `middleware`, его промежуточная программа вставляется до `Rack::Runtime` в стеке промежуточных программ.

* `set_clear_dependencies_hook`: Этот инициализатор - запускающийся, только если `cache_classes` установлена `false` - использует `ActionDispatch::Callbacks.after` для удаления констант, на которые ссылались на протяжении запроса от пространства объекта, так что они могут быть перезагружены в течение следующего запроса.

* `initialize_dependency_mechanism`: Если `config.cache_classes` true, конфигурирует `ActiveSupport::Dependencies.mechanism` требовать (`require`) зависимости, а не загружать (`load`) их.

* `bootstrap_hook`: Запускает все сконфигурированные блоки `before_initialize`.

* `i18n.callbacks`: В среде development, настраивает колбэк `to_prepare`, вызывающий `I18n.reload!`, если любая из локалей изменилась с последнего запроса. В режиме production этот колбэк запускается один раз при первом запросе.

* `active_support.deprecation_behavior`: Настраивает отчеты об устаревании для сред, по умолчанию `:log` для development, `:notify` для production и `:stderr` для test. Если для `config.active_support.deprecation` не установлено значение, то инициализатор подскажет пользователю сконфигурировать эту строчку в файле `config/environments` текущей среды. Можно установить массив значений.

* `active_support.initialize_time_zone`: Устанавливает для приложения временную зону по умолчанию, основываясь на настройке `config.time_zone`, которая по умолчанию равна "UTC".

* `active_support.initialize_beginning_of_week`: Устанавливает начало недели по умолчанию для приложения, основываясь на настройке `config.beginning_of_week`, которая по умолчанию `:monday`.

* `active_support.set_configs`: Настраивает Active Support с помощью настроек в `config.active_support` посылая имена методов в качестве сеттеров в `ActiveSupport` и передавая им значения.

* `action_dispatch.configure`: Конфигурирует `ActionDispatch::Http::URL.tld_length` быть равным значению `config.action_dispatch.tld_length`.

* `action_view.set_configs`: Устанавливает, чтобы Action View использовал настройки в `config.action_view`, посылая имена методов через `send` как сеттер в `ActionView::Base` и передавая в него значения.

* `action_controller.assets_config`: Инициализирует `config.actions_controller.assets_dir` директорией public приложения, если не сконфигурирована явно.

* `action_controller.set_helpers_path`: Устанавливает helpers_path у Action Controller равным helpers_path приложения.

* `action_controller.parameters_config`: Конфигурирует опции strong parameters для `ActionController::Parameters`.

* `action_controller.set_configs`: Устанавливает, чтобы Action Controller использовал настройки в `config.action_controller`, посылая имена методов через `send` как сеттер в `ActionController::Base` и передавая в него значения.

* `action_controller.compile_config_methods`: Инициализирует методы для указанных конфигурационных настроек, чтобы доступ к ним был быстрее.

* `active_record.initialize_timezone`: Устанавливает `ActiveRecord::Base.time_zone_aware_attributes` `true`, а также `ActiveRecord::Base.default_timezone` UTC. Когда атрибуты считываются из базы данных, они будут конвертированы во временную зону с использованием `Time.zone`.

* `active_record.logger`: Устанавливает `ActiveRecord::Base.logger` - если еще не установлен - как `Rails.logger`.

* `active_record.migration_error`: Конфигурирует промежуточную программу для проверки невыполненных миграций.

* `active_record.check_schema_cache_dump`: Загружает кэш выгрузки схемы, если настроен и доступен.

* `active_record.warn_on_records_fetched_greater_than`: Включает предупреждения, когда запросы возвращают большое количество записей.

* `active_record.set_configs`: Устанавливает, чтобы Active Record использовал настройки в `config.active_record`, посылая имена методов через `send` как сеттер в `ActiveRecord::Base` и передавая в него значения.

* `active_record.initialize_database`: Загружает конфигурацию базы данных (по умолчанию) из `config/database.yml` и устанавливает соединение для текущей среды.

* `active_record.log_runtime`: Включает `ActiveRecord::Railties::ControllerRuntime`, ответственный за отчет в логгер по времени, затраченному вызовом Active Record для запроса.

* `active_record.set_reloader_hooks`: Сбрасывает все перезагружаемые соединения к базе данных, если `config.cache_classes` установлена `false`.

* `active_record.add_watchable_files`: Добавляет файлы `schema.rb` и `structure.sql` в отслеживаемые.

* `active_job.logger`: Устанавливает `ActiveRecord::Base.logger` - если еще не установлен - как `Rails.logger`.

* `active_job.set_configs`: Устанавливает, чтобы Active Job использовал настройки `config.active_job`, посылая имена методов через `send` как сеттер в `ActiveRecord::Base` и передавая в него значения.

* `action_mailer.logger`: Устанавливает `ActionMailer::Base.logger` - если еще не установлен - как `Rails.logger`.

* `action_mailer.set_configs`: Устанавливает, чтобы Action Mailer использовал настройки в `config.action_mailer`, посылая имена методов через `send` как сеттер в `ActionMailer::Base` и передавая в него значения.

* `action_mailer.compile_config_methods`: Инициализирует методы для указанных конфигурационных настроек, чтобы доступ к ним был быстрее.

* `set_load_path`: Этот инициализатор запускается перед `bootstrap_hook`. Добавляет пути, определенные `config.load_paths`, и пути автозагрузки к `$LOAD_PATH`.

* `set_autoload_paths`: Этот инициализатор запускается перед `bootstrap_hook`. Добавляет все поддиректории `app` и пути, определенные `config.autoload_paths`, `config.eager_load_paths` и `config.autoload_once_paths` в `ActiveSupport::Dependencies.autoload_paths`.

* `add_routing_paths`: Загружает (по умолчанию) все файлы `config/routes.rb` (в приложении и railties, включая engine-ы) и настраивает маршруты для приложения.

* `add_locales`: Добавляет файлы в `config/locales` (из приложения, railties и engine-ов) в `I18n.load_path`, делая доступными переводы в этих файлах.

* `add_view_paths`: Добавляет директорию `app/views` из приложения, railties и engine-ов в путь поиска файлов вьюх приложения.

* `load_environment_config`: Загружает файл `config/environments` для текущей среды.

* `prepend_helpers_path`: Добавляет директорию `app/helpers` из приложения, railties и engine-ов в путь поиска файлов хелперов приложения.

* `load_config_initializers`: Загружает все файлы Ruby из `config/initializers` в приложении, railties и engine-ах. Файлы в этой директории могут использоваться для хранения конфигурационных настроек, которые нужно сделать после загрузки всех фреймворков.

* `engines_blank_point`: Предоставляет точку инициализации для хука, если нужно что-то сделать до того, как загрузятся engine-ы. После этой точки будут запущены все инициализаторы railties и engine-ов.

* `add_generator_templates`: Находит шаблоны для генераторов в `lib/templates` приложения, railties и engine-ов, и добавляет их в настройку `config.generators.templates`, что делает шаблоны доступными для всех ссылающихся генераторов.

* `ensure_autoload_once_paths_as_subset`: Убеждается, что `config.autoload_once_paths` содержит пути только из `config.autoload_paths`. Если она содержит другие пути, будет вызвано исключение.

* `add_to_prepare_blocks`: Блок для каждого вызова `config.to_prepare` в приложении, railtie или engine добавляется в колбэк `to_prepare` для Action Dispatch, который будет запущен при каждом запросе в development или перед первым запросом в production.

* `add_builtin_route`: Если приложение запускается в среде development, то в маршруты приложения будет добавлен маршрут для `rails/info/properties`. Этот маршрут предоставляет подробную информацию, такую как версию Rails и Ruby для `public/index.html` в приложении Rails по умолчанию.

* `build_middleware_stack`: Создает стек промежуточных программ для приложения, возвращает объект, у которого есть метод `call`, принимающий объект среды Rack для запроса.

* `eager_load!`: Если `config.eager_load` `true`, запускает хуки `config.before_eager_load`, а затем вызывает `eager_load!`, загружающий все `config.eager_load_namespaces`.

* `finisher_hook`: Представляет хук после завершения процесса инициализации приложения, а также запускает все блоки `config.after_initialize` для приложения, railties и engine-ов.

* `set_routes_reloader`: Конфигурирует Action Dispatch, перезагружая файл маршрутов с использованием `ActiveSupport::Callbacks.to_run`.

* `disable_dependency_loading`: Отключает автоматическую загрузку зависимостей, если `config.eager_load` установлена true.

Настройка пула подключений к базе данных
----------------------------------------

Соединения с базой данных Active Record управляются с помощью `ActiveRecord::ConnectionAdapters::ConnectionPool`, который обеспечивает, что пул подключений синхронизирует количество тредов, получающих доступ, с ограниченным количеством подключений к базе данных. Этот лимит по умолчанию 5, и может быть настроен в `database.yml`.

```ruby
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
```

Поскольку управление пулом подключений по умолчанию происходит внутри Active Record, все серверы приложения (Thin, Puma, Unicorn и т.д.) должны вести себя так же. В самом начале пул подключений к базе данных пуст. По мере роста запросов на дополнительные подключения, он будет создавать их, пока не достигнет ограничения на подключения.

Каждый новый запрос займет подключение, как только он впервые запросит доступ в базу данных. В конце запроса он освободит подключение. Это означает, что дополнительный слот подключения будет снова доступен для следующего запроса в очереди.

Если попытаться использовать больше соединений, чем доступно, Active Record заблокируется и подождет соединение из пула. Если он не сможет получить соединение, будет вызвана следующая ошибка тайм-аута.

```ruby
ActiveRecord::ConnectionTimeoutError - could not obtain a database connection within 5.000 seconds (waited 5.000 seconds)
```

Если вы получаете вышеприведенную ошибку, можно попытаться увеличить размер пула соединений, увеличив опцию `pool` в `database.yml`

NOTE: Если вы запускаете многотредовую среду, есть вероятность, что несколько тредов могут получить доступ к нескольким подключениям одновременно. Поэтому, в зависимости от текущей загрузки, вы можете легко получить несколько тредов, претендующих на ограниченное количество подключений.

Произвольные настройки
----------------------

Можно настроить свой собственный код с помощью конфигурационного объекта Rails с произвольными настройками или в пространстве имен `config.x`, либо непосредственно в `config`. Ключевой разницей между этими двумя вариантами является то, что необходимо использовать `config.x`, если вы определяете _вложенную_ конфигурацию (например, `config.x.nested.nested.hi`), и просто `config` для _одноуровневой_ конфигурации (например, `config.hello`).

  ```ruby
  config.x.payment_processing.schedule = :daily
  config.x.payment_processing.retries  = 3
  config.super_debugger = true
  ```

Эти конфигурационные настройки доступны с помощью конфигурационного объекта:

  ```ruby
  Rails.configuration.x.payment_processing.schedule # => :daily
  Rails.configuration.x.payment_processing.retries  # => 3
  Rails.configuration.x.payment_processing.not_set  # => nil
  Rails.configuration.super_debugger                # => true
  ```

Также можно использовать `Rails::Application.config_for` для загрузки целых конфигурационных файлов:

  ```ruby
  # config/payment.yml:
  production:
    environment: production
    merchant_id: production_merchant_id
    public_key:  production_public_key
    private_key: production_private_key
  development:
    environment: sandbox
    merchant_id: development_merchant_id
    public_key:  development_public_key
    private_key: development_private_key

  # config/application.rb
  module MyApp
    class Application < Rails::Application
      config.payment = config_for(:payment)
    end
  end
  ```

  ```ruby
  Rails.configuration.payment['merchant_id'] # => production_merchant_id or development_merchant_id
  ```

Индексирование поисковыми движками
----------------------------------

Иногда вы можете захотеть, чтобы некоторые страницы вашего приложения не были видимыми для поисковых сайтов, таких как Google, Bing, Yahoo или Duck Duck Go. Роботы, которые индексируют для этих сайтов, сначала анализируют файл `http://your-site.com/robots.txt`, который знает, какие страницы доступны для индексации.

Rails создает этот файл для вас внутри папки `/public`. По умолчанию все страницы вашего приложения доступны для индексации поисковыми движками. Если бы хотите запретить индексировать все страницы вашего приложения, используйте следующее:

```
User-agent: *
Disallow: /
```

Чтобы запретить только определенные страницы, необходимо использовать более сложный синтаксис. Изучите его в [официальной документации](http://www.robotstxt.org/robotstxt.html).

Наблюдение событийной файловой системы
--------------------------------------

Если загружен [гем listen](https://github.com/guard/listen), Rails использует наблюдение событийной файловой системы для обнаружения изменений, когда `config.cache_classes` равен `false`:

```ruby
group :development do
  gem 'listen', '>= 3.0.5', '< 3.2'
end
```

В противном случае, в каждом запросе Rails проходит по дереву приложения для проверки, не было ли что-то изменено.

Для Linux и macOS не нужны дополнительные гемы, но требуются [для *BSD](https://github.com/guard/listen#on-bsd) и
[для Windows](https://github.com/guard/listen#on-windows).

Отметьте, что [некоторые настройки не поддерживаются](https://github.com/guard/listen#issues--limitations).
