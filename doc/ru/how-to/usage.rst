
.. _usage:

Как вызвать pytest
==========================================

..  seealso:: :ref:`Complete pytest command-line flag reference <command-line-flags>`

Обычно pytest вызывается с помощью команды ``pytest``. Это выполнит все тесты во всех файлах, имена
которых соответствуют форме ``test_*.py`` или ``\*_test.py`` в текущем каталоге и его подкаталогах.
В более общем плане pytest следует стандартным тестовым правилам обнаружения :ref:`standard test discovery rules <test discovery>`.

..  seealso:: :ref:`invoke-other`


.. _select-tests:

Указание тестов для запуска
------------------------------

Pytest поддерживает несколько способов запуска и выбора тестов из командной строки.

**Запуск тестов в модуле**

.. code-block:: bash

    pytest test_mod.py

**Запуск тестов в каталоге**

.. code-block:: bash

    pytest testing/

**Запуск тестов по ключевым словам**

.. code-block:: bash

    pytest -k "MyClass and not method"

Эта команда запустит тесты, имена которых удовлетворяют заданному строковому выражению (без учета
регистра). Строковые выражения могут включать операторы ``Python``, которые используют имена файлов,
классов и функций в качестве переменных. В приведенном выше примере будет запущен тест
``MyClass.test_something``, но не  будет запущен тест ``TestMyClass.test_method_simple``.

.. _nodeids:

**Запуск тестов по идентификаторам узлов**

Каждому собранному тесту присваивается уникальный идентификатор ``nodeid``,
который состоит из имени файла модуля, за которым следуют спецификаторы,
такие как имена классов, имена функций и параметры из параметризации, разделенные символами ``::``:

Чтобы запустить конкретный тест из модуля, выполните:

.. code-block:: bash

    pytest test_mod.py::test_func


Другой пример указания метода тестирования в командной строке:

.. code-block:: bash

    pytest test_mod.py::TestClass::test_method

**Запуск маркированных тестов**

.. code-block:: bash

    pytest -m slow

Будут запущены тесты, помеченные декоратором ``@pytest.mark.slow``.

Подробнее см. :ref:`marks <mark>`.

**Запуск тестов из пакетов**

.. code-block:: bash

    pytest --pyargs pkg.testing

Будет импортирован пакет ``pkg.testing``, и его расположение в файловой системе
будет использовано для поиска и запуска тестов.


Возможные коды выхода
--------------------------------------------------------------

Запуск ``pytest`` может привести к шести различным кодам:

:Exit code 0: Все тесты собраны и пройдены успешно.
:Exit code 1: Были собраны и запущены тесты, но некоторые из них не прошли.
:Exit code 2: Выполнение теста было прервано пользователем.
:Exit code 3: Произошла внутренняя ошибка при выполнении тестов.
:Exit code 4: Ошибка использования командной строки pytest.
:Exit code 5: Никаких тестов не собрано.

Они перечислены в :class:`pytest.ExitCode`. Выходные коды, являющиеся частью общедоступного API, могут быть
импортированы и доступны напрямую с помощью:

.. code-block:: python

    from pytest import ExitCode

.. note::

    Если вы хотите настроить код выхода в некоторых сценариях, особенно когда тесты не собираются, рассмотрите возможность использования плагина
    `pytest-custom_exit_code <https://github.com/yashtodi94/pytest-custom_exit_code>`__.


Получение справки по версии, названиям опций, переменным среды
-----------------------------------------------------------------

.. code-block:: bash

    pytest --version   # shows where pytest was imported from
    pytest --fixtures  # show available builtin function arguments
    pytest -h | --help # show help on command line and config file options



.. _maxfail:

Остановка после первых (или N) падений
---------------------------------------------------

Чтобы остановить процесс тестирования после первых N падений, используются параметры:

.. code-block:: bash

    pytest -x           # остановка после первого упавшего теста
    pytest --maxfail=2  # остановка после первых двух упавших тестов


Управление выводом pytest
--------------------------

Изменение вывода сообщений трассировки
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Примеры вывода:

.. code-block:: bash

    pytest --showlocals # показать локальные переменные в трассировке
    pytest -l           # показать локальные переменные(краткий вариант)

    pytest --tb=auto    # (по умолчанию) "расширенный" вывод для первого и
                         # последнего сообщений, и "короткий" для остальных
    pytest --tb=long    # исчерпывающий, подробный формат сообщений
    pytest --tb=short   # сокращенный формат сообщений
    pytest --tb=line    # только одна строка на падение
    pytest --tb=native  # стандартный формат из библиотеки Python
    pytest --tb=no      # никаких сообщений

Использование ``--full-trace`` приводит к тому, что при ошибке печатаются очень длинные
трассировки (длиннее, чем при ``--tb=long``). Параметр также гарантирует, что сообщения
трассировки будут напечатаны при **прерывании выполнения c клавиатуры** с помощью Ctrl+C.
Это очень полезно, если тесты занимают слишком много времени, и вы прерываете их
с клавиатуры с помощью Ctrl+C, чтобы узнать, где они зависли. По умолчанию при прерывании
вывод не будет показан (поскольку исключение ``KeyboardInterrupt`` будет поймано ``pytest``).
Используя этот параметр, вы можете быть уверены, что увидите трассировку.


Verbosity
---------

Флаг ``-v`` контролирует подробность вывода pytest в различных аспектах: прогресс сеанса тестирования,
подробности assert, когда тесты падают, детали фикстур с ``--fixtures``, и т.д.

.. regendoc:wipe

Рассмотрим этот простой файл:

.. code-block:: python

    # листинг test_verbosity_example.py
    def test_ok():
        pass


    def test_words_fail():
        fruits1 = ["banana", "apple", "grapes", "melon", "kiwi"]
        fruits2 = ["banana", "apple", "orange", "melon", "kiwi"]
        assert fruits1 == fruits2


    def test_numbers_fail():
        number_to_text1 = {str(x): x for x in range(5)}
        number_to_text2 = {str(x * 10): x * 10 for x in range(5)}
        assert number_to_text1 == number_to_text2


    def test_long_text_fail():
        long_text = "Lorem ipsum dolor sit amet " * 10
        assert "hello world" in long_text

Выполнение pytest обычно дает нам этот результат (мы пропускаем заголовок, чтобы сосредоточиться на остальном):

.. code-block:: pytest

    $ pytest --no-header
    =========================== test session starts ===========================
    collected 4 items

    test_verbosity_example.py .FFF                                       [100%]

    ================================ FAILURES =================================
    _____________________________ test_words_fail _____________________________

        def test_words_fail():
            fruits1 = ["banana", "apple", "grapes", "melon", "kiwi"]
            fruits2 = ["banana", "apple", "orange", "melon", "kiwi"]
    >       assert fruits1 == fruits2
    E       AssertionError: assert ['banana', 'a...elon', 'kiwi'] == ['banana', 'a...elon', 'kiwi']
    E         At index 2 diff: 'grapes' != 'orange'
    E         Use -v to get the full diff

    test_verbosity_example.py:8: AssertionError
    ____________________________ test_numbers_fail ____________________________

        def test_numbers_fail():
            number_to_text1 = {str(x): x for x in range(5)}
            number_to_text2 = {str(x * 10): x * 10 for x in range(5)}
    >       assert number_to_text1 == number_to_text2
    E       AssertionError: assert {'0': 0, '1':..., '3': 3, ...} == {'0': 0, '10'...'30': 30, ...}
    E         Omitting 1 identical items, use -vv to show
    E         Left contains 4 more items:
    E         {'1': 1, '2': 2, '3': 3, '4': 4}
    E         Right contains 4 more items:
    E         {'10': 10, '20': 20, '30': 30, '40': 40}
    E         Use -v to get the full diff

    test_verbosity_example.py:14: AssertionError
    ___________________________ test_long_text_fail ___________________________

        def test_long_text_fail():
            long_text = "Lorem ipsum dolor sit amet " * 10
    >       assert "hello world" in long_text
    E       AssertionError: assert 'hello world' in 'Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ips... sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet '

    test_verbosity_example.py:19: AssertionError
    ========================= short test summary info =========================
    FAILED test_verbosity_example.py::test_words_fail - AssertionError: asser...
    FAILED test_verbosity_example.py::test_numbers_fail - AssertionError: ass...
    FAILED test_verbosity_example.py::test_long_text_fail - AssertionError: a...
    ======================= 3 failed, 1 passed in 0.08s =======================

Заметьте:

* Каждый тест внутри файла отображается одним символом в выводе: ``.`` для прохождения, ``F`` для падения.
* ``test_words_fail`` упал, и нам показывают краткую сводку, указывающую, что индекс 2 в двух списках различается.
* ``test_numbers_fail`` упал, и нам показана сводка различий между левыми и правыми элементами словаря. Идентичные позиции пропущены.
* ``test_long_text_fail`` упал, а правая часть оператора ``in`` обрезается с помощью``...``
  потому что он длиннее внутреннего порога (в настоящее время 240 символов).

Теперь мы можем увеличить вывод pytest:

.. code-block:: pytest

    $ pytest --no-header -v
    =========================== test session starts ===========================
    collecting ... collected 4 items

    test_verbosity_example.py::test_ok PASSED                            [ 25%]
    test_verbosity_example.py::test_words_fail FAILED                    [ 50%]
    test_verbosity_example.py::test_numbers_fail FAILED                  [ 75%]
    test_verbosity_example.py::test_long_text_fail FAILED                [100%]

    ================================ FAILURES =================================
    _____________________________ test_words_fail _____________________________

        def test_words_fail():
            fruits1 = ["banana", "apple", "grapes", "melon", "kiwi"]
            fruits2 = ["banana", "apple", "orange", "melon", "kiwi"]
    >       assert fruits1 == fruits2
    E       AssertionError: assert ['banana', 'a...elon', 'kiwi'] == ['banana', 'a...elon', 'kiwi']
    E         At index 2 diff: 'grapes' != 'orange'
    E         Full diff:
    E         - ['banana', 'apple', 'orange', 'melon', 'kiwi']
    E         ?                      ^  ^^
    E         + ['banana', 'apple', 'grapes', 'melon', 'kiwi']
    E         ?                      ^  ^ +

    test_verbosity_example.py:8: AssertionError
    ____________________________ test_numbers_fail ____________________________

        def test_numbers_fail():
            number_to_text1 = {str(x): x for x in range(5)}
            number_to_text2 = {str(x * 10): x * 10 for x in range(5)}
    >       assert number_to_text1 == number_to_text2
    E       AssertionError: assert {'0': 0, '1':..., '3': 3, ...} == {'0': 0, '10'...'30': 30, ...}
    E         Omitting 1 identical items, use -vv to show
    E         Left contains 4 more items:
    E         {'1': 1, '2': 2, '3': 3, '4': 4}
    E         Right contains 4 more items:
    E         {'10': 10, '20': 20, '30': 30, '40': 40}
    E         Full diff:
    E         - {'0': 0, '10': 10, '20': 20, '30': 30, '40': 40}...
    E
    E         ...Full output truncated (3 lines hidden), use '-vv' to show

    test_verbosity_example.py:14: AssertionError
    ___________________________ test_long_text_fail ___________________________

        def test_long_text_fail():
            long_text = "Lorem ipsum dolor sit amet " * 10
    >       assert "hello world" in long_text
    E       AssertionError: assert 'hello world' in 'Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet '

    test_verbosity_example.py:19: AssertionError
    ========================= short test summary info =========================
    FAILED test_verbosity_example.py::test_words_fail - AssertionError: asser...
    FAILED test_verbosity_example.py::test_numbers_fail - AssertionError: ass...
    FAILED test_verbosity_example.py::test_long_text_fail - AssertionError: a...
    ======================= 3 failed, 1 passed in 0.07s =======================

Обратите внимание, что:

* Каждый тест внутри файла получает свою собственную строку на выходе.
* ``test_words_fail`` теперь показывает два списка ошибок полностью, в дополнение к тому, какой индекс отличается.
* ``test_numbers_fail`` теперь показывает текстовое различие двух словарей, усеченное.
* ``test_long_text_fail`` больше не усекает правую часть оператора ``in``, потому что внутренний
  порог усечения теперь больше (2400 символов в настоящее время).

Теперь, если мы еще больше увеличим многословность:

.. code-block:: pytest

    $ pytest --no-header -vv
    =========================== test session starts ===========================
    collecting ... collected 4 items

    test_verbosity_example.py::test_ok PASSED                            [ 25%]
    test_verbosity_example.py::test_words_fail FAILED                    [ 50%]
    test_verbosity_example.py::test_numbers_fail FAILED                  [ 75%]
    test_verbosity_example.py::test_long_text_fail FAILED                [100%]

    ================================ FAILURES =================================
    _____________________________ test_words_fail _____________________________

        def test_words_fail():
            fruits1 = ["banana", "apple", "grapes", "melon", "kiwi"]
            fruits2 = ["banana", "apple", "orange", "melon", "kiwi"]
    >       assert fruits1 == fruits2
    E       AssertionError: assert ['banana', 'apple', 'grapes', 'melon', 'kiwi'] == ['banana', 'apple', 'orange', 'melon', 'kiwi']
    E         At index 2 diff: 'grapes' != 'orange'
    E         Full diff:
    E         - ['banana', 'apple', 'orange', 'melon', 'kiwi']
    E         ?                      ^  ^^
    E         + ['banana', 'apple', 'grapes', 'melon', 'kiwi']
    E         ?                      ^  ^ +

    test_verbosity_example.py:8: AssertionError
    ____________________________ test_numbers_fail ____________________________

        def test_numbers_fail():
            number_to_text1 = {str(x): x for x in range(5)}
            number_to_text2 = {str(x * 10): x * 10 for x in range(5)}
    >       assert number_to_text1 == number_to_text2
    E       AssertionError: assert {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4} == {'0': 0, '10': 10, '20': 20, '30': 30, '40': 40}
    E         Common items:
    E         {'0': 0}
    E         Left contains 4 more items:
    E         {'1': 1, '2': 2, '3': 3, '4': 4}
    E         Right contains 4 more items:
    E         {'10': 10, '20': 20, '30': 30, '40': 40}
    E         Full diff:
    E         - {'0': 0, '10': 10, '20': 20, '30': 30, '40': 40}
    E         ?            -    -    -    -    -    -    -    -
    E         + {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4}

    test_verbosity_example.py:14: AssertionError
    ___________________________ test_long_text_fail ___________________________

        def test_long_text_fail():
            long_text = "Lorem ipsum dolor sit amet " * 10
    >       assert "hello world" in long_text
    E       AssertionError: assert 'hello world' in 'Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet Lorem ipsum dolor sit amet '

    test_verbosity_example.py:19: AssertionError
    ========================= short test summary info =========================
    FAILED test_verbosity_example.py::test_words_fail - AssertionError: asser...
    FAILED test_verbosity_example.py::test_numbers_fail - AssertionError: ass...
    FAILED test_verbosity_example.py::test_long_text_fail - AssertionError: a...
    ======================= 3 failed, 1 passed in 0.07s =======================

Обратите внимание, что:

* Каждый тест внутри файла получает свою собственную строку на выходе.
* ``test_words_fail`` дает тот же результат, что и раньше, в этом случае.
* ``test_numbers_fail`` теперь показывает полную текстовую разницу двух словарей.
* ``test_long_text_fail`` также не обрезается с правой стороны, как раньше, но теперь pytest вообще
не обрезает какой-либо текст, независимо от его размера.

Это были примеры того, как многословность влияет на нормальный вывод тестового сеанса, но многословность также
используется в других ситуациях, например, вам показаны даже фикстуры, которые начинаются с ``_`` если вы используете ``pytest --fixtures -v``.

Использование более высоких уровней детализации (``-vvv``, ``-vvvv``, ...) поддерживается, но на данный момент не
имеет никакого эффекта в самом pytest, однако некоторые плагины могут использовать более высокий уровень детализации.

.. _`pytest.detailed_failed_tests_usage`:

Детализация сводного отчета
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Флаг ``-r`` можно использовать для отображения "краткой сводной информации по тестированию"
в конце тестового сеанса, что упрощает получение четкой картины всех сбоев, пропусков, xfails и т. д.

По умолчанию для списка сбоев и ошибок используется добавочная комбинация ``fE``.

.. regendoc:wipe

Пример:

.. code-block:: python

    # листинг test_example.py
    import pytest


    @pytest.fixture
    def error_fixture():
        assert 0


    def test_ok():
        print("ok")


    def test_fail():
        assert 0


    def test_error(error_fixture):
        pass


    def test_skip():
        pytest.skip("skipping this test")


    def test_xfail():
        pytest.xfail("xfailing this test")


    @pytest.mark.xfail(reason="always xfail")
    def test_xpass():
        pass


.. code-block:: pytest

    $ pytest -ra
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 6 items

    test_example.py .FEsxX                                               [100%]

    ================================== ERRORS ==================================
    _______________________ ERROR at setup of test_error _______________________

        @pytest.fixture
        def error_fixture():
    >       assert 0
    E       assert 0

    test_example.py:6: AssertionError
    ================================= FAILURES =================================
    ________________________________ test_fail _________________________________

        def test_fail():
    >       assert 0
    E       assert 0

    test_example.py:14: AssertionError
    ========================= short test summary info ==========================
    SKIPPED [1] test_example.py:22: skipping this test
    XFAIL test_example.py::test_xfail
      reason: xfailing this test
    XPASS test_example.py::test_xpass always xfail
    ERROR test_example.py::test_error - assert 0
    FAILED test_example.py::test_fail - assert 0
    == 1 failed, 1 passed, 1 skipped, 1 xfailed, 1 xpassed, 1 error in 0.12s ===

Параметр ``-r`` принимает ряд символов после себя. Использованный выше символ ``а`` означает
“все, кроме успешных".

Вот полный список доступных символов, которые могут быть использованы:

 - ``f`` - упавшие
 - ``E`` - ошибки
 - ``s`` - пропущенные
 - ``x`` - тесты XFAIL
 - ``X`` - тесты XPASS
 - ``p`` - успешные
 - ``P`` - успешные с выводом

Есть и специальные символы для пропуска отдельных групп:

 - ``a`` - выводить все, кроме ``pP``
 - ``A`` - выводить все
 - ``N`` - ничего не выводить (может быть полезным, поскольку по умолчанию используется комбинация ``fE``).

Можно использовать более одного символа. Например, для того, чтобы увидеть только
упавшие и пропущенные тесты, можно выполнить:

.. code-block:: pytest

    $ pytest -rfs
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 6 items

    test_example.py .FEsxX                                               [100%]

    ================================== ERRORS ==================================
    _______________________ ERROR at setup of test_error _______________________

        @pytest.fixture
        def error_fixture():
    >       assert 0
    E       assert 0

    test_example.py:6: AssertionError
    ================================= FAILURES =================================
    ________________________________ test_fail _________________________________

        def test_fail():
    >       assert 0
    E       assert 0

    test_example.py:14: AssertionError
    ========================= short test summary info ==========================
    FAILED test_example.py::test_fail - assert 0
    SKIPPED [1] test_example.py:22: skipping this test
    == 1 failed, 1 passed, 1 skipped, 1 xfailed, 1 xpassed, 1 error in 0.12s ===

Использование «p» перечисляет пройденные тесты, в то время как ``p`` добавляет дополнительный раздел «PASSES» с
теми тестами, которые прошли, но перехватили вывод:

.. code-block:: pytest

    $ pytest -rpP
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 6 items

    test_example.py .FEsxX                                               [100%]

    ================================== ERRORS ==================================
    _______________________ ERROR at setup of test_error _______________________

        @pytest.fixture
        def error_fixture():
    >       assert 0
    E       assert 0

    test_example.py:6: AssertionError
    ================================= FAILURES =================================
    ________________________________ test_fail _________________________________

        def test_fail():
    >       assert 0
    E       assert 0

    test_example.py:14: AssertionError
    ================================== PASSES ==================================
    _________________________________ test_ok __________________________________
    --------------------------- Captured stdout call ---------------------------
    ok
    ========================= short test summary info ==========================
    PASSED test_example.py::test_ok
    == 1 failed, 1 passed, 1 skipped, 1 xfailed, 1 xpassed, 1 error in 0.12s ===


Создание файлов формата resultlog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы создать машиночитаемые файлы результатов в виде простого текста, вы можете выполнить:

.. code-block:: bash

    pytest --resultlog=path

и посмотрите на контент в локации ``path``. Такие файлы используются, например, на веб-странице `PyPy-test`_, чтобы
показать результаты тестирования нескольких ревизий.

.. warning::

    Этот параметр используется редко и планируется удалить в pytest 6.0.

    Если вы используете эту опцию, рассмотрите возможность использования нового `pytest-reportlog <https://github.com/pytest-dev/pytest-reportlog>`__ plugin instead.

    См. `the deprecation docs <https://docs.pytest.org/en/stable/deprecations.html#result-log-result-log>`__
    для большей информации.


.. _`PyPy-test`: http://buildbot.pypy.org/summary


Отправка отчетов на сервис pastebin
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Создание ссылки для каждого упавшего теста**:

.. code-block:: bash

    pytest --pastebin=failed

Эта команда отправит информацию о прохождении теста на удаленный сервис регистрации и сгенерирует ссылку для
каждого падения. Тесты можно отбирать как обычно, или, например, добавить ``-x``, если вы хотите отправить данные
по конкретному упавшему тесту.

**Создание ссылки для лога тестовой сессии**:

.. code-block:: bash

    pytest --pastebin=all

В настоящее время реализована регистрация только в сервисе http://bpaste.net.

.. versionchanged:: 5.2

Если по каким-то причинам не удалось создать ссылку, вместо падения всего тестового набора
генерируется предупреждение.


.. _pdb-option:

Использование PDB_ (отладчика Python) с pytest
----------------------------------------------------------

Запуск отладчика PDB_ (Python Debugger) при падении тестов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _PDB: http://docs.python.org/library/pdb.html

``python`` содержит встроенный отладчик PDB_ (Python Debugger). ``pytest`` позволяет
запустить отладчик с помощью параметра командной строки:

.. code-block:: bash

    pytest --pdb

Использование параметра позволяет запускать отладчик при каждом падении теста
(или прерывании его с клавиатуры). Часто хочется сделать это для первого же упавшего теста,
чтобы понять причину его падения:

.. code-block:: bash

    pytest -x --pdb   # вызывает отладчик при первом падении и завершает тестовую сессию
    pytest --pdb --maxfail=3  # вызывает отладчик для первых трех падений

Обратите внимание, что при любом падении информация об исключении сохраняется в
``sys.last_value``, ``sys.last_type`` и ``sys.last_traceback``. При интерактивном использовании
это позволяет перейти к отладке после падения с помощью любого инструмента отладки.
Можно также вручную получить доступ к информации об исключениях, например::

    >>> import sys
    >>> sys.last_traceback.tb_lineno
    42
    >>> sys.last_value
    AssertionError('assert result == "ok"',)


.. _trace-option:

Переход на PDB_ в начале теста
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``pytest`` позволяет сразу запустить отладчик PDB_ в начале каждого теста с помощью параметра командной строки:

.. code-block:: bash

    pytest --trace

В этом случае отладчик будет вызываться при запуске каждого теста.

.. _breakpoints:

Установка точек останова
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded: 2.4.0

Чтобы установить точку останова, вызовите в коде ``import pdb;pdb.set_trace()``, и
``pytest`` автоматически отключит перехват вывода для этого теста, при этом:

* на перехват вывода в других тестах это не повлияет;
* весь перехваченный ранее вывод будет обработан как есть;
* перехват вывода возобновится после завершения отладочной сессии (с помощью команды ``continue``).


.. _`breakpoint-builtin`:

Использование встроенной функции breakpoint
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Python 3.7`` содержит встроенную функцию ``breakpoint()``.
``pytest`` поддерживает использование ``breakpoint()`` следующим образом:

 - если вызывается ``breakpoint()``, и при этом переменная ``PYTHONBREAKPOINT`` установлена в
   значение по умолчанию, ``pytest`` использует расширяемый отладчик PDB_ вместо
   системного;
 - когда тестирование будет завершено, система снова будет использовать отладчик ``Pdb`` по умолчанию;
 - если ``pytest`` вызывается с опцией ``--pdb`` то расширяемый отладчик PDB_ используется
   как для функции ``breakpoint()``, так и для упавших тестов/необработанных исключений;
 - для определения пользовательского класса отладчика можно использовать ``--pdbcls``.


.. _durations:

Профилирование продолжительности выполнения теста
-------------------------------------------------------

.. versionchanged:: 6.0

Чтобы получить список из 10 самых медленных тестов длительностью более 1,0 с:

.. code-block:: bash

    pytest --durations=10 --durations-min=1.0

По умолчанию pytest не будет показывать слишком малую продолжительность теста (<0,005 с), если в командной строке не передан параметр ``-vv``.


.. _faulthandler:

Модуль ``faulthandler``
-----------------------

.. versionadded:: 5.0

Стандартный модуль `faulthandler <https://docs.python.org/3/library/faulthandler.html>`__
можно использовать для сброса трассировок ``Python`` при ошибке или по истечении времени ожидания.

При запуске ``pytest`` модуль автоматически подключается, если только в командной строке
не используется опция ``-p no:faulthandler``.

Также :confval:`faulthandler_timeout=X<faulthandler_timeout>` параметр конфигурации может использоваться
для сброса трассировки всех потоков, если для завершения теста требуется больше, чем ``X`` секунд
(недоступно в Windows).

.. note::

    Эта функциональность была интегрирована из внешнего плагина
    `pytest-faulthandler <https://github.com/pytest-dev/pytest-faulthandler>`__ , с двумя небольшими изменениями:

    * чтобы ее отключить, используйте ``-p no:faulthandler`` вместо ``--no-faulthandler``: первый может
      быть использован с любым плагином, так что это экономит один вариант.

    * опция командной строки ``--faulthandler-timeout`` стала вариантом конфигурации
      :confval:`faulthandler_timeout`. Ее по-прежнему можно настроить из команндной строки,
      используя ``-o faulthandler_timeout=X``.


.. _unraisable:

Предупреждение о необрабатываемых исключениях и необработанных исключениях потоков
--------------------------------------------------------------------------------------

.. versionadded:: 6.2

.. note::

    Эти функции работают только на Python>=3.8.

Необработанные исключения - это исключения, которые возникают в ситуации, когда они не могут
быть переданы вызывающему. Чаще всего возникает исключение, возникающее в реализации :meth:`__del__ <object.__del__>`.

Необработанные исключения потока - это исключения, которые возникают в :class:`~threading.Thread`,
но не обрабатываются, вызывая несанкционированное завершение потока.

Оба типа исключений обычно считаются ошибками, но могут остаться незамеченными, потому что они не
вызывают сбой самой программы. Pytest обнаруживает эти условия и выдает предупреждение, которое
отображается в сводке тестового запуска.

Плагины автоматически включаются для запуска pytest, если только
``-p no:unraisableexception`` (для неприемлемых исключений) и
``-p no:threadexception`` (для исключений потоков) параметры указаны в командной строке.

Предупреждения можно отключить выборочно с помощью меток :ref:`pytest.mark.filterwarnings ref`.
Категории предупреждений: :class:`pytest.PytestUnraisableExceptionWarning` и
:class:`pytest.PytestUnhandledThreadExceptionWarning`.


Создание файлов формата JUnitXML
----------------------------------------------------

Чтобы создать результирующие файлы в формате, понятном  Jenkins_
или другому серверу непрерывной интеграции, используйте вызов:

.. code-block:: bash

    pytest --junitxml=path

для создания xml-файл по указанному пути ``path``.



Чтобы задать имя корневого xml-элемента для набора тестов, можно настроить параметр
``junit_suite_name`` в конфигурационном файле:

.. code-block:: ini

    [pytest]
    junit_suite_name = my_suite

.. versionadded:: 4.0

Спецификация JUnit XML, по-видимому, указывает, что атрибут ``"time"`` должен сообщать
об общем времени выполнения теста, включая выполнение setup- и teardown- методов
(`1 <http://windyroad.com.au/dl/Open%20Source/JUnit.xsd>`_, `2
<https://www.ibm.com/support/knowledgecenter/en/SSQ2R2_14.1.0/com.ibm.rsar.analysis.codereview.cobol.doc/topics/cac_useresults_junit.html>`_).
Это поведение ``pytest`` по умолчанию. Чтобы вместо этого сообщать только о длительности вызовов,
настройте параметр ``junit_duration_report`` следующим образом:

.. code-block:: ini

    [pytest]
    junit_duration_report = call

.. _record_property example:

record_property
~~~~~~~~~~~~~~~~~

Чтобы записать дополнительную информацию для теста, используйте фикстуру ``record_property``:

.. code-block:: python

    def test_function(record_property):
        record_property("example_key", 1)
        assert True

Такая запись добавит дополнительное свойство ``example_key="1"`` к сгенерированному тегу ``testcase``:

.. code-block:: xml

    <testcase classname="test_function" file="test_function.py" line="0" name="test_function" time="0.0009">
      <properties>
        <property name="example_key" value="1" />
      </properties>
    </testcase>

Эту функциональность также можно использовать совместно с пользовательскими маркерами:

.. code-block:: python

    # листинг conftest.py


    def pytest_collection_modifyitems(session, config, items):
        for item in items:
            for marker in item.iter_markers(name="test_id"):
                test_id = marker.args[0]
                item.user_properties.append(("test_id", test_id))

И в тесте:

.. code-block:: python

    # листинг test_function.py
    import pytest


    @pytest.mark.test_id(1501)
    def test_function():
        assert True

В результате получится:

.. code-block:: xml

    <testcase classname="test_function" file="test_function.py" line="0" name="test_function" time="0.0009">
      <properties>
        <property name="test_id" value="1501" />
      </properties>
    </testcase>

.. warning::

    Please note that using this feature will break schema verifications for the latest JUnitXML schema.
    This might be a problem when used with some CI servers.


record_xml_attribute
~~~~~~~~~~~~~~~~~~~~~~~

Чтобы добавить дополнительный атрибут в элемент ``testcase``, можно использовать
фикстуру ``record_xml_attribute``. Ее также можно использовать для переопределения
существующих значений:

.. code-block:: python

    def test_function(record_xml_attribute):
        record_xml_attribute("assertions", "REQ-1234")
        record_xml_attribute("classname", "custom_classname")
        print("hello world")
        assert True

В отличие от ``record_property``, дочерний элемент в данном случае не добавляется.
Вместо этого в элемент ``testcase`` будет добавлен атрибут ``assertions="REQ-1234"``,
а значение атрибута ``classname`` по умолчанию будет заменено на ``"classname=custom_classname"``:

.. code-block:: xml

    <testcase classname="custom_classname" file="test_function.py" line="0" name="test_function" time="0.003" assertions="REQ-1234">
        <system-out>
            hello world
        </system-out>
    </testcase>

.. warning::

    ``record_xml_attribute`` пока используется в режиме эксперимента, и в будущем может быть
    заменен чем-то более мощным и/или общим. Однако сама функциональность как таковая будет сохранена.

    Использование ``record_xml_attribute`` поверх ``record_xml_property``  может помочь при использовании инструментов ci для
    анализа xml-отчета. Однако некоторые парсеры довольно строго относятся к разрешенным элементам и
    атрибутам. Многие инструменты используют схему xsd (как в примере ниже) для проверки входящего xml.
    Убедитесь, что вы используете имена атрибутов, разрешенные вашим парсером.

    Ниже представлена схема, которую использует ``Jenkins`` для валидации xml-отчетов:

    .. code-block:: xml

        <xs:element name="testcase">
            <xs:complexType>
                <xs:sequence>
                    <xs:element ref="skipped" minOccurs="0" maxOccurs="1"/>
                    <xs:element ref="error" minOccurs="0" maxOccurs="unbounded"/>
                    <xs:element ref="failure" minOccurs="0" maxOccurs="unbounded"/>
                    <xs:element ref="system-out" minOccurs="0" maxOccurs="unbounded"/>
                    <xs:element ref="system-err" minOccurs="0" maxOccurs="unbounded"/>
                </xs:sequence>
                <xs:attribute name="name" type="xs:string" use="required"/>
                <xs:attribute name="assertions" type="xs:string" use="optional"/>
                <xs:attribute name="time" type="xs:string" use="optional"/>
                <xs:attribute name="classname" type="xs:string" use="optional"/>
                <xs:attribute name="status" type="xs:string" use="optional"/>
            </xs:complexType>
        </xs:element>

.. warning::

    Обратите внимание, что использование этой функции нарушит проверку схемы для последней схемы
    JUnitXML. Это может быть проблемой при использовании с некоторыми серверами CI.

.. _record_testsuite_property example:

record_testsuite_property
^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 4.5

Если вы хотите добавить узел свойств на уровне набора тестов, который может содержать свойства,
относящиеся ко всем тестам, вы можете использовать фикстуру с привязкой к сеансу
``record_testsuite_property``:

Фикстура ``record_testsuite_property`` с привязкой к сеансу может использоваться для добавления свойств,
относящихся ко всем тестам.

.. code-block:: python

    import pytest


    @pytest.fixture(scope="session", autouse=True)
    def log_global_env_facts(record_testsuite_property):
        record_testsuite_property("ARCH", "PPC")
        record_testsuite_property("STORAGE_TYPE", "CEPH")


    class TestMe:
        def test_foo(self):
            assert True

Этой фикстуре передаются имя (``name``) и значение (``value``) тэга ``<property>``, который
добавляется на уровне тестового набора для генерируемого xml-файла:

.. code-block:: xml

    <testsuite errors="0" failures="0" name="pytest" skipped="0" tests="1" time="0.006">
      <properties>
        <property name="ARCH" value="PPC"/>
        <property name="STORAGE_TYPE" value="CEPH"/>
      </properties>
      <testcase classname="test_me.TestMe" file="test_me.py" line="16" name="test_foo" time="0.000243663787842"/>
    </testsuite>

``name`` должно быть строкой, а  ``value`` будет преобразовано в строку и корректно экранировано.

В отличие от случаев использования `record_property`_ и `record_xml_attribute`_
созданный xml-файл будет совместим с последним стандартом ``xunit``.


Управление загрузкой плагинов
-------------------------------

Плагины ранней загрузки
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В командной строке можно явно подгрузить какой-либо внутренний или внешний плагин, используя опцию ``-p``::

    pytest -p mypluginmodule

Опция принимает параметр ``name``, который может быть:

* Полным именем модуля, записанным через точку, например ``myproject.plugins``. Имя должно быть импортируемым.
* "Входным" именем плагина, которое передается в ``setuptools`` при регистрации плагина. К примеру, чтобы подгрузить
  `pytest-cov <https://pypi.org/project/pytest-cov/>`__ , нужно использовать::

    pytest -p pytest_cov


Отключение плагинов
~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы отключить загрузку определенных плагинов во время вызова, используйте опцию ``-p`` с префиксом ``no:``.

Пример: чтобы отключить загрузку плагина ``doctest``, который отвечает за выполнение
тестов из строк "docstring", вызовите  ``pytest`` следующим образом:

.. code-block:: bash

    pytest -p no:doctest


.. _invoke-other:

Другие способы вызова pytest
-----------------------------------------------------

.. _invoke-python:

Вызов pytest через ``python -m pytest``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вы можете вызвать тестирование через интерпретатор Python из командной строки:

.. code-block:: text

    python -m pytest [...]

Это почти эквивалентно прямому вызову сценария командной строки ``pytest [...]``, за исключением того,
что вызов через ``python`` также добавит текущий каталог в ``sys.path``.


.. _`pytest.main-usage`:

Вызов ``pytest`` из кода Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``pytest`` можно вызвать прямо в коде Python:

.. code-block:: python

    pytest.main()

Такой способ эквивалентен вызову "pytest" из командной строки.
В этом случае вместо исключения ``SystemExit`` возвращается статус завершения.
Можно также передавать параметры и опции:

.. code-block:: python

    pytest.main(["-x", "mytestdir"])

Вы можете указать дополнительные плагины в ``pytest.main``:

.. code-block:: python

    # листинг myinvoke.py
    import pytest


    class MyPlugin:
        def pytest_sessionfinish(self):
            print("*** test run reporting finishing")


    pytest.main(["-qq"], plugins=[MyPlugin()])

Запуск покажет, что ``MyPlugin`` был добавлен, и его хук был вызван:

.. code-block:: pytest

    $ python myinvoke.py
    .FEsxX.                                                              [100%]*** test run reporting finishing

    ================================== ERRORS ==================================
    _______________________ ERROR at setup of test_error _______________________

        @pytest.fixture
        def error_fixture():
    >       assert 0
    E       assert 0

    test_example.py:6: AssertionError
    ================================= FAILURES =================================
    ________________________________ test_fail _________________________________

        def test_fail():
    >       assert 0
    E       assert 0

    test_example.py:14: AssertionError
    ========================= short test summary info ==========================
    FAILED test_example.py::test_fail - assert 0
    ERROR test_example.py::test_error - assert 0

.. note::

    Вызов ``pytest.main()`` приводит к тому, что импортируются не только тесты,
    но и все модули, которые они используют. Из-за механизма кэширования импорта
    ``Python`` последующие вызовы ``pytest.main()`` из того же процесса не будут учитывать
    изменения в файлах, внесенные между вызовами. Поэтому не рекомендуется многократное
    использование ``pytest.main()`` в одном и том же процессе (например, при перезапуске тестов).

.. _jenkins: http://jenkins-ci.org/
