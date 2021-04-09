
.. _paramexamples:

Параметризация тестов
=================================================

.. currentmodule:: _pytest.python

``pytest`` позволяет легко параметризовать тестовые функции.
Базовую документацию см. :ref:`parametrize-basics`.

Ниже мы приводим несколько примеров с использованием встроенных механизмов.

Генерация комбинаций параметров в зависимости от командной строки
----------------------------------------------------------------------------

.. regendoc:wipe

Допустим, мы хотим выполнить тест с разными параметрами вычислений, и диапазон
параметров должен определяться аргументом командной строки. Давайте сначала напишем
простой (ничего не делающий) вычислительный тест:

.. code-block:: python

    # листинг test_compute.py


    def test_compute(param1):
        assert param1 < 4

Теперь мы добавляем такую тестовую конфигурацию:

.. code-block:: python

    # листинг conftest.py


    def pytest_addoption(parser):
        parser.addoption("--all", action="store_true", help="run all combinations")


    def pytest_generate_tests(metafunc):
        if "param1" in metafunc.fixturenames:
            if metafunc.config.getoption("all"):
                end = 5
            else:
                end = 2
            metafunc.parametrize("param1", range(end))

Это означает, что мы запускаем только 2 теста, если мы не используем ``--all``:

.. code-block:: pytest

    $ pytest -q test_compute.py
    ..                                                                   [100%]
    2 passed in 0.12s

Мы запускаем только два вычисления, поэтому мы видим две точки.
Давайте воспользуемся нашей опцией:

.. code-block:: pytest

    $ pytest -q --all
    ....F                                                                [100%]
    ================================= FAILURES =================================
    _____________________________ test_compute[4] ______________________________

    param1 = 4

        def test_compute(param1):
    >       assert param1 < 4
    E       assert 4 < 4

    test_compute.py:4: AssertionError
    ========================= short test summary info ==========================
    FAILED test_compute.py::test_compute[4] - assert 4 < 4
    1 failed, 4 passed in 0.12s

Как и ожидалось, при выполнении тестов по всему диапазону значений
``param1``, мы получили ошибку на последнем тесте.


Различные способы определения ID тестов
----------------------------------------

``pytest`` конструирует строку, которая является идентификатором (ID) теста
для каждого множества значений параметризованного теста. Эти идентификаторы
можно использовать с опцией ``-k``, чтобы отобрать для выполнения определенные
тесты, и они же идентифицируют конкретный упавший тест. Запустив
``pytest --collect-only`` , можно посмотреть сгенерированные ID.

У чисел, строк, логических значений и значения None есть свои строковые
представления, которые используются в ID тестов. Для остальных объектов
``pytest`` генерирует ID на основании имен аргументов:

.. code-block:: python

    # листинг test_time.py

    from datetime import datetime, timedelta

    import pytest

    testdata = [
        (datetime(2001, 12, 12), datetime(2001, 12, 11), timedelta(1)),
        (datetime(2001, 12, 11), datetime(2001, 12, 12), timedelta(-1)),
    ]


    @pytest.mark.parametrize("a,b,expected", testdata)
    def test_timedistance_v0(a, b, expected):
        diff = a - b
        assert diff == expected


    @pytest.mark.parametrize("a,b,expected", testdata, ids=["forward", "backward"])
    def test_timedistance_v1(a, b, expected):
        diff = a - b
        assert diff == expected


    def idfn(val):
        if isinstance(val, (datetime,)):
            # обратите внимание, здесь не будут отображаться часы/минуты/секунды
            return val.strftime("%Y%m%d")


    @pytest.mark.parametrize("a,b,expected", testdata, ids=idfn)
    def test_timedistance_v2(a, b, expected):
        diff = a - b
        assert diff == expected


    @pytest.mark.parametrize(
        "a,b,expected",
        [
            pytest.param(
                datetime(2001, 12, 12), datetime(2001, 12, 11), timedelta(1), id="forward"
            ),
            pytest.param(
                datetime(2001, 12, 11), datetime(2001, 12, 12), timedelta(-1), id="backward"
            ),
        ],
    )
    def test_timedistance_v3(a, b, expected):
        diff = a - b
        assert diff == expected

В ``test_timedistance_v0`` мы позволяем ``pytest`` самому генерировать ID.

В ``test_timedistance_v1`` мы указали ``ids`` как список строк, которые были использованы
в качестве ID. Они весьма кратки, но могут быть сложны для поддержания.

В ``test_timedistance_v2`` мы определяем ``ids`` как функции, которые могут генерировать
строковое представление для включения в тестовый ID.
Здесь обозначения наших ``datetime``-аргументов генерируются функцией ``idfn``,
но из-за того, что мы не можем формировать представление для объектов ``timedelta``,
для них все еще используется стандартное представление ``pytest``:

.. code-block:: pytest

    $ pytest test_time.py --collect-only
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 8 items

    <Module test_time.py>
      <Function test_timedistance_v0[a0-b0-expected0]>
      <Function test_timedistance_v0[a1-b1-expected1]>
      <Function test_timedistance_v1[forward]>
      <Function test_timedistance_v1[backward]>
      <Function test_timedistance_v2[20011212-20011211-expected0]>
      <Function test_timedistance_v2[20011211-20011212-expected1]>
      <Function test_timedistance_v3[forward]>
      <Function test_timedistance_v3[backward]>

    ======================== 8 tests collected in 0.12s ========================

В ``test_timedistance_v3`` мы используем ``pytest.param`` для указания ID вместе
с конкретными данными, вместо того, чтобы перечислять их по отдельности.

Быстрый запуск "testscenarios"
------------------------------------

.. _`test scenarios`: https://pypi.org/project/testscenarios/

Вот быстрый способ запуска тестов, сконфигурированных с помощью `test scenarios`_,
дополнения от Роберта Коллинза для стандартного фреймворка ``unittest``. Тут придется
немного потрудиться с созданием корректных аргументов для :py:func:`Metafunc.parametrize`:

.. code-block:: python

    # листинг test_scenarios.py


    def pytest_generate_tests(metafunc):
        idlist = []
        argvalues = []
        for scenario in metafunc.cls.scenarios:
            idlist.append(scenario[0])
            items = scenario[1].items()
            argnames = [x[0] for x in items]
            argvalues.append([x[1] for x in items])
        metafunc.parametrize(argnames, argvalues, ids=idlist, scope="class")


    scenario1 = ("basic", {"attribute": "value"})
    scenario2 = ("advanced", {"attribute": "value2"})


    class TestSampleWithScenarios:
        scenarios = [scenario1, scenario2]

        def test_demo1(self, attribute):
            assert isinstance(attribute, str)

        def test_demo2(self, attribute):
            assert isinstance(attribute, str)

это полностью самодостаточный пример, который можно запустить:

.. code-block:: pytest

    $ pytest test_scenarios.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items

    test_scenarios.py ....                                               [100%]

    ============================ 4 passed in 0.12s =============================

Если вы просто соберете тесты, то увидите 'advanced' и 'basic' в качестве
переменных тестовой функции:

.. code-block:: pytest

    $ pytest --collect-only test_scenarios.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 4 items

    <Module test_scenarios.py>
      <Class TestSampleWithScenarios>
          <Function test_demo1[basic]>
          <Function test_demo2[basic]>
          <Function test_demo1[advanced]>
          <Function test_demo2[advanced]>

    ======================== 4 tests collected in 0.12s ========================

Обратите внимание: мы сообщили ``metafunc.parametrize()``, что значения
вашего сценария должны рассматриваться как классово ограниченными. В pytest-2.3 это
приводит к упорядочиванию на основе ресурсов.


Отсрочка настройки параметризованных ресурсов
---------------------------------------------------

.. regendoc:wipe

Параметризация тестовой функции происходит во время сборки тестов.
Рекомендуется настраивать затратные по ресурсам, такие как подключения к БД или
подпроцессы, только при запуске фактического теста. Вот простой пример, как этого
добиться. В этом тесте требуется объектовая фикстура ``db``:


.. code-block:: python

    # листинг test_backends.py

    import pytest


    def test_db_initialized(db):
        # макет теста
        if db.__class__.__name__ == "DB2":
            pytest.fail("deliberately failing for demo purposes")

Теперь мы можем добавить тестовую конфигурацию, которая генерирует два вызова
функции ``test_db_initialized`` и также реализует фабрику, которая создает
объект базы данных для фактического вызова теста:


.. code-block:: python

    # листинг conftest.py
    import pytest


    def pytest_generate_tests(metafunc):
        if "db" in metafunc.fixturenames:
            metafunc.parametrize("db", ["d1", "d2"], indirect=True)


    class DB1:
        "первый объект базы данных"


    class DB2:
        "альтернативный объект базы данных"


    @pytest.fixture
    def db(request):
        if request.param == "d1":
            return DB1()
        elif request.param == "d2":
            return DB2()
        else:
            raise ValueError("invalid internal test config")

Давайте сначала посмотрим, как все это выглядит во время сборки:

.. code-block:: pytest

    $ pytest test_backends.py --collect-only
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 2 items

    <Module test_backends.py>
      <Function test_db_initialized[d1]>
      <Function test_db_initialized[d2]>

    ======================== 2 tests collected in 0.12s ========================

И затем, когда мы запустим тест:

.. code-block:: pytest

    $ pytest -q test_backends.py
    .F                                                                   [100%]
    ================================= FAILURES =================================
    _________________________ test_db_initialized[d2] __________________________

    db = <conftest.DB2 object at 0xdeadbeef>

        def test_db_initialized(db):
            # a dummy test
            if db.__class__.__name__ == "DB2":
    >           pytest.fail("deliberately failing for demo purposes")
    E           Failed: deliberately failing for demo purposes

    test_backends.py:8: Failed
    ========================= short test summary info ==========================
    FAILED test_backends.py::test_db_initialized[d2] - Failed: deliberately f...
    1 failed, 1 passed in 0.12s

Первый вызов с ``db == "DB1"`` прошел, в то время как второй с ``db == "DB2"`` - упал.
Наша фикстура ``db`` создала экземпляр каждого из значений DB на этапе настройки, в
то время как ``pytest_generate_tests`` сгенерировал два соответствующих вызова
``test_db_initialized`` на этапе сборки.


Косвенная(indirect) параметризация
---------------------------------------------------

Использование параметра ``indirect=True`` при параметризации теста позволяет
параметризовать тест фикстурой, получающей значения перед передачей их в тест:

.. code-block:: python

    import pytest


    @pytest.fixture
    def fixt(request):
        return request.param * 3


    @pytest.mark.parametrize("fixt", ["a", "b"], indirect=True)
    def test_indirect(fixt):
        assert len(fixt) == 3

Это можно использовать, например, для более подробной настройки во время выполнения
теста в фикстуре, вместо того, чтобы выполнять эти шаги настройки во время сбора
данных.


.. regendoc:wipe

Применение ``indirect`` к отдельному аргументу
---------------------------------------------------

Очень часто при параметризации используется более одного аргумента.
Существует возможность применить параметр ``indirect`` к конкретным аргументам.
Это можно сделать, передав список или кортеж имен аргументов параметру ``indirect``.
В нижеприведенном примере есть функция ``test_indirect``, которая
использует две фикстуры: ``x`` и ``y``.
Здесь мы передаем ``indirect`` список, который содержит имя фикстуры ``x``.
Параметр ``indirect`` будет применен только к этому аргументу, а значение ``a``
будет передано соответствующей функции фикстуры:


.. code-block:: python

    # листинг test_indirect_list.py

    import pytest


    @pytest.fixture(scope="function")
    def x(request):
        return request.param * 3


    @pytest.fixture(scope="function")
    def y(request):
        return request.param * 2


    @pytest.mark.parametrize("x, y", [("a", "b")], indirect=["x"])
    def test_indirect(x, y):
        assert x == "aaa"
        assert y == "b"

Тест пройдет успешно:

.. code-block:: pytest

    $ pytest -v test_indirect_list.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 1 item

    test_indirect_list.py::test_indirect[a-b] PASSED                     [100%]

    ============================ 1 passed in 0.12s =============================

.. regendoc:wipe

Параметризация тестовых методов через конфигурацию для каждого класса
-----------------------------------------------------------------------------

.. _`unittest parametrizer`: https://github.com/testing-cabal/unittest-ext/blob/master/params.py

Вот пример функции ``pytest_generate_tests``, реализующей схему параметризации,
аналогичную `unittest parametrizer`_ от Michael Foord, но с более коротким кодом:

.. code-block:: python

    # листинг ./test_parametrize.py
    import pytest


    def pytest_generate_tests(metafunc):
        # вызывается один раз для каждой тестовой функции
        funcarglist = metafunc.cls.params[metafunc.function.__name__]
        argnames = sorted(funcarglist[0])
        metafunc.parametrize(
            argnames, [[funcargs[name] for name in argnames] for funcargs in funcarglist]
        )


    class TestClass:
        # схема, определяющая несколько наборов аргументов для тестового метода
        params = {
            "test_equals": [dict(a=1, b=2), dict(a=3, b=3)],
            "test_zerodivision": [dict(a=1, b=0)],
        }

        def test_equals(self, a, b):
            assert a == b

        def test_zerodivision(self, a, b):
            with pytest.raises(ZeroDivisionError):
                a / b

Наш тестовый генератор отслеживает определение множества аргументов
для каждой тестовой функции на уровне класса. Запустим его:


.. code-block:: pytest

    $ pytest -q
    F..                                                                  [100%]
    ================================= FAILURES =================================
    ________________________ TestClass.test_equals[1-2] ________________________

    self = <test_parametrize.TestClass object at 0xdeadbeef>, a = 1, b = 2

        def test_equals(self, a, b):
    >       assert a == b
    E       assert 1 == 2

    test_parametrize.py:21: AssertionError
    ========================= short test summary info ==========================
    FAILED test_parametrize.py::TestClass::test_equals[1-2] - assert 1 == 2
    1 failed, 2 passed in 0.12s

Косвенная(Indirect) параметризация с несколькими фикстурами
--------------------------------------------------------------

Вот сокращенный реальный пример использования параметризованного тестирования
для тестирования сериализации объектов различными интерпретаторами Python.
Мы определяем функцию ``test_basic_objects``, которая должна запускаться с
различными множествами трех своих аргументов:

* ``python1``: первый интерпретатор python, запускается для сериализации объекта в файл
* ``python2``: второй интерпретатор python, запускается для восстановления объекта из файла
* ``obj``: объект, который будет выгружен/загружен

.. literalinclude:: multipython.py

Запуск приводит к некоторым пропускам, если не установлены все интерпретаторы
python, а в противном случае запускаются все возможные комбинации (3 интерпретатора
умножить на 3 интерпретатора умножить на 3 объекта для сериализации/десериализации):

.. code-block:: pytest

   . $ pytest -rs -q multipython.py
   sssssssssssssssssssssssssss                                          [100%]
   ========================= short test summary info ==========================
   SKIPPED [9] multipython.py:29: 'python3.5' not found
   SKIPPED [9] multipython.py:29: 'python3.6' not found
   SKIPPED [9] multipython.py:29: 'python3.7' not found
   27 skipped in 0.12s

Косвенная параметризация дополнительных реализаций/импорта
--------------------------------------------------------------------

Если вы хотите сравнить результаты нескольких реализаций данного API, вы можете написать
тестовые функции, которые получают уже импортированные реализации и пропускаются в
случае, если реализация недоступна для импорта или недоступна. Допустим, у нас есть
«базовая» реализация, а другая (возможно, оптимизированная) должна давать аналогичные
результаты:

.. code-block:: python

    # листинг conftest.py

    import pytest


    @pytest.fixture(scope="session")
    def basemod(request):
        return pytest.importorskip("base")


    @pytest.fixture(scope="session", params=["opt1", "opt2"])
    def optmod(request):
        return pytest.importorskip(request.param)

А затем базовая реализация простой функции:

.. code-block:: python

    # листинг base.py
    def func1():
        return 1

И оптимизированная версия:

.. code-block:: python

    # листинг opt1.py
    def func1():
        return 1.0001

И наконец небольшой тестовый модуль:

.. code-block:: python

    # листинг test_module.py


    def test_func1(basemod, optmod):
        assert round(basemod.func1(), 3) == round(optmod.func1(), 3)


Если вы запустите с включенном отчетом о пропусках:

.. code-block:: pytest

    $ pytest -rs test_module.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 2 items

    test_module.py .s                                                    [100%]

    ========================= short test summary info ==========================
    SKIPPED [1] conftest.py:12: could not import 'opt2': No module named 'opt2'
    ======================= 1 passed, 1 skipped in 0.12s =======================

Вы увидите, что у нас нет модуля ``opt2``, поэтому второй тест ``test_func1``
был пропущен.  Несколько замечаний:

- фикстуры в файле ``conftest.py`` имеют уровень сессии, ибо нам не нужно импортировать
  модуль больше одного раза;

- если у вас несколько тестовых функций и пропущены импортированные, счетчик
  ``[1]`` увеличится в отчете;

- для параметризации тестовых функций можно также использовать
  :ref:`@pytest.mark.parametrize <@pytest.mark.parametrize>`.


Установка маркеров или ID для индивидуального параметризованного теста
-----------------------------------------------------------------------

Используйте ``pytest.param``, чтобы применить маркеры или установить ID конкретному тесту.
Например:

.. code-block:: python

    # листинг test_pytest_param_example.py
    import pytest


    @pytest.mark.parametrize(
        "test_input,expected",
        [
            ("3+5", 8),
            pytest.param("1+7", 8, marks=pytest.mark.basic),
            pytest.param("2+4", 6, marks=pytest.mark.basic, id="basic_2+4"),
            pytest.param(
                "6*9", 42, marks=[pytest.mark.basic, pytest.mark.xfail], id="basic_6*9"
            ),
        ],
    )
    def test_eval(test_input, expected):
        assert eval(test_input) == expected

В этом примере 4 параметризованных теста. Исключая первый тест, отмечаем остальные
три параметризованных теста специальным маркером ``basic``, а для четвертого теста
используем встроенный маркер ``xfail``, чтобы указать, что ожидаем его падение.
Для наглядности устанавливаем ID для некоторых тестов.

Теперь запустим ``pytest`` в подробном режиме и только с маркерами ``basic``:

.. code-block:: pytest

    $ pytest -v -m basic
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 24 items / 21 deselected / 3 selected

    test_pytest_param_example.py::test_eval[1+7-8] PASSED                [ 33%]
    test_pytest_param_example.py::test_eval[basic_2+4] PASSED            [ 66%]
    test_pytest_param_example.py::test_eval[basic_6*9] XFAIL             [100%]

    =============== 2 passed, 21 deselected, 1 xfailed in 0.12s ================

В результате:

- было собрано четыре теста;
- один тест был отменен, поскольку он не имел маркера ``basic``;
- три теста с маркером ``basic`` были выбраны;
- тест ``test_eval[1+7-8]`` прошел, но его имя генерируется автоматически и сбивает с толку;
- тест ``test_eval[basic_2+4]`` прошел;
- тест ``test_eval[basic_6*9]`` должен был упасть и действительно не прошел.

.. _`parametrizing_conditional_raising`:

Параметризованные условные исключения
--------------------------------------------------------------------

Используйте :func:`pytest.raises` совместно с декоратором :ref:`pytest.mark.parametrize ref`,
чтобы писать параметризованные тесты, в которых одни тесты вызывают исключения, а
другие - нет.

Полезно определить менеджер контекста ``does_not_raise``, который
будет служить дополнением к ``raises``. Например:

.. code-block:: python

    from contextlib import contextmanager
    import pytest


    @contextmanager
    def does_not_raise():
        yield


    @pytest.mark.parametrize(
        "example_input,expectation",
        [
            (3, does_not_raise()),
            (2, does_not_raise()),
            (1, does_not_raise()),
            (0, pytest.raises(ZeroDivisionError)),
        ],
    )
    def test_division(example_input, expectation):
        """Проверка насколько я знаю деление."""
        with expectation:
            assert (6 / example_input) is not None

В приведенном выше примере, первые три тестовых примера должны выполняться
без исключений, а четвертый должен вызывать ``ZeroDivisionError``.

Если планируется поддержка только Python 3.7+, можно просто использовать ``nullcontext``
вместо ``does_not_raise``:

.. code-block:: python

    from contextlib import nullcontext as does_not_raise

Если поддерживается ``Python 3.3`` и выше:

.. code-block:: python

    from contextlib import ExitStack as does_not_raise

Или при желании можно установить ``pip install contextlib2`` и использовать:

.. code-block:: python

    from contextlib2 import nullcontext as does_not_raise
