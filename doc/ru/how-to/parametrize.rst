
.. _`test generators`:
.. _`parametrizing-tests`:
.. _`parametrized test functions`:
.. _`parametrize`:

.. _`parametrize-basics`:

Параметризация фикстур и тестовых функций
==========================================================================

``pytest`` обеспечивает параметризацию тестовых функций на нескольких уровнях:

- :py:func:`pytest.fixture` позволяет :ref:`parametrize fixture functions <fixture-parametrize>`.

* `@pytest.mark.parametrize`_ позволяет определить множество аргументов
  и фикстур для тестовой функции или класса.

* `pytest_generate_tests`_ позволяет определять пользовательские расширения
  и схемы параметризации.

.. _parametrizemark:
.. _`@pytest.mark.parametrize`:


``@pytest.mark.parametrize``: параметризация тестовых функций
---------------------------------------------------------------------

.. regendoc: wipe



    Несколько улучшений.

Встроенный декоратор :ref:`pytest.mark.parametrize ref`
позволяет параметризовать аргументы тестовых функций. Ниже приведен типичный пример тестовой функции,
реализующей проверку того, что определенный ввод приводит к ожидаемому выводу:

.. code-block:: python

    # листинг test_expectation.py
    import pytest


    @pytest.mark.parametrize("test_input,expected", [("3+5", 8), ("2+4", 6), ("6*9", 42)])
    def test_eval(test_input, expected):
        assert eval(test_input) == expected

Здесь декоратор ``@parametrize`` определяет три различных кортежа ``(test_input, expected)``,
так что функция ``test_eval`` будет запускаться три раза, используя их по очереди:

.. code-block:: pytest

    $ pytest
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 3 items

    test_expectation.py ..F                                              [100%]

    ================================= FAILURES =================================
    ____________________________ test_eval[6*9-42] _____________________________

    test_input = '6*9', expected = 42

        @pytest.mark.parametrize("test_input,expected", [("3+5", 8), ("2+4", 6), ("6*9", 42)])
        def test_eval(test_input, expected):
    >       assert eval(test_input) == expected
    E       AssertionError: assert 54 == 42
    E        +  where 54 = eval('6*9')

    test_expectation.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_expectation.py::test_eval[6*9-42] - AssertionError: assert 54...
    ======================= 1 failed, 2 passed in 0.12s ========================

.. note::

    Значения параметров передаются тестам как есть (без копирования).

    Например, если вы передаете список или словарь в качестве значения параметра, и код тестового кейса
    изменяет его, мутации будут отражены в последующих вызовах тестового кейса.

.. note::

    По умолчанию ``pytest`` экранирует любые не ASCII-символы, которые используются в строках unicode для
    параметризации, потому что существуют некоторые недостатки. Если вы хотите
    использовать строки unicode в параметризации и видеть их в терминале
    как есть (без экранирования), пропишите в файле ``pytest.ini`` следующее:

    .. code-block:: ini

        [pytest]
        disable_test_id_escaping_and_forfeit_all_rights_to_community_support = True

    При этом имейте в виду, что в некоторых ОС и при установке некоторых плагинов
    такое использование может приводить к неожиданным побочным эффектам и даже ошибкам,
    так что используйте это на свой страх и риск.


В примере выше только одна пара параметров приводит к падению теста.
И, как обычно, в трассировке можно увидеть входные (``input``)
и выходные (``output``) значения аргументов функции.

Обратите внимание, что маркер ``parametrize``  можно использовать также
и для классов и  модулей (см. :ref:`mark`)  и это также приведет к вызову
нескольких функций с разным набором аргументов.

Также есть возможность пометить отдельные тестовые экземпляры в параметризации,
например со встроенным ``mark.xfail``:

.. code-block:: python

    # листинг test_expectation.py
    import pytest


    @pytest.mark.parametrize(
        "test_input,expected",
        [("3+5", 8), ("2+4", 6), pytest.param("6*9", 42, marks=pytest.mark.xfail)],
    )
    def test_eval(test_input, expected):
        assert eval(test_input) == expected

Давайте запустим:

.. code-block:: pytest

    $ pytest
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 3 items

    test_expectation.py ..x                                              [100%]

    ======================= 2 passed, 1 xfailed in 0.12s =======================

Тот набор параметров, который раньше вызывал сбой, теперь помечается  как
``xfailed`` (ожидаемое падение).

Когда значения, передаваемые при параметризации, оказываются пустым
списком - например, если они динамически генерируются некоторой
функцией, то поведение ``pytest`` определяется опцией :confval:`empty_parameter_set_mark`.

Чтобы получить все комбинации нескольких параметризованных аргументов, вы можете складывать
``parametrize`` декораторы:

.. code-block:: python

    import pytest


    @pytest.mark.parametrize("x", [0, 1])
    @pytest.mark.parametrize("y", [2, 3])
    def test_foo(x, y):
        pass

Это запустит тест с аргументами, установленными в ``x=0/y=2``, ``x=1/y=2``,
``x=0/y=3`` и ``x=1/y=3``, используя параметры в порядке следования декораторов.

.. _`pytest_generate_tests`:

Базовый пример ``pytest_generate_tests``
---------------------------------------------

Иногда вам может потребоваться реализовать собственную схему параметризации или  некоторый динамизм
для определения параметров или области применения фикстуры. Для этого можно использовать hook-функцию
``pytest_generate_tests``, которая вызывается при сборке тестовой функции.
Через переданный ``metafunc``-объект можно запросить требуемый контекст тестов
и, самое главное, можно вызвать ``metafunc.parametrize()`` для параметризации.

Давайте предположим, что мы хотим запустить тест с использованием строковых входных данных, которые
нужно устанавливать с помощью новой опции командной строки ``pytest``. Давайте сначала напишем
простой тест, который принимает фикстуру ``stringinput`` в качестве аргумента:

.. code-block:: python

    # листинг test_strings.py


    def test_valid_string(stringinput):
        assert stringinput.isalpha()

Затем мы пропишем в файл ``conftest.py`` добавление опции командной строки и параметризацию нашей
тестовой функции:

.. code-block:: python

    # листинг conftest.py


    def pytest_addoption(parser):
        parser.addoption(
            "--stringinput",
            action="append",
            default=[],
            help="list of stringinputs to pass to test functions",
        )


    def pytest_generate_tests(metafunc):
        if "stringinput" in metafunc.fixturenames:
            metafunc.parametrize("stringinput", metafunc.config.getoption("stringinput"))

Теперь, если мы передадим две входных строки, наш тест будет выполнен дважды:

.. code-block:: pytest

    $ pytest -q --stringinput="hello" --stringinput="world" test_strings.py
    ..                                                                   [100%]
    2 passed in 0.12s

Let's also run with a stringinput that will lead to a failing test:

.. code-block:: pytest

    $ pytest -q --stringinput="!" test_strings.py
    F                                                                    [100%]
    ================================= FAILURES =================================
    ___________________________ test_valid_string[!] ___________________________

    stringinput = '!'

        def test_valid_string(stringinput):
    >       assert stringinput.isalpha()
    E       AssertionError: assert False
    E        +  where False = <built-in method isalpha of str object at 0xdeadbeef>()
    E        +    where <built-in method isalpha of str object at 0xdeadbeef> = '!'.isalpha

    test_strings.py:4: AssertionError
    ========================= short test summary info ==========================
    FAILED test_strings.py::test_valid_string[!] - AssertionError: assert False
    1 failed in 0.12s

Как и ожидалось, наш тест упал.

Если при запуске вы не укажете строковое значение, то тест будет пропущен, поскольку функция
``metafunc.parametrize()`` будет вызвана с пустым списком параметров:

.. code-block:: pytest

    $ pytest -q -rs test_strings.py
    s                                                                    [100%]
    ========================= short test summary info ==========================
    SKIPPED [1] test_strings.py: got empty parameter set ['stringinput'], function test_valid_string at $REGENDOC_TMPDIR/test_strings.py:2
    1 skipped in 0.12s

Обратите внимание, что при многократном вызове ``metafunc.parametrize`` с различными множествами
параметров, имена параметров в множестве не должны дублироваться, иначе возникнет ошибка.

More examples
-------------

Больше примеров можно увидеть здесь: :ref:`more parametrization examples <paramexamples>`.
