.. _`cache_provider`:
.. _cache:


Повторный запуск упавших тестов и сохранение состояния между тестовыми запусками
=====================================================================================


Применение
-----------

Плагин предоставляет два параметра командной строки для повторного запуска из последнего
вызова ``pytest``:

* ``--lf``, ``--last-failed`` - только повторно запустить сбои.
* ``--ff``, ``--failed-first`` - сначала запустить сбои, а затем остальные тесты.

Для очистки, которая обычно не требуется, параметр ``--cache-clear`` позволяет удалить все содержимое
межсессионного кеша перед запуском теста.

Другие плагины могут обращаться к объекту `config.cache`_ для установки/получения значений, **кодируемых в
json**, между вызовами ``pytest``.

.. note::

    Этот плагин включен по умолчанию, но при необходимости его можно отключить:
    см. :ref:`cmdunregister` (внутреннее имя этого плагина ``cacheprovider``).


Повторный запуск только отказов или отказов в первую очередь
-----------------------------------------------------------------

Во-первых, давайте создадим 50 тестовых вызовов, из которых только 2 не упадут:

.. code-block:: python

    # листинг test_50.py
    import pytest


    @pytest.mark.parametrize("i", range(50))
    def test_num(i):
        if i in (17, 25):
            pytest.fail("bad luck")

Если вы запустите это в первый раз, вы увидите два падения:

.. code-block:: pytest

    $ pytest -q
    .................F.......F........................                   [100%]
    ================================= FAILURES =================================
    _______________________________ test_num[17] _______________________________

    i = 17

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    _______________________________ test_num[25] _______________________________

    i = 25

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    ========================= short test summary info ==========================
    FAILED test_50.py::test_num[17] - Failed: bad luck
    FAILED test_50.py::test_num[25] - Failed: bad luck
    2 failed, 48 passed in 0.12s

Если вы затем запустите его с ``--lf``:

.. code-block:: pytest

    $ pytest --lf
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 2 items
    run-last-failure: rerun previous 2 failures

    test_50.py FF                                                        [100%]

    ================================= FAILURES =================================
    _______________________________ test_num[17] _______________________________

    i = 17

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    _______________________________ test_num[25] _______________________________

    i = 25

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    ========================= short test summary info ==========================
    FAILED test_50.py::test_num[17] - Failed: bad luck
    FAILED test_50.py::test_num[25] - Failed: bad luck
    ============================ 2 failed in 0.12s =============================

Выполнились только два упавших теста из последнего запуска, в то время как 48 успешных тестов
не были запущены.("deselected").

Теперь, если вы запустите с параметром ``--ff``, все тесты будут запущены, но первые предыдущие
ошибочные будут выполнены первыми (как видно из серии ``FF`` и точек):

.. code-block:: pytest

    $ pytest --ff
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 50 items
    run-last-failure: rerun previous 2 failures first

    test_50.py FF................................................        [100%]

    ================================= FAILURES =================================
    _______________________________ test_num[17] _______________________________

    i = 17

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    _______________________________ test_num[25] _______________________________

    i = 25

        @pytest.mark.parametrize("i", range(50))
        def test_num(i):
            if i in (17, 25):
    >           pytest.fail("bad luck")
    E           Failed: bad luck

    test_50.py:7: Failed
    ========================= short test summary info ==========================
    FAILED test_50.py::test_num[17] - Failed: bad luck
    FAILED test_50.py::test_num[25] - Failed: bad luck
    ======================= 2 failed, 48 passed in 0.12s =======================

.. _`config.cache`:

Также есть ``новые`` параметры ``--nf``, ``--new-first``: сначала запускать новые тесты, а затем
остальные. В обоих случаях тесты также сортируются по времени изменения файла, с более новыми
файлами на первом месте.

Поведение при отсутствии неудачных тестов при последнем запуске
------------------------------------------------------------------

Когда ни один тест не упал в последнем прогоне или когда не найдены данные ``lastfailed``, ``pytest``
можно настроить либо на запуск всех тестов, либо на пропуск тестов,
используя аргумент ``--last-failed-no-failures``, который принимает одно из следующих значений:

.. code-block:: bash

    pytest --last-failed --last-failed-no-failures all    # запустить все тесты (поведение по умолчанию)
    pytest --last-failed --last-failed-no-failures none   # не запускать тесты и выйти

Новый объект config.cache
--------------------------------

.. regendoc:wipe

Плагины или ``conftest.py`` могут получить кешированное значение с помощью объекта
``config``. Вот базовый пример плагина, использующий фикстуру :ref:`fixture <fixture>`,
который повторно использует ранее созданное состояние при вызовах ``pytest``:

.. code-block:: python

    # листинг test_caching.py
    import pytest
    import time


    def expensive_computation():
        print("running expensive computation...")


    @pytest.fixture
    def mydata(request):
        val = request.config.cache.get("example/value", None)
        if val is None:
            expensive_computation()
            val = 42
            request.config.cache.set("example/value", val)
        return val


    def test_function(mydata):
        assert mydata == 23

Если вы запустите эту команду в первый раз, вы увидите сообщение:

.. code-block:: pytest

    $ pytest -q
    F                                                                    [100%]
    ================================= FAILURES =================================
    ______________________________ test_function _______________________________

    mydata = 42

        def test_function(mydata):
    >       assert mydata == 23
    E       assert 42 == 23

    test_caching.py:20: AssertionError
    -------------------------- Captured stdout setup ---------------------------
    running expensive computation...
    ========================= short test summary info ==========================
    FAILED test_caching.py::test_function - assert 42 == 23
    1 failed in 0.12s

Если запустить его повторно, значение будет извлечено из кеша и ничего не будет напечатано:

.. code-block:: pytest

    $ pytest -q
    F                                                                    [100%]
    ================================= FAILURES =================================
    ______________________________ test_function _______________________________

    mydata = 42

        def test_function(mydata):
    >       assert mydata == 23
    E       assert 42 == 23

    test_caching.py:20: AssertionError
    ========================= short test summary info ==========================
    FAILED test_caching.py::test_function - assert 42 == 23
    1 failed in 0.12s

Больше подробностей см. :fixture:`config.cache fixture <cache>`.


Проверка содержимого кеша
--------------------------

Вы всегда можете посмотреть содержимое кеша с помощью опции командной строки ``--cache-show``:

.. code-block:: pytest

    $ pytest --cache-show
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    cachedir: $PYTHON_PREFIX/.pytest_cache
    --------------------------- cache values for '*' ---------------------------
    cache/lastfailed contains:
      {'test_50.py::test_num[17]': True,
       'test_50.py::test_num[25]': True,
       'test_assert1.py::test_function': True,
       'test_assert2.py::test_set_comparison': True,
       'test_caching.py::test_function': True,
       'test_foocompare.py::test_compare': True}
    cache/nodeids contains:
      ['test_50.py::test_num[0]',
       'test_50.py::test_num[10]',
       'test_50.py::test_num[11]',
       'test_50.py::test_num[12]',
       'test_50.py::test_num[13]',
       'test_50.py::test_num[14]',
       'test_50.py::test_num[15]',
       'test_50.py::test_num[16]',
       'test_50.py::test_num[17]',
       'test_50.py::test_num[18]',
       'test_50.py::test_num[19]',
       'test_50.py::test_num[1]',
       'test_50.py::test_num[20]',
       'test_50.py::test_num[21]',
       'test_50.py::test_num[22]',
       'test_50.py::test_num[23]',
       'test_50.py::test_num[24]',
       'test_50.py::test_num[25]',
       'test_50.py::test_num[26]',
       'test_50.py::test_num[27]',
       'test_50.py::test_num[28]',
       'test_50.py::test_num[29]',
       'test_50.py::test_num[2]',
       'test_50.py::test_num[30]',
       'test_50.py::test_num[31]',
       'test_50.py::test_num[32]',
       'test_50.py::test_num[33]',
       'test_50.py::test_num[34]',
       'test_50.py::test_num[35]',
       'test_50.py::test_num[36]',
       'test_50.py::test_num[37]',
       'test_50.py::test_num[38]',
       'test_50.py::test_num[39]',
       'test_50.py::test_num[3]',
       'test_50.py::test_num[40]',
       'test_50.py::test_num[41]',
       'test_50.py::test_num[42]',
       'test_50.py::test_num[43]',
       'test_50.py::test_num[44]',
       'test_50.py::test_num[45]',
       'test_50.py::test_num[46]',
       'test_50.py::test_num[47]',
       'test_50.py::test_num[48]',
       'test_50.py::test_num[49]',
       'test_50.py::test_num[4]',
       'test_50.py::test_num[5]',
       'test_50.py::test_num[6]',
       'test_50.py::test_num[7]',
       'test_50.py::test_num[8]',
       'test_50.py::test_num[9]',
       'test_assert1.py::test_function',
       'test_assert2.py::test_set_comparison',
       'test_caching.py::test_function',
       'test_foocompare.py::test_compare']
    cache/stepwise contains:
      []
    example/value contains:
      42

    ========================== no tests ran in 0.12s ===========================

С параметром ``--cache-show`` принимается необязательный аргумент, чтобы указать глобальный шаблон
для фильтрации:

.. code-block:: pytest

    $ pytest --cache-show example/*
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    cachedir: $PYTHON_PREFIX/.pytest_cache
    ----------------------- cache values for 'example/*' -----------------------
    example/value contains:
      42

    ========================== no tests ran in 0.12s ===========================

Очистка содержимого кеша
-------------------------

Можно указать ``pytest`` очистить все файлы кеша и значения, добавив параметр ``--cache-clear``, например:

.. code-block:: bash

    pytest --cache-clear

Этот параметр рекомендуется использовать с серверами непрерывной интеграции(CI), где изоляция и корректность важнее
скорости.


Пошаговое выполнение
----------------------

В качестве альтернативы ``--lf -x``, особенно для случаев, когда вы ожидаете, что большая часть
набора тестов упадет, ``--sw``, ``--stepwise`` позволяют исправлять их по одному.
Набор тестов будет работать до первого падения, а затем остановится. При следующем вызове тесты
продолжатся с последнего упавшего теста, а затем продолжатся до следующего упавшего теста.
Вы можете использовать параметр ``--stepwise-skip``, чтобы проигнорировать первый упавший тест и вместо
этого остановить выполнение на втором упавшем тесте. Это полезно, если вы застряли на упавшем
тесте и просто хотите проигнорировать его, вернувшись к нему позже.
