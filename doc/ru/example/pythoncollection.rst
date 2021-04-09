Изменение стандартных правил поиска тестов Python
===================================================

Игнорирование путей при поиске тестов
-------------------------------------------

Возможно игнорировать определенные тестовые каталоги и модули во время запуска, передав
``--ignore=path`` в командной строке. ``pytest`` позволяет использовать несколько параметров
``--ignore``. Пример:

.. code-block:: text

    tests/
    |-- example
    |   |-- test_example_01.py
    |   |-- test_example_02.py
    |   '-- test_example_03.py
    |-- foobar
    |   |-- test_foobar_01.py
    |   |-- test_foobar_02.py
    |   '-- test_foobar_03.py
    '-- hello
        '-- world
            |-- test_world_01.py
            |-- test_world_02.py
            '-- test_world_03.py

Если вызвать ``pytest`` с ``--ignore=tests/foobar/test_foobar_03.py --ignore=tests/hello/``,
то ``pytest`` выберет только те тестовые модули, которые не соответствуют указанным шаблонам:

.. code-block:: pytest

    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-5.x.y, py-1.x.y, pluggy-0.x.y
    rootdir: $REGENDOC_TMPDIR, inifile:
    collected 5 items

    tests/example/test_example_01.py .                                   [ 20%]
    tests/example/test_example_02.py .                                   [ 40%]
    tests/example/test_example_03.py .                                   [ 60%]
    tests/foobar/test_foobar_01.py .                                     [ 80%]
    tests/foobar/test_foobar_02.py .                                     [100%]

    ========================= 5 passed in 0.02 seconds =========================

Опция ``--ignore-glob`` позволяет игнорировать пути в виде шаблонов Unix. Если
хотите исключить тестовые модули, которые заканчиваются на ``_01.py``,
можно запустить ``pytest`` с опцией ``--ignore-glob='*_01.py'``.

Отмена выбора тестов во время запуска
---------------------------------------

Во время сборки тестов можно по отдельности отменить выбор некоторых, передав параметр
``--deselect=item``. Например, ``tests/foobar/test_foobar_01.py`` содержит ``test_a`` и
``test_b``. Возможно запустить все тесты в ``tests/`` *за исключением* ``tests/foobar/test_foobar_01.py::test_a``,
запуская ``pytest`` с ``--deselect tests/foobar/test_foobar_01.py::test_a``.
``pytest`` поддерживает несколько параметров ``--deselect``.

Использование повторяющихся путей, указанных в командной строке
--------------------------------------------------------------

По умолчанию ``pytest`` игнорирует несколько путей, указанные в командной строке.
Например:

.. code-block:: pytest

    pytest path_a path_a

    ...
    collected 1 item
    ...

Будет собран только один элемент.

Для сборки двойных тестов, используйте ``--keep-duplicates`` в командной строке.
Пример:

.. code-block:: pytest

    pytest --keep-duplicates path_a path_a

    ...
    collected 2 items
    ...

Поскольку сборщик работает только с каталогами, если вы укажете дважды один тестовый
файл, ``pytest`` все равно соберет его дважды, даже если опция ``--keep-duplicates``
не применялась.
Пример:

.. code-block:: pytest

    pytest test_a.py test_a.py

    ...
    collected 2 items
    ...


Изменение правил рекурсивного обхода
-----------------------------------------------------

Опцию :confval:`norecursedirs` можно задавать в ini-файле, например, в ``pytest.ini``
корневого каталога проекта:

.. code-block:: ini

    # content of pytest.ini
    [pytest]
    norecursedirs = .svn _build tmp*

Такая запись указывает ``pytest`` не проводить рекурсивный поиск тестов в каталоге ``.svn``
subversion, в "build"-директориях "sphinx" и в любых каталогах с префиксом ``tmp``.

.. _`change naming conventions`:

Изменение соглашений об именах тестов для поиска
-----------------------------------------------------

Вы можете настроить свои правила для поиска тестов по имени, установив опции
:confval:`python_files`, :confval:`python_classes` и
:confval:`python_functions` в файле конфигурации :ref:`configuration file <config file formats>`.
Вот пример:

.. code-block:: ini

    # листинг pytest.ini
    # Пример 1: pytest ищет "check" вместо "test"
    [pytest]
    python_files = check_*.py
    python_classes = Check
    python_functions = *_check

Такая настройка заставит ``pytest`` искать тесты в файлах по глобальному шаблону
``check_*.py``, префикс ``Check`` в классах, а функции и методы по шаблону
``*_check``. Например:

.. code-block:: python

    # листинг check_myapp.py
    class CheckMyApp:
        def simple_check(self):
            pass

        def complex_check(self):
            pass

Набор тестов будет выглядеть так:

.. code-block:: pytest

    $ pytest --collect-only
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR, configfile: pytest.ini
    collected 2 items

    <Module check_myapp.py>
      <Class CheckMyApp>
          <Function simple_check>
          <Function complex_check>

    ======================== 2 tests collected in 0.12s ========================

Вы можете использовать несколько глобальных шаблонов, добавив пробел между ними:

.. code-block:: ini

    # Пример 2: пусть pytest ищет файлы с "test" и "example"
    # листинг pytest.ini
    [pytest]
    python_files = test_*.py example_*.py

.. note::

   параметры ``python_functions`` и ``python_classes`` не оказывают никакого действия
   на поиск ``unittest.TestCase``, поскольку обнаружение таких тестов
   производится средствами ``unittest``.

Интерпретация аргументов командной строки как пакетов Python
---------------------------------------------------------------

Можно использовать опцию ``--pyargs``, чтобы ``pytest`` попытался интерпретировать
аргументы как имена пакетов ``python``, получить путь к их файловой системе и затем
запустить тест. Например, если у вас установлен ``unittest2``, вы можете выполнить команду:

.. code-block:: bash

    pytest --pyargs unittest2.test.test_skipping -q

которая запустит соответствующий тестовый модуль. Как и с другими опциями,
через ini-файл и опциями :confval:`addopts` вы можете сделать их постоянными:

.. code-block:: ini

    # листинг pytest.ini
    [pytest]
    addopts = --pyargs

Теперь простой вызов ``pytest NAME`` проверит, существует ли ``NAME`` как импортируемый
модуль пакета, и в противном случае обработает его как путь в файловой системе.

Просмотр дерева найденных тестов
-----------------------------------------------

Всегда возможно заглянуть в дерево собранных тестов, не выполняя самих тестов:

.. code-block:: pytest

    . $ pytest --collect-only pythoncollection.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR, configfile: pytest.ini
    collected 3 items

    <Module CWD/pythoncollection.py>
      <Function test_function>
      <Class TestClass>
          <Function test_method>
          <Function test_anothermethod>

    ======================== 3 tests collected in 0.12s ========================

.. _customizing-test-collection:

Настройка поиска тестов
---------------------------

.. regendoc:wipe

Можно легко указать ``pytest`` обнаруживать тесты в любом ``python``-файле:

.. code-block:: ini

    # листинг pytest.ini
    [pytest]
    python_files = *.py

Однако, во многих проектах есть файл ``setup.py``, который не хотелось бы импортировать.
Более того, там могут присутствовать файлы, которые можно импортировать только
определенной версией ``Python``. В таких случаях можно динамически определить
игнорируемые файлы, перечислив их в ``conftest.py``:

.. code-block:: python

    # листинг conftest.py
    import sys

    collect_ignore = ["setup.py"]
    if sys.version_info[0] > 2:
        collect_ignore.append("pkg/module_py2.py")

а затем, если у вас есть файл модуля, подобный этому:

.. code-block:: python

    # листинг pkg/module_py2.py
    def test_only_on_python2():
        try:
            assert 0
        except Exception, e:
            pass

и макет файла ``setup.py``:

.. code-block:: python

    # листинг setup.py
    0 / 0  # вызовет исключение, если импортировано

Тогда при запуске ``pytest`` в интерпретаторе ```Python 2`` мы соберем 1 тест,
а файл``setup.py`` будет проигнорирован:

.. code-block:: pytest

    #$ pytest --collect-only
    ====== test session starts ======
    platform linux2 -- Python 2.7.10, pytest-2.9.1, py-1.4.31, pluggy-0.3.1
    rootdir: $REGENDOC_TMPDIR, inifile: pytest.ini
    collected 1 items
    <Module 'pkg/module_py2.py'>
      <Function 'test_only_on_python2'>

    ====== 1 tests found in 0.04 seconds ======

При запустите на ``Python 3`` будут исключены:

.. code-block:: pytest

    $ pytest --collect-only
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR, configfile: pytest.ini
    collected 0 items

    ======================= no tests collected in 0.12s ========================

Для определения файлов, которые должны быть пропущены, можно также добавлять в
:globalvar:`collect_ignore_glob` шаблоны в стиле Unix.

В следующем примере в ``conftest.py`` игнорируется файл ``setup.py`` и все файлы,
которые оканчиваются на ``*_py2.py`` и запускаются с помощью ``python`` версии 3 и выше:

.. code-block:: python

    # листинг conftest.py
    import sys

    collect_ignore = ["setup.py"]
    if sys.version_info[0] > 2:
        collect_ignore_glob = ["*_py2.py"]

Начиная с Pytest 2.6, пользователи могут запретить pytest обнаруживать классы,
начинающиеся с ``Test`` установив логическое значение атрибута ``__test__`` в ``False``.

.. code-block:: python

    # Не будет обнаружен как тест
    class TestClass:
        __test__ = False
