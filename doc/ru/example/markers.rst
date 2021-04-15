
.. _`mark examples`:

Работа с пользовательскими маркерами
=================================================

Несколько примеров использования механизма :ref:`mark`.

.. _`mark run`:

Маркировка тестовых функций и отбор маркированных тестов для запуска
---------------------------------------------------------------------

Вы можете «маркировать» тестовую функцию с помощью настраиваемых метаданных:

.. code-block:: python

    # листинг test_server.py

    import pytest


    @pytest.mark.webtest
    def test_send_http():
        pass  # выполнить веб-тест для вашего приложения


    def test_something_quick():
        pass


    def test_another():
        pass


    class TestClass:
        def test_method(self):
            pass



Затем вы можете ограничить тестовый запуск только тестами, отмеченными ``webtest``:

.. code-block:: pytest

    $ pytest -v -m webtest
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 4 items / 3 deselected / 1 selected

    test_server.py::test_send_http PASSED                                [100%]

    ===================== 1 passed, 3 deselected in 0.12s ======================

Или наоборот, выполняя все тесты, кроме ``webtest``:

.. code-block:: pytest

    $ pytest -v -m "not webtest"
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 4 items / 1 deselected / 3 selected

    test_server.py::test_something_quick PASSED                          [ 33%]
    test_server.py::test_another PASSED                                  [ 66%]
    test_server.py::TestClass::test_method PASSED                        [100%]

    ===================== 3 passed, 1 deselected in 0.12s ======================

Выбор тестов на основе их идентификатора узла(node ID)
------------------------------------------------------

Вы можете передать один или несколько :ref:`node IDs <node-id>`  в качестве
позиционных аргументов для выбора только указанных тестов. Это упрощает выбор тестов
на основе их модуля, класса, метода или имени функции:

.. code-block:: pytest

    $ pytest -v test_server.py::TestClass::test_method
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 1 item

    test_server.py::TestClass::test_method PASSED                        [100%]

    ============================ 1 passed in 0.12s =============================

Вы также можете выбрать по классу:

.. code-block:: pytest

    $ pytest -v test_server.py::TestClass
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 1 item

    test_server.py::TestClass::test_method PASSED                        [100%]

    ============================ 1 passed in 0.12s =============================

Или выбрать несколько узлов:

.. code-block:: pytest

    $ pytest -v test_server.py::TestClass test_server.py::test_send_http
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 2 items

    test_server.py::TestClass::test_method PASSED                        [ 50%]
    test_server.py::test_send_http PASSED                                [100%]

    ============================ 2 passed in 0.12s =============================

.. _node-id:

.. note::

    Идентификаторы узлов имеют формат ``module.py::class::method`` или
    ``module.py::function``.  Идентификаторы узлов определяют, какие тесты собираются,
    так что ``module.py::class`` будут выбраны все тестовые методы класса.
    Для каждого параметра параметризованной фикстуры или функции
    также создаются узлы, так что идентификатор для отбора конкретного
    параметризованного теста должен включать значение параметра, например,
    ``module.py::function[param]``.

    Идентификаторы узлов упавшего теста отображаются в сводном отчете,
    если ``pytest`` запущен с опцией ``-rf``.  Вы также можете создать идентификаторы
    узлов из вывода ``pytest --collectonly``.

Использование опции ``-k "выражение"`` для отбора тестов по их именам
----------------------------------------------------------------------

.. versionadded:: 2.0/2.3.4

Опцию ``-k`` командной строки можно использовать, чтобы указать подстроку,
которая должна присутствовать в именах тестов (при использовании опции ``-m``
проверяется точное совпадение). Это облегчает отбор тестов по именам:

.. versionchanged:: 5.4

Сопоставление выражений теперь не чувствительно к регистру.

.. code-block:: pytest

    $ pytest -v -k http  # running with the above defined example module
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 4 items / 3 deselected / 1 selected

    test_server.py::test_send_http PASSED                                [100%]

    ===================== 1 passed, 3 deselected in 0.12s ======================

Можно также запустить все тесты, которые не содержат ключевого слова:

.. code-block:: pytest

    $ pytest -k "not send_http" -v
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 4 items / 1 deselected / 3 selected

    test_server.py::test_something_quick PASSED                          [ 33%]
    test_server.py::test_another PASSED                                  [ 66%]
    test_server.py::TestClass::test_method PASSED                        [100%]

    ===================== 3 passed, 1 deselected in 0.12s ======================

Или выбрать все тесты, в именах которых есть подстрока "http" или "quick":

.. code-block:: pytest

    $ pytest -k "http or quick" -v
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 4 items / 2 deselected / 2 selected

    test_server.py::test_send_http PASSED                                [ 50%]
    test_server.py::test_something_quick PASSED                          [100%]

    ===================== 2 passed, 2 deselected in 0.12s ======================

Можно использовать ``and``, ``or``, ``not`` и круглые скобки.


Помимо имени теста, ``-k`` также соответствует именам родителей теста (обычно, имени файла
и класса, в котором он находится), атрибутам, установленным в тестовой функции, маркерам,
примененным к нему или его родителям и любому :attr:`extra keywords <_pytest.nodes.Node.extra_keyword_matches>`
явно добавленным к нему или его родителям.


Регистрация маркеров
-------------------------------------



.. ini-syntax for custom markers:

Зарегистрировать маркеры для вашего набора тестов просто:

.. code-block:: ini

    # content of pytest.ini
    [pytest]
    markers =
        webtest: mark a test as a webtest.
        slow: mark test as slow.

Можно зарегистрировать несколько пользовательских маркеров, определив каждый в отдельной строке, как показано в примере выше.

Вы можете спросить, какие маркеры существуют для вашего набора тестов - в список включены только что определенные маркеры ``webtest`` и ``slow``:

.. code-block:: pytest

    $ pytest --markers
    @pytest.mark.webtest: mark a test as a webtest.

    @pytest.mark.slow: mark test as slow.

    @pytest.mark.filterwarnings(warning): add a warning filter to the given test. see https://docs.pytest.org/en/stable/warnings.html#pytest-mark-filterwarnings

    @pytest.mark.skip(reason=None): skip the given test function with an optional reason. Example: skip(reason="no way of currently testing this") skips the test.

    @pytest.mark.skipif(condition, ..., *, reason=...): skip the given test function if any of the conditions evaluate to True. Example: skipif(sys.platform == 'win32') skips the test if we are on the win32 platform. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-skipif

    @pytest.mark.xfail(condition, ..., *, reason=..., run=True, raises=None, strict=xfail_strict): mark the test function as an expected failure if any of the conditions evaluate to True. Optionally specify a reason for better reporting and run=False if you don't even want to execute the test function. If only specific exception(s) are expected, you can list them in raises, and if the test fails in other ways, it will be reported as a true failure. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-xfail

    @pytest.mark.parametrize(argnames, argvalues): call a test function multiple times passing in different arguments in turn. argvalues generally needs to be a list of values if argnames specifies only one name or a list of tuples of values if argnames specifies multiple names. Example: @parametrize('arg1', [1,2]) would lead to two calls of the decorated test function, one with arg1=1 and another with arg1=2.see https://docs.pytest.org/en/stable/parametrize.html for more info and examples.

    @pytest.mark.usefixtures(fixturename1, fixturename2, ...): mark tests as needing all of the specified fixtures. see https://docs.pytest.org/en/stable/fixture.html#usefixtures

    @pytest.mark.tryfirst: mark a hook implementation function such that the plugin machinery will try to call it first/as early as possible.

    @pytest.mark.trylast: mark a hook implementation function such that the plugin machinery will try to call it last/as late as possible.


Пример того, как добавлять маркеры и работать с ними из плагина:
:ref:`adding a custom marker from a plugin`.

.. note::

    Рекомендуется явно регистрировать маркеры так, чтобы:

    * Ваши маркеры определялись только в одном месте тестового набора

    * Получение списка маркеров с помощью ``pytest --markers`` давало правильный результат

    * Опечатки в маркерах рассматривались как ошибка при использовании опции ``--strict-markers``.

.. _`scoped-marking`:

Маркировка целых классов или модулей
----------------------------------------------------

Декоратор ``pytest.mark`` можно применять для классов, чтобы пометить все его тестовые методы:

.. code-block:: python

    # листинг test_mark_classlevel.py
    import pytest


    @pytest.mark.webtest
    class TestClass:
        def test_startup(self):
            pass

        def test_startup_and_more(self):
            pass

Такая запись равносильна применению декоратора к обеим тестовым функциям.

Чтобы применить отметки на уровне модуля, используйте глобальные переменные :globalvar:`pytestmark`::

    import pytest
    pytestmark = pytest.mark.webtest

или несколько маркеров::

    pytestmark = [pytest.mark.webtest, pytest.mark.slowtest]


Из-за устаревших причин до того, как были введены декораторы классов, можно было установить
:globalvar:`pytestmark` атрибут в тестовом классе, подобном этому:

.. code-block:: python

    import pytest


    class TestClass:
        pytestmark = pytest.mark.webtest

.. _`marking individual tests when using parametrize`:

Маркировка отдельных тестов при использовании параметризации
-------------------------------------------------------------

При использовании параметризации, маркировка будет применяться к каждому отдельному тесту.
Однако также можно применить маркер к отдельному экземпляру теста:

.. code-block:: python

    import pytest


    @pytest.mark.foo
    @pytest.mark.parametrize(
        ("n", "expected"), [(1, 2), pytest.param(1, 3, marks=pytest.mark.bar), (2, 3)]
    )
    def test_increment(n, expected):
        assert n + 1 == expected

В приведенном выше примере маркером "foo" окажутся помечены
все три запускаемых теста, а вот маркер "bar" будет применен только ко второму.
Тем же способом можно пометить ``skip`` и ``xfail`` тесты,
см. :ref:`skip/xfail with parametrize`.

.. _`adding a custom marker from a plugin`:

Пользовательский маркер и параметр командной строки для управления запуска тестов
---------------------------------------------------------------------------------------

.. regendoc:wipe

Плагины могут предоставлять настраиваемые маркеры и реализовывать
определенное поведение на их основе. Вот полноценный пример
добавления опции командной строки и параметризованного маркера тестовой
функции для запуска тестов в определенных виртуальных средах:

.. code-block:: python

    # листинг conftest.py

    import pytest


    def pytest_addoption(parser):
        parser.addoption(
            "-E",
            action="store",
            metavar="NAME",
            help="only run tests matching the environment NAME.",
        )


    def pytest_configure(config):
        # регистрация дополнительного маркера
        config.addinivalue_line(
            "markers", "env(name): mark test to run only on named environment"
        )


    def pytest_runtest_setup(item):
        envnames = [mark.args[0] for mark in item.iter_markers(name="env")]
        if envnames:
            if item.config.getoption("-E") not in envnames:
                pytest.skip("test requires env in {!r}".format(envnames))

Тестовый файл с использованием этого локального плагина:

.. code-block:: python

    # листинг test_someenv.py

    import pytest


    @pytest.mark.env("stage1")
    def test_basic_db_operation():
        pass

и пример вызовов, определяющих виртуальное окружение, отличные от того, что требуется для теста:

.. code-block:: pytest

    $ pytest -E stage2
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_someenv.py s                                                    [100%]

    ============================ 1 skipped in 0.12s ============================

и здесь мы запускаем тест в нужном виртуальном окружении:

.. code-block:: pytest

    $ pytest -E stage1
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_someenv.py .                                                    [100%]

    ============================ 1 passed in 0.12s =============================

Опцию ``--markers`` всегда можно использовать для получения актуального списка доступных маркеров:

.. code-block:: pytest

    $ pytest --markers
    @pytest.mark.env(name): mark test to run only on named environment

    @pytest.mark.filterwarnings(warning): add a warning filter to the given test. see https://docs.pytest.org/en/stable/warnings.html#pytest-mark-filterwarnings

    @pytest.mark.skip(reason=None): skip the given test function with an optional reason. Example: skip(reason="no way of currently testing this") skips the test.

    @pytest.mark.skipif(condition, ..., *, reason=...): skip the given test function if any of the conditions evaluate to True. Example: skipif(sys.platform == 'win32') skips the test if we are on the win32 platform. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-skipif

    @pytest.mark.xfail(condition, ..., *, reason=..., run=True, raises=None, strict=xfail_strict): mark the test function as an expected failure if any of the conditions evaluate to True. Optionally specify a reason for better reporting and run=False if you don't even want to execute the test function. If only specific exception(s) are expected, you can list them in raises, and if the test fails in other ways, it will be reported as a true failure. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-xfail

    @pytest.mark.parametrize(argnames, argvalues): call a test function multiple times passing in different arguments in turn. argvalues generally needs to be a list of values if argnames specifies only one name or a list of tuples of values if argnames specifies multiple names. Example: @parametrize('arg1', [1,2]) would lead to two calls of the decorated test function, one with arg1=1 and another with arg1=2.see https://docs.pytest.org/en/stable/parametrize.html for more info and examples.

    @pytest.mark.usefixtures(fixturename1, fixturename2, ...): mark tests as needing all of the specified fixtures. see https://docs.pytest.org/en/stable/fixture.html#usefixtures

    @pytest.mark.tryfirst: mark a hook implementation function such that the plugin machinery will try to call it first/as early as possible.

    @pytest.mark.trylast: mark a hook implementation function such that the plugin machinery will try to call it last/as late as possible.


.. _`passing callables to custom markers`:

Передача вызываемого объекта пользовательскими маркерами
---------------------------------------------------------

.. regendoc:wipe

Ниже приведен файл конфигурации, который будет использоваться в следующих примерах:

.. code-block:: python

    # листинг conftest.py
    import sys


    def pytest_runtest_setup(item):
        for marker in item.iter_markers(name="my_marker"):
            print(marker)
            sys.stdout.flush()

Настраиваемый маркер может иметь свое множество позиционных и именованных аргументов, т. е. свойств
``args`` и ``kwargs``, которые можно передать как с помощью вызова callable-объекта, так и с помощью
``pytest.mark.MARKER_NAME.with_args``. Эти два метода в большинстве случаев дают одинаковый эффект.

Однако, если единственным позиционным аргументом является callable-объект
без именованных аргументов, использование ``pytest.mark.MARKER_NAME(c)`` не передаст
``c`` в качестве позиционного аргумента, а просто обернет ``c`` нашим маркером
(см. :ref:`MarkDecorator <mark>`). К счастью, на помощь приходит ``pytest.mark.MARKER_NAME.with_args``:

.. code-block:: python

    # листинг test_custom_marker.py
    import pytest


    def hello_world(*args, **kwargs):
        return "Hello World"


    @pytest.mark.my_marker.with_args(hello_world)
    def test_with_args():
        pass

Результат выглядит следующим образом:

.. code-block:: pytest

    $ pytest -q -s
    Mark(name='my_marker', args=(<function hello_world at 0xdeadbeef>,), kwargs={})
    .
    1 passed in 0.12s

Можно заметить, что у нашего настраиваемого маркера есть свое множество аргументов,
одним из которых является функция ``hello_world``. В этом и заключается ключевое различие между
созданием маркера в качестве callable-объекта, который за кулисами
вызывает ``__call__``, и использованием ``with_args``.


Считывание маркеров, которые были установлены из нескольких мест
----------------------------------------------------------------

.. versionadded: 2.2.2

.. regendoc:wipe

Если вы активно используете маркеры в своем наборе тестов, вы можете столкнуться со случаем, когда
маркер применяется несколько раз к тестовой функции. Из кода плагина вы можете прочитать
все такие настройки. Пример:

.. code-block:: python

    # листинг test_mark_three_times.py
    import pytest

    pytestmark = pytest.mark.glob("module", x=1)


    @pytest.mark.glob("class", x=2)
    class TestClass:
        @pytest.mark.glob("function", x=3)
        def test_something(self):
            pass

Здесь у нас маркер "glob" применяется к одной и той же функции три раза.
Мы можем увидеть это, прописав в файле ``conftest.py``:

.. code-block:: python

    # листинг conftest.py
    import sys


    def pytest_runtest_setup(item):
        for mark in item.iter_markers(name="glob"):
            print("glob args={} kwargs={}".format(mark.args, mark.kwargs))
            sys.stdout.flush()

Давайте запустим без перехвата вывода и посмотрим, что получится:

.. code-block:: pytest

    $ pytest -q -s
    glob args=('function',) kwargs={'x': 3}
    glob args=('class',) kwargs={'x': 2}
    glob args=('module',) kwargs={'x': 1}
    .
    1 passed in 0.12s

Маркировка тестов, специфичных для платформы, с помощью pytest
---------------------------------------------------------------

.. regendoc:wipe

Предположим, что у нас есть тестовый набор, в котором мы используем маркеры
``pytest.mark.darwin``, ``pytest.mark.win32`` и т. п. для маркировки тестов,
запускаемых на разных платформах. При этом в набор также входят тесты, которые должны
проводиться на всех платформах, и они никак не помечены. Теперь, если вы хотите запустить
тесты для конкретной платформы, может пригодиться такой плагин:

.. code-block:: python

    # листинг conftest.py
    #
    import sys
    import pytest

    ALL = set("darwin linux win32".split())


    def pytest_runtest_setup(item):
        supported_platforms = ALL.intersection(mark.name for mark in item.iter_markers())
        plat = sys.platform
        if supported_platforms and plat not in supported_platforms:
            pytest.skip("cannot run on platform {}".format(plat))

то тесты будут пропущены, если они были указаны для другой платформы.
Давайте создадим небольшой тестовый файл, чтобы показать, как это выглядит:

.. code-block:: python

    # листинг test_plat.py

    import pytest


    @pytest.mark.darwin
    def test_if_apple_is_evil():
        pass


    @pytest.mark.linux
    def test_if_linux_works():
        pass


    @pytest.mark.win32
    def test_if_win32_crashes():
        pass


    def test_runs_everywhere():
        pass

тогда вы увидите два пропущенных теста и два выполненных, как и ожидалось:

.. code-block:: pytest

    $ pytest -rs # this option reports skip reasons
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items

    test_plat.py s.s.                                                    [100%]

    ========================= short test summary info ==========================
    SKIPPED [2] conftest.py:12: cannot run on platform linux
    ======================= 2 passed, 2 skipped in 0.12s =======================

Обратите внимание, что если вы определяете платформу с помощью маркера и опции ``-m``,
как показано ниже:

.. code-block:: pytest

    $ pytest -m linux
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items / 3 deselected / 1 selected

    test_plat.py .                                                       [100%]

    ===================== 1 passed, 3 deselected in 0.12s ======================

то непомеченные тесты запускаться не будут. Таким образом, это способ ограничиться
выполнением конкретных тестов.

Автоматическое добавление маркеров на основе имен тестов
---------------------------------------------------------

.. regendoc:wipe

Если у вас есть набор тестов, в котором имена тестовых функций указывают на определенный
тип теста, вы можете реализовать ловушку(hook), которая автоматически определяет маркеры, чтобы
вы могли использовать с ним опцию ``-m``. Посмотрим на этот тестовый модуль:

.. code-block:: python

    # листинг test_module.py


    def test_interface_simple():
        assert 0


    def test_interface_complex():
        assert 0


    def test_event_simple():
        assert 0


    def test_something_else():
        assert 0

Мы хотим динамически маркировать тесты и можем сделать это в плагине ``conftest.py``:

.. code-block:: python

    # листинг conftest.py

    import pytest


    def pytest_collection_modifyitems(items):
        for item in items:
            if "interface" in item.nodeid:
                item.add_marker(pytest.mark.interface)
            elif "event" in item.nodeid:
                item.add_marker(pytest.mark.event)

Теперь мы можем использовать ``-m option`` для выбора одного набора:

.. code-block:: pytest

    $ pytest -m interface --tb=short
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items / 2 deselected / 2 selected

    test_module.py FF                                                    [100%]

    ================================= FAILURES =================================
    __________________________ test_interface_simple ___________________________
    test_module.py:4: in test_interface_simple
        assert 0
    E   assert 0
    __________________________ test_interface_complex __________________________
    test_module.py:8: in test_interface_complex
        assert 0
    E   assert 0
    ========================= short test summary info ==========================
    FAILED test_module.py::test_interface_simple - assert 0
    FAILED test_module.py::test_interface_complex - assert 0
    ===================== 2 failed, 2 deselected in 0.12s ======================

или выбрать оба теста: "event" и "interface":

.. code-block:: pytest

    $ pytest -m "interface or event" --tb=short
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items / 1 deselected / 3 selected

    test_module.py FFF                                                   [100%]

    ================================= FAILURES =================================
    __________________________ test_interface_simple ___________________________
    test_module.py:4: in test_interface_simple
        assert 0
    E   assert 0
    __________________________ test_interface_complex __________________________
    test_module.py:8: in test_interface_complex
        assert 0
    E   assert 0
    ____________________________ test_event_simple _____________________________
    test_module.py:12: in test_event_simple
        assert 0
    E   assert 0
    ========================= short test summary info ==========================
    FAILED test_module.py::test_interface_simple - assert 0
    FAILED test_module.py::test_interface_complex - assert 0
    FAILED test_module.py::test_event_simple - assert 0
    ===================== 3 failed, 1 deselected in 0.12s ======================
