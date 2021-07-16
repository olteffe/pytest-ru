:orphan:

.. _`pytest helpers`:

Pytest API и встроенные фикстуры
================================================


Большая часть информации на этой странице перенесена в :ref:`api-reference`.

Для получения информации о подключаемых модулях и объектах см. :ref:`plugins`.

Для получения информации о механиках ``pytest.mark`` см. :ref:`mark`.

Для получения информации о фикстурах см. :ref:`fixtures`. Чтобы просмотреть полный список доступных
фикстур (добавьте ``-v``, чтобы также увидеть фикстуры с ведущим ``_``), введите :

.. code-block:: pytest

    $ pytest -q --fixtures
    cache
        Возвращает объект кеша, который может сохранять состояние между сеансами тестирования.

        cache.get(key, default)
        cache.set(key, value)

        Ключи должны быть строками, разделенными ``/``, где первая часть обычно является
        имя вашего плагина или приложения, чтобы избежать конфликта с другими пользователями кэша.

        Значениями могут быть любые объекты, обрабатываемые модулем json stdlib.

    capsys
        Включает захват текста при записи в ``sys.stdout`` и ``sys.stderr``.

        Захваченный вывод доступен через вызовы метода ``capsys.readouterr()``, которые возвращают именованный кортеж ``(out, err)``.
        ``out`` и ``err`` будут объектами ``text``.

    capsysbinary
        Включает перехват байтов при записи в ``sys.stdout`` и ``sys.stderr``.

        Захваченный вывод доступен через вызовы метода ``capsysbinary.readouterr()``,
        который возвращают именованный кортеж ``(out, err)``.
        ``out`` и ``err`` будут объектами ``bytes``.

    capfd
        Включает захват текста при записи в файловые дескрипторы ``1`` и ``2``.

        Захваченный вывод доступен через вызовы метода ``capfd.readouterr()``, которые возвращают именованный кортеж ``(out, err)``.
        ``out`` и ``err`` будут объектами ``text``.

    capfdbinary
        Включает перехват байтов при записи в файловые дескрипторы ``1`` и ``2``.

        Захваченный вывод доступен через вызовы метода ``capfd.readouterr()``, которые возвращают именованный кортеж ``(out, err)``.
        ``out`` и ``err`` будут объектами ``byte``.

    doctest_namespace [session scope]
        Фикстура, возвращающая :py:class:`dict`, который будет введен в
        пространство имен doctests.

    pytestconfig [session scope]
        Фикстура с привязкой к сеансу, которая возвращает объект :class:`_pytest.config.Config`.

        Например::

            def test_foo(pytestconfig):
                if pytestconfig.getoption("verbose") > 0:
                    ...

    record_property
        Добавляет дополнительные свойства к вызывающему тесту.

        Пользовательские свойства становятся частью отчета теста и доступны для
        настроенных ответчиков, таких как JUnit XML.

        Фикстура вызывается с помощью ``name, value``. Значение автоматически
        XML-кодируется.

        Например::

            def test_function(record_property):
                record_property("example_key", 1)

    record_xml_attribute
        Добавление дополнительных атрибутов xml в тег для вызывающего теста.

        Фикстура вызывается с помощью ``name, value``. Значение автоматически
        XML-кодируется.

    record_testsuite_property [session scope]
        Запишите новый тег ``<property>`` как дочерний тег корневого ``<testsuite>``.

        Это подходит для записи глобальной информации обо всем тестовом кейсе
        и совместимо с семейством ``xunit2`` JUnit.

        Это фикстура области ``session``, которая вызывается с помощью ``(name, value)``. Пример:

        .. code-block:: python

            def test_foo(record_testsuite_property):
                record_testsuite_property("ARCH", "PPC")
                record_testsuite_property("STORAGE_TYPE", "CEPH")

        ``name`` должна быть строкой, ``value`` будет преобразовано в строку и правильно экранировано xml.

        .. warning::

            В настоящее время эта фикстура **не** работает с плагином
            `pytest-xdist <https://github.com/pytest-dev/pytest-xdist>`__ . См. issue
            `#7767 <https://github.com/pytest-dev/pytest/issues/7767>`__ для подробностей.

    caplog
        Доступ и контроль регистрации журнала.

        Захваченные журналы доступны с помощью следующих методов свойств::

        * caplog.messages        -> список сообщений журнала с интерполяцией формата
        * caplog.text            -> строка, содержащая форматированный вывод журнала
        * caplog.records         -> список экземпляров logging.LogRecord
        * caplog.record_tuples   -> список кортежей (logger_name, level, message)
        * caplog.clear()         -> очистить захваченные записи и отформатированную строку вывода журнала

    monkeypatch
        Удобная фикстура для monkey-исправления.

        Фикстура предоставляет следующие методы для изменения объектов, словарей или
        os.environ::

            monkeypatch.setattr(obj, name, value, raising=True)
            monkeypatch.delattr(obj, name, raising=True)
            monkeypatch.setitem(mapping, name, value)
            monkeypatch.delitem(obj, name, raising=True)
            monkeypatch.setenv(name, value, prepend=False)
            monkeypatch.delenv(name, raising=True)
            monkeypatch.syspath_prepend(path)
            monkeypatch.chdir(path)

        Все изменения будут отменены после завершения работы запрашивающей испытательной функции или
        при завершения фикстуры. Параметр ``raising`` определяет, будет ли выдан KeyError
        или AttributeError, если для операции установки/удаления нет цели.

    recwarn
        Возвращает экземпляр :class:`WarningsRecorder`, который записывает все предупреждения, выдаваемые тестовыми функциями.

        См. http://docs.python.org/library/warnings.html для получения информации
        о категориях предупреждений.

    tmpdir_factory [session scope]
        Возвращает экземпляр :class:`pytest.TempdirFactory` для тестовой сессии.

    tmp_path_factory [session scope]
        Возвращает экземпляр :class:`pytest.TempPathFactory` для тестовой сессии.

    tmpdir
        Возвращает объект пути к временному каталогу, уникальный для каждого вызова тестовой
        функции, созданный как подкаталог базового временного каталога.

        По умолчанию на каждый сеанс тестирования создается новый базовый временный каталог,
        а старые базы удаляются через 3 сессии, чтобы облегчить отладку. Если
        ``--basetemp`` используется, то он очищается после каждой сессии. См. :ref:`base temporary directory`.

        Возвращаемый объект - это объект `py.path.local`_.

        .. _`py.path.local`: https://py.readthedocs.io/en/latest/path.html

    tmp_path
        Возвращает объект пути к временному каталогу, уникальный для каждого тестовой
        функции, созданный как подкаталог базового временного каталога.

        По умолчанию, на каждый сеанс тестирования создается новый базовый временный каталог,
        а старые базовые удаляются через 3 сессии, чтобы облегчить отладку. Если
        ``--basetemp`` используется, то он очищается после каждой сессии. :ref:`base temporary directory`.

        Возвращаемый объект - это объект :class:`pathlib.Path`.


    no tests ran in 0.12s

Вы также можете интерактивно вызвать справку, например, набрав в консоли Python что-нибудь на подобие:

.. code-block:: python

    import pytest

    help(pytest)
