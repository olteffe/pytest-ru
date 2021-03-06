Конфигурация
=============

Опции командной строки и настройки конфигурационного файла
-----------------------------------------------------------------

Вы можете получить справку по параметрам и значениям командной строки в файлах конфигурации INI,
используя стандартный аргумент командной строки для справки:

.. code-block:: bash

    pytest -h   # напечатает параметры и настройки файла конфигурации

Будут отображены опции командной строки и конфигурационных файлов, которые зарегистрированы
установленными плагинами.

.. _`config file formats`:

Форматы файлов конфигурации
----------------------------

Многие настройки pytest :ref:`pytest settings <ini options ref>` могут быть заданы в *конфигурационном файле*, который
по традиции находится в корне вашего репозитория или в вашей папке tests.

Небольшие примеры файлов конфигурации, поддерживаемых pytest:

pytest.ini
~~~~~~~~~~

Файлы``pytest.ini`` имеют приоритет над другими файлами, даже если они пусты.

.. code-block:: ini

    # pytest.ini
    [pytest]
    minversion = 6.0
    addopts = -ra -q
    testpaths =
        tests
        integration


pyproject.toml
~~~~~~~~~~~~~~

.. versionadded:: 6.0

``pyproject.toml`` рассматривается для конфигурации, когда он содержит секцию ``tool.pytest.ini_options``.

.. code-block:: toml

    # pyproject.toml
    [tool.pytest.ini_options]
    minversion = "6.0"
    addopts = "-ra -q"
    testpaths = [
        "tests",
        "integration",
    ]

.. note::

    Может возникнуть вопрос, почему ``[tool.pytest.ini_options]``, а не ``[tool.pytest]``, как в случае
    с другими инструментами.

    Причина в том, что команда pytest намерена полностью использовать богатый функционал данных TOML
    для конфигурации в будущем, зарезервировав для этого секцию ``[tool.pytest]``.
    Секция ``ini_options`` пока используется в качестве связки между существующими конфигурациями в формате
    ``.ini`` и будущим форматом конфигурации.

tox.ini
~~~~~~~

Файлы ``tox.ini`` являются конфигурационными файлами проекта ``tox <https://tox.readthedocs.io>`__,
и могут также использоваться для хранения конфигурации pytest, если в них есть секция ``[pytest]``.

.. code-block:: ini

    # tox.ini
    [pytest]
    minversion = 6.0
    addopts = -ra -q
    testpaths =
        tests
        integration


setup.cfg
~~~~~~~~~

Файлы ``setup.cfg`` являются конфигурационными файлами общего назначения, первоначально использовались в
``distutils <https://docs.python.org/3/distutils/configfile.html>`__, и могут также использоваться для
хранения конфигурации pytest, если в них есть секция ``[tool:pytest]``.

.. code-block:: ini

    # setup.cfg
    [tool:pytest]
    minversion = 6.0
    addopts = -ra -q
    testpaths =
        tests
        integration

.. warning::

    Использование ``setup.cfg`` не рекомендуется, за исключением очень простых случаев. Файлы ``.cfg``
    используют парсер, отличный от ``pytest.ini`` и ``tox.ini``, что может привести к трудно отслеживаемым проблемам.
    По возможности, рекомендуется использовать последние или ``pyproject.toml`` для хранения вашей
    конфигурации pytest.


.. _rootdir:
.. _configfiles:

Инициализация: определение корневой директории и файла инициализации
-----------------------------------------------------------------------

pytest определяет корневой каталог ``rootdir`` для каждого запуска теста, который зависит от
аргументов командной строки (заданных тестовых файлов, путей) и от
существования конфигурационных файлов.  Определенные ``rootdir`` и ``configfile``
печатаются как часть заголовка pytest при запуске.

Вот краткое описание того, для чего ``pytest`` использует ``rootdir``:

* Конструировать *nodeids* во время сборки; каждому тесту присваивается
  уникальный *nodeid*, который располагается в корне ``rootdir`` и учитывает
  полный путь, имя класса, имя функции и параметризацию (если указано).

* Используется плагинами как стабильное место для хранения специфической информации о проекте/запуске теста;
  например, внутренний плагин :ref:`cache <cache>` создает поддиректорию ``.pytest_cache`` в ``rootdir`` для хранения состояния кросс-тестового запуска.
  в ``rootdir`` для сохранения состояния выполнения кросс-теста.

``rootdir`` **НЕ** используется для изменения ``sys.path``/ ``PYTHONPATH`` или
влияет на то, как импортируются модули. Более подробную информацию смотрите в :ref:`pythonpath`.

Опция командной строки ``--rootdir=path`` может быть использована для принудительной установки определенного каталога.
Обратите внимание, что в отличие от других опций командной строки, ``--rootdir`` нельзя использовать с опцией
:confval:`addopts` внутри ``pytest.ini``, поскольку ``rootdir`` уже используется для *поиска* ``pytest.ini``.

Алгоритм нахождения ``rootdir``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вот алгоритм поиска корневого каталога из ``args``:

- определяем общий родительский каталог для переданных аргументов, которые распознаны в качестве
  существующих путей файловой системы.
  Если таких аргументов-путей нет, то текущей рабочей директорией становится
  общий родительский каталог;

- ищем файлы ``pytest.ini``, ``pyproject.toml``, ``tox.ini``, и ``setup.cfg`` в каталоге-предке
  и выше. Если находим, он становится ``configfile``, а его каталог становится ``rootdir``.

- Если конфигурационный файл не найден, ищем ``setup.py`` вверх от общего
  каталога-предка, чтобы определить ``rootdir``.

- если файл ``setup.py`` не найден, ищем файлы ``pytest.ini``, ``pyproject.toml``, ``tox.ini``, и
  ``setup.cfg`` в каждом указанном аргументе ``args`` и выше. Если нашли -
  он становится ``configfile``, а его директория становится корневой.

- если вообще никаких ``configfile`` не найдено, то в качестве ``rootdir``
  используется уже определенный общий каталог-предок. Это позволяет использовать
  ``pytest`` в стуктурах, которые не являются частью пакета и не имеют
  никакой конкретной конфигурации.

Если никакие ``args`` не указаны, ``pytest`` начинает определение
``rootdir`` из текущего рабочего каталога и оттуда же собирает тесты.

Файлы будут сопоставлены как конфигурационные, только если:

* ``pytest.ini``: всегда будет иметь приоритет, даже если будет пуст.
* ``pyproject.toml``: содержит секцию ``[tool.pytest.ini_options]``.
* ``tox.ini``: содержит секцию  ``[pytest]``.
* ``setup.cfg``: содержит секцию ``[tool:pytest]``.

Файлы рассматриваются в указанном выше порядке. Опции из нескольких кандидатов для ``configfiles''
никогда не объединяются - побеждает первый вариант.

Внутренний объект :class:`Config <_pytest.config.Config>` (доступный через хуки или через фикстуру :fixture:`pytestconfig`)
впоследствии будет содержать эти атрибуты:

- :attr:`config.rootpath <_pytest.config.Config.rootpath>`: определенный корневой каталог, который гарантированно существует.

- :attr:`config.inipath <_pytest.config.Config.inipath>`: определенный ``configfile``, может быть ``None``
  (он назван ``inipath`` по историческим причинам).

.. versionadded:: 6.1
    Добавлены свойства ``config.rootpath`` и ``config.inipath``. Они являются версиями :class:`pathlib.Path`
    старых ``config.rootdir`` и ``config.inifile``, которые имеют тип ``py.path.local``, и все еще
    существуют для обратной совместимости.

Корневой каталог ``rootdir`` используется в качестве справочного каталога для построения тестовых
адресов ("nodeids") и может также использоваться плагинами для хранения информации о каждом запуске теста.

Пример:

.. code-block:: bash

    pytest path/to/testdir path/other/

определит общего предка как ``path`` и затем
проверит наличие конфигурационных файлов следующим образом:

.. code-block:: text

    # сначала ищем файлы pytest.ini
    path/pytest.ini
    path/pyproject.toml  # должен содержать секцию [tool.pytest.ini_options] для соответствия
    path/tox.ini         # должен содержать секцию  [pytest] для соответствия
    path/setup.cfg       # должен содержать секцию [tool:pytest] для соответствия
    pytest.ini
    ... # все пути вплоть до корня

    # теперь ищем setup.py
    path/setup.py
    setup.py
    ... # все пути вплоть до корня


.. warning::

    Аргументы командной строки пользовательского плагина pytest могут включать путь, как в примере
    ``pytest --log-output ../../test.log args``. Тогда ``args`` является обязательным,
    в противном случае pytest использует папку test.log для определения ``rootdir``.
    (см. также `выпуск 1435 <https://github.com/pytest-dev/pytest/issues/1435>`_).
    Также возможна точка ``.`` для ссылки на текущий рабочий каталог.


.. _`how to change command line options defaults`:
.. _`adding default options`:


Параметры встроенного файла конфигурации
----------------------------------------------

Полный список опций см. :ref:`reference documentation <ini options ref>`.
