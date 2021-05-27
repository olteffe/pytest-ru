.. _monkeypatching:

Использование monkeypatch/mock модули и окружение
================================================================

.. currentmodule:: _pytest.monkeypatch

Иногда тесты должны вызывать функциональность, которая зависит от глобальных настроек или которая вызывает
код, который нелегко протестировать, например доступ к сети. Фикстура ``monkeypatch`` помогает вам
безопасно установить/удалить атрибут, элемент словаря или переменной среды или изменить ``sys.path``
для импорта.

Фикстура ``monkeypatch`` предоставляет эти вспомогательные методы для безопасного исправления и имитации
функциональности в тестах:

.. code-block:: python

    monkeypatch.setattr(obj, name, value, raising=True)
    monkeypatch.setattr("somemodule.obj.name", value, raising=True)
    monkeypatch.delattr(obj, name, raising=True)
    monkeypatch.setitem(mapping, name, value)
    monkeypatch.delitem(obj, name, raising=True)
    monkeypatch.setenv(name, value, prepend=False)
    monkeypatch.delenv(name, raising=True)
    monkeypatch.syspath_prepend(path)
    monkeypatch.chdir(path)

Все изменения будут отменены после завершения запрашивающей тестовой функции или фикстуры. Параметр
``raising`` определяет, будет ли возникать ошибка ``KeyError`` или ``AttributeError``, если цель операции
set/deletion не существует.

Рассмотрим следующие сценарии:

1. Изменение поведения функции или свойства класса для теста, например: есть вызов API или соединение с
базой данных, которое вы не сделаете для теста, но вы знаете, каким должен быть ожидаемый результат.
Примените :py:meth:`monkeypatch.setattr <MonkeyPatch.setattr>`, чтобы изменить функцию или свойство с
желаемым поведением тестирования. Это может включать ваши собственные функции.
Примените :py:meth:`monkeypatch.delattr <MonkeyPatch.delattr>`, чтобы удалить функцию или свойство для тестирования.

2. Изменение значений словарей: например у вас есть глобальная конфигурация, которую вы хотите изменить
для определенных тестовых кейсов. Используйте :py:meth:`monkeypatch.setitem <MonkeyPatch.setitem>` чтобы изменить
словарь для теста. Можно использовать :py:meth:`monkeypatch.delitem <MonkeyPatch.delitem>` для удаления.

3. Изменение переменных среды для теста, например для проверки поведения программы, если переменная окружения
отсутствует, или установить несколько значений известной переменной.
Можно использовать :py:meth:`monkeypatch.setenv <MonkeyPatch.setenv>` и :py:meth:`monkeypatch.delenv <MonkeyPatch.delenv>`
для этих патчей.

4. Использование ``monkeypatch.setenv("PATH", value, prepend=os.pathsep)`` для изменения ``$PATH``, и
:py:meth:`monkeypatch.chdir <MonkeyPatch.chdir>` для изменения контекста текущего рабочего каталога во время теста.

5. Использовать :py:meth:`monkeypatch.syspath_prepend <MonkeyPatch.syspath_prepend>` для изменения ``sys.path``
который также будет вызывать ``pkg_resources.fixup_namespace_packages`` и :py:func:`importlib.invalidate_caches`.

См. `monkeypatch blog post`_ для вводного материала и обсуждения.

.. _`monkeypatch blog post`: http://tetamap.wordpress.com/2009/03/03/monkeypatching-in-unit-tests-done-right/

Простой пример: monkeypatching функции
----------------------------------------

Рассмотрим сценарий, в котором вы работаете с пользовательскими каталогами. В контексте тестирования вы
не хотите, чтобы ваш тест зависел от текущего пользователя. ``monkeypatch`` можно использовать для
исправления функций, зависящих от пользователя, чтобы всегда возвращать определенное значение.

В этом примере, :py:meth:`monkeypatch.setattr <MonkeyPatch.setattr>` используется для исправления ``Path.home``
так что известный путь тестирования ``Path("/abc")`` всегда используется при запуске теста.
Это устраняет любую зависимость от запущенного пользователя в целях тестирования.
:py:meth:`monkeypatch.setattr <MonkeyPatch.setattr>` должен быть вызван перед вызовом функции, которая
будет использовать исправленную функцию.
После завершения тестовой функции модификация ``Path.home`` будет отменена.

.. code-block:: python

    # листинг test_module.py с исходным кодом и тестом
    from pathlib import Path


    def getssh():
        """Простая функция для возврата расширенного пути ssh homedir."""
        return Path.home() / ".ssh"


    def test_getssh(monkeypatch):
        # mocked-возвращающая функция для замены Path.home
        # всегда возвращает '/abc'
        def mockreturn():
            return Path("/abc")

        # Приложение monkeypatch для замены Path.home
        # с поведением mockreturn, определенным выше.
        monkeypatch.setattr(Path, "home", mockreturn)

        # Вызов getssh() будет использовать mockreturn вместо Path.home
        # для этого теста с monkeypatch.
        x = getssh()
        assert x == Path("/abc/.ssh")

Monkeypatch-возвращенные объекты: создание mock-классов
----------------------------------------------------------

:py:meth:`monkeypatch.setattr <MonkeyPatch.setattr>` может использоваться вместе с классами для имитации
возвращаемых объектов из функций вместо значений.
Представьте себе простую функцию, которая принимает URL-адрес API и возвращает ответ json.

.. code-block:: python

    # листинг app.py, простой пример получения API
    import requests


    def get_json(url):
        """Принимает URL-адрес и возвращает JSON."""
        r = requests.get(url)
        return r.json()

Нам нужно имитировать(mock) ``r``, возвращающий объект ответа для целей тестирования.
Имитации ``r`` необходим метод ``.json()``, который возвращает словарь.
Это можно сделать в нашем тестовом файле, определив класс для представления ``r``.

.. code-block:: python

    # листинг test_app.py, простой тест для получения нашего API
    # запросы на импорт для целей monkeypatching
    import requests

    # наш app.py, который включен в функцию get_json()
    # это предыдущий пример блока кода
    import app

    # пользовательский класс, который будет mock-возвращаемым значением, переопределит запросы
    # requests.Response, возвращенные из requests.get
    class MockResponse:

        # mock-метод json() всегда возвращает конкретный тестовый словарь
        @staticmethod
        def json():
            return {"mock_key": "mock_response"}


    def test_get_json(monkeypatch):

        # Могут быть переданы любые аргументы, и mock_get () всегда будет возвращать наш mock
        # объект, который имеет только метод .json ().
        def mock_get(*args, **kwargs):
            return MockResponse()

        # применение monkeypatch для requests.get к mock_get
        monkeypatch.setattr(requests, "get", mock_get)

        # app.get_json, который содержит requests.get, используя monkeypatch
        result = app.get_json("https://fakeurl")
        assert result["mock_key"] == "mock_response"


``monkeypatch`` применяет mock для ``requests.get`` с нашей функцией ``mock_get``.
Функция ``mock_get`` возвращает экземпляр класса ``MockResponse``, у которого
есть метод ``json()``, определенный для возврата известного тестового словаря и не требует подключения к
внешнему API.

Вы можете создать класс ``MockResponse``, с соответствующей степенью сложности для тестируемого
вами сценария. Или, например, он может включать свойство ``ok``, которое всегда возвращает ``True``,
или вернуть разные значения из mock-метода ``json()`` на основе входных строк.

Этот mock можно использовать в разных тестах с помощью ``фикстуры``:

.. code-block:: python

    # листинг test_app.py, простой тест для получения нашего API
    import pytest
    import requests

    # app.py который включает функцию get_json ()
    import app

    # настраиваемый класс, который будет фиктивным(mock) возвращаемым значением requests.get()
    class MockResponse:
        @staticmethod
        def json():
            return {"mock_key": "mock_response"}


    # monkeypatch requests.get перемещен в фикстуру
    @pytest.fixture
    def mock_response(monkeypatch):
        """Requests.get() имитирует для возврата {'mock_key':'mock_response'}."""

        def mock_get(*args, **kwargs):
            return MockResponse()

        monkeypatch.setattr(requests, "get", mock_get)


    # обратите внимание, что наш тест использует настраиваемую фикстуру вместо monkeypatch напрямую
    def test_get_json(mock_response):
        result = app.get_json("https://fakeurl")
        assert result["mock_key"] == "mock_response"


Кроме того, если mock был разработан для применения во всех тестах, то ``фикстуру`` можно было бы
переместить в файл ``conftest.py`` и использовать параметр ``autouse=True``.


Пример глобального патча: предотвращение "запросов" от дистанционных операций
-------------------------------------------------------------------------------

Если вы хотите, чтобы библиотека "запросов" не выполняла HTTP-запросы во всех ваших тестах, вы
можете сделать:

.. code-block:: python

    # листинг conftest.py
    import pytest


    @pytest.fixture(autouse=True)
    def no_requests(monkeypatch):
        """Удаление requests.sessions.Session.request для всех тестов."""
        monkeypatch.delattr("requests.sessions.Session.request")

Эта autouse-фикстура будет выполняться для каждой тестовой функции, и она удалит метод
``request.session.Session.request``, так что любые попытки в рамках тестов создать HTTP-запросы
потерпят неудачу.


.. note::

    Имейте в виду, что не рекомендуется изменять встроенные функции, такие как ``open``,
    ``compile``, и т.д., потому что это может сломать pytest. Если это необходимо, может помочь
    ``--tb=native``, ``--assert=plain`` и ``--capture=no``, хотя нет никаких гарантий.

.. note::

    Имейте в виду, что исправление функции ``stdlib`` и некоторые сторонние библиотеки, используемые
    pytest, могут сломать сам pytest, поэтому в этих случаях рекомендуется использовать
    :meth:`MonkeyPatch.context` чтобы ограничить исправление блоком, который вы хотите протестировать:

    .. code-block:: python

        import functools


        def test_partial(monkeypatch):
            with monkeypatch.context() as m:
                m.setattr(functools, "partial", 3)
                assert functools.partial == 3

    См. Проблему `#3290 <https://github.com/pytest-dev/pytest/issues/3290>`_ для подробностей.


Переменные среды Monkeypatching
------------------------------------

Если вы работаете с переменными среды, вам часто необходимо безопасно изменить значения или удалить их
из системы в целях тестирования. ``monkeypatch`` предоставляет механизм для этого, используя методы
``setenv`` и ``delenv``. Наш пример кода для тестирования:

.. code-block:: python

    # листинг исходного файла кода, например code.py
    import os


    def get_os_user_lower():
        """Простая функция получения.
        Возвращает строчный USER или исключение OSError."""
        username = os.getenv("USER")

        if username is None:
            raise OSError("USER environment is not set.")

        return username.lower()

There are two potential paths. First, the ``USER`` environment variable is set to a
value. Second, the ``USER`` environment variable does not exist. Using ``monkeypatch``
both paths can be safely tested without impacting the running environment:

Есть два возможных пути. Во-первых, переменная окружения ``USER`` устанавливается в значение.
Во-вторых, переменная окружения ``USER`` не существует. Используя ``monkeypatch``
можно безопасно проверить оба пути, не затрагивая рабочее окружение.

.. code-block:: python

    # содержимое нашего тестового файла, например test_code.py
    import pytest


    def test_upper_to_lower(monkeypatch):
        """Установите переменную USER env var для утверждения поведения."""
        monkeypatch.setenv("USER", "TestingUser")
        assert get_os_user_lower() == "testinguser"


    def test_raise_exception(monkeypatch):
        """Удалите переменную USER env и убедитесь, что возникла ошибка OSError."""
        monkeypatch.delenv("USER", raising=False)

        with pytest.raises(OSError):
            _ = get_os_user_lower()

Это поведение может быть перенесено в структуры ``фикстуры`` и совместно использоваться в тестах:

.. code-block:: python

    # содержимое нашего тестового файла, например test_code.py
    import pytest


    @pytest.fixture
    def mock_env_user(monkeypatch):
        monkeypatch.setenv("USER", "TestingUser")


    @pytest.fixture
    def mock_env_missing(monkeypatch):
        monkeypatch.delenv("USER", raising=False)


    # обратите внимание, что тесты ссылаются на фикстуры для mock
    def test_upper_to_lower(mock_env_user):
        assert get_os_user_lower() == "testinguser"


    def test_raise_exception(mock_env_missing):
        with pytest.raises(OSError):
            _ = get_os_user_lower()


Словари Monkeypatching
---------------------------

:py:meth:`monkeypatch.setitem <MonkeyPatch.setitem>` можно использовать для безопасной установки
значений словарей на определенные значения во время тестов. Например этот упрощенный пример:

.. code-block:: python

    # листинг app.py для создания простой строки подключения
    DEFAULT_CONFIG = {"user": "user1", "database": "db1"}


    def create_connection_string(config=None):
        """Создает строку подключения из ввода или значений по умолчанию."""
        config = config or DEFAULT_CONFIG
        return f"User Id={config['user']}; Location={config['database']};"

В целях тестирования мы можем исправить словарь ``DEFAULT_CONFIG`` на определенные значения.

.. code-block:: python

    # листинг test_app.py
    # app.py с функцией строки подключения (предыдущий блок кода)
    import app


    def test_connection(monkeypatch):

        # Исправление значения DEFAULT_CONFIG к конкретным
        # значениям тестирования только для этого теста.
        monkeypatch.setitem(app.DEFAULT_CONFIG, "user", "test_user")
        monkeypatch.setitem(app.DEFAULT_CONFIG, "database", "test_db")

        # ожидаемый результат на основе mock
        expected = "User Id=test_user; Location=test_db;"

        # тест использует настройки словаря monkeypatched
        result = app.create_connection_string()
        assert result == expected

Вы можете использовать :py:meth:`monkeypatch.delitem <MonkeyPatch.delitem>` для удаления значения.

.. code-block:: python

    # листинг test_app.py
    import pytest

    # app.py с функцией строки подключения
    import app


    def test_missing_user(monkeypatch):

        # исправление DEFAULT_CONFIG, чтобы не пропустить ключ 'user'
        monkeypatch.delitem(app.DEFAULT_CONFIG, "user", raising=False)

        # Ожидается ошибка ключа, поскольку конфигурация не передана, и
        # по умолчанию теперь отсутствует запись 'user'.
        with pytest.raises(KeyError):
            _ = app.create_connection_string()


Модульность фикстур дает вам возможность определять отдельные фикстуры для каждого
потенциального mock и ссылаться на них в необходимых тестах.

.. code-block:: python

    # листинг test_app.py
    import pytest

    # app.py с функцией строки подключения
    import app

    # все mock перемещены в отдельные фикстуры
    @pytest.fixture
    def mock_test_user(monkeypatch):
        """Установка DEFAULT_CONFIG user в test_user."""
        monkeypatch.setitem(app.DEFAULT_CONFIG, "user", "test_user")


    @pytest.fixture
    def mock_test_database(monkeypatch):
        """Установка DEFAULT_CONFIG database в test_db."""
        monkeypatch.setitem(app.DEFAULT_CONFIG, "database", "test_db")


    @pytest.fixture
    def mock_missing_default_user(monkeypatch):
        """Удаление ключа user из DEFAULT_CONFIG"""
        monkeypatch.delitem(app.DEFAULT_CONFIG, "user", raising=False)


    # тесты ссылаются только на необходимые mock фикстуры
    def test_connection(mock_test_user, mock_test_database):

        expected = "User Id=test_user; Location=test_db;"

        result = app.create_connection_string()
        assert result == expected


    def test_missing_user(mock_missing_default_user):

        with pytest.raises(KeyError):
            _ = app.create_connection_string()


.. currentmodule:: _pytest.monkeypatch

Справка по API
-----------------

Обратитесь к документации по классу :class:`MonkeyPatch`.
