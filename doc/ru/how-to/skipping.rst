.. _`skip and xfail`:

.. _skipping:

Как использовать ``Skip`` и ``xfail`` для работы с тестами, которые не могут быть пройдены
==============================================================================================

Тестовые функции, которые не могут быть запущены на определенных платформах
или от которых вы ожидаете сбоя, можно пометить так, чтобы ``pytest``
работал с ними соответствующим образом и представлял сводку сеанса тестирования,
считая набор тестов пройденным и помечая его *зеленым*.

**skip** (пропуск) используется в случае, когда вы ожидаете, что ваш тест пройдет
только при соблюдении некоторых условий, в противном случае ``pytest``
должен полностью пропустить выполнение теста. Распространенными примерами
являются пропуск тестов только для windows на платформах, отличных от windows,
или пропуск тестов, зависящих от внешнего ресурса, который в данный момент
недоступен (например, базы данных).

**xfail** применяется, когда вы ожидаете, что тест по каким-то причинам должен упасть.
Обычный пример - это тест на еще не реализованную функцию или еще не исправленную ошибку.
Когда тест, помеченный ``pytest.mark.xfail``, проходит, несмотря на ожидаемое падение,
в сводке результатов он будет помечен как **xpass**.

``pytest`` подсчитывает и перечисляет тесты, помеченные ``skip`` и ``xfail``,
отдельно. Подробная информация о пропущенных / упавших тестах по умолчанию не отображается,
чтобы не загромождать выходные данные. Чтобы увидеть детали, соответствующие "коротким" буквам,
показанным в ходе выполнения теста, можно использовать параметр ``-r``, как показано ниже:

.. code-block:: bash

    pytest -rxXs  # show extra info on xfailed, xpassed, and skipped tests

Больше информации о параметре  ``-r`` можно получить, выполнив команду ``pytest -h`` (вызов справки).

(См. :ref:`how to change command line options defaults`)

.. _skipif:
.. _skip:
.. _`condition booleans`:

Пропуск тестовых функций
---------------------------

Простейший способ пропустить тестовую функцию - пометить ее декоратором ``skip``,
которому может быть передана в качестве параметра ``reason`` причина пропуска:

.. code-block:: python

    @pytest.mark.skip(reason="no way of currently testing this")
    def test_the_unknown():
        ...

Также можно пропустить тест непосредственно во время выполнения,
вызвав функцию ``pytest.skip(reason)``:

.. code-block:: python

    def test_function():
        if not valid_config():
            pytest.skip("unsupported configuration")

Такой способ может быть полезен, когда невозможно определить условие пропуска во время импорта.

Можно также пропустить выполнение всего тестового модуля - для этого на уровне модуля
используется метод ``pytest.skip(reason, allow_module_level = True)``:

.. code-block:: python

    import sys
    import pytest

    if not sys.platform.startswith("win"):
        pytest.skip("skipping windows-only tests", allow_module_level=True)


**Reference**: :ref:`pytest.mark.skip ref`

``skipif``
~~~~~~~~~~

Этот декоратор используется, если вы хотите пропускать или не пропускать тесты
в зависимости от выполнения какого-либо условия. Ниже - пример тестовой функции,
которую следует пропустить при запуске интерпретатора Python ниже версии 3.6:

.. code-block:: python

    import sys


    @pytest.mark.skipif(sys.version_info < (3, 7), reason="requires python3.7 or higher")
    def test_function():
        ...

Если во время сбора данных условие выполняется (принимает значение ``True``) - тестовая
функция будет пропущена, а указанная причина при использовании параметра ``-rs`` отобразится в отчете.

Маркер ``skipif`` можно использовать совместно для нескольких модулей.
Рассмотрим следующий тестовый модуль:

.. code-block:: python

    # листинг test_mymodule.py
    import mymodule

    minversion = pytest.mark.skipif(
        mymodule.__versioninfo__ < (1, 1), reason="at least mymodule-1.1 required"
    )


    @minversion
    def test_function():
        ...

Можно импортировать маркер и использовать его в другом тестовом модуле:

.. code-block:: python

    # test_myothermodule.py
    from test_mymodule import minversion


    @minversion
    def test_anotherfunction():
        ...

Для больших наборов тестов обычно рекомендуется иметь один файл,
в котором определяются маркеры, которые затем последовательно применяются во всем наборе тестов.

Кроме того, можно  использовать строки условий :ref:`condition strings <string conditions>` вместо
логических значений, но их нельзя легко переносить между модулями, поэтому они поддерживаются главным
образом из соображений обратной совместимости.

**Справка**: :ref:`pytest.mark.skipif ref`


Пропуск всех тестовых функций класса или модуля
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Маркер ``skipif`` (так же, как и остальные маркеры) можно использовать для класса:

.. code-block:: python

    @pytest.mark.skipif(sys.platform == "win32", reason="does not run on windows")
    class TestPosixCalls:
        def test_function(self):
            "will not be setup or run under 'win32' platform"

Если условие ``True``, этот маркер даст результат пропуска для каждого из тестовых методов этого класса.

Если вы хотите пропустить все тестовые функции модуля, вы можете использовать
:globalvar:`pytestmark` на глобальном уровне:

.. code-block:: python

    # test_module.py
    pytestmark = pytest.mark.skipif(...)

Когда к тестовой функции применяется несколько декораторов ``skipif``,
она будет пропущена, если верно любое из условий пропуска.

.. _`whole class- or module level`: mark.html#scoped-marking


Пропуск файлов и директорий
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Иногда может потребоваться пропустить весь файл или каталог, например, если
тесты основаны на специфических для версии Python функциях или содержат код,
который вы не хотите запускать  с помощью ``pytest``. В этом случае необходимо
исключить файлы и каталоги из коллекции. Дополнительную информацию
смотрите в разделе :ref:`customizing-test-collection`.


Пропуск тестов в зависимости от успешности импорта
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вы можете пропустить тесты в случае неудачного импорта, используя :ref:`pytest.importorskip ref`
на уровне модуля, в рамках теста или функции настройки теста.

.. code-block:: python

    docutils = pytest.importorskip("docutils")

В данном случае, если  ``docutils`` не будет импортирован, то тест будет пропущен.
Также можно пропустить тест в зависимости от версии импортируемой библиотеки:

.. code-block:: python

    docutils = pytest.importorskip("docutils", minversion="0.3")

При этом версия считывается из специального атрибута модуля ``__version__``.

Краткая сводка
~~~~~~~~~~~~~~~~~~~~~

Вот краткое руководство о том, как пропускать тесты в модуле в разных ситуациях:

1. Безоговорочно пропустить все тесты в модуле:

  .. code-block:: python

        pytestmark = pytest.mark.skip("all tests still WIP")

2. Пропустить все тесты в модуле по некоторому условию:

  .. code-block:: python

        pytestmark = pytest.mark.skipif(sys.platform == "win32", reason="tests for linux only")

3. Пропустить все тесты в модуле, если отсутствует какой-либо импорт:

  .. code-block:: python

        pexpect = pytest.importorskip("pexpect")


.. _xfail:

XFail: маркируем тесты, которые должны упасть
----------------------------------------------

Маркер ``xfail`` используется для пометки ожидаемо падающих тестов:

.. code-block:: python

    @pytest.mark.xfail
    def test_function():
        ...

Такой тест будет запущен, но при падении не вызовет сообщения об ошибке.
В отчете он будет помещен в раздел ожидаемых сбоев (``XFAIL``) или неожиданно
прошедших (``XPASS``).

В качестве альтернативы вы также можете пометить тест как ``XFAIL`` из теста или его функции
настройки в обязательном порядке:

.. code-block:: python

    def test_function():
        if not valid_config():
            pytest.xfail("failing configuration (but should work)")

.. code-block:: python

    def test_function2():
        import slow_module

        if slow_module.slow_function():
            pytest.xfail("slow_module taking too long")

Эти два примера иллюстрируют ситуации, когда вы не хотите проверять условие на уровне модуля, когда
в противном случае условие оценивалось бы на предмет меток.

Это приведет ``test_function`` к ``XFAIL``. Обратите внимание, что никакой другой код не выполняется
после вызова :func:`pytest.xfail`, в отличие от маркера. Это потому, что это реализовано внутри, вызывая
известное исключение.

**ссылка**: :ref:`pytest.mark.xfail ref`


параметр ``condition``
~~~~~~~~~~~~~~~~~~~~~~~

Если ожидается, что тест упадет только при определенных условиях, вы можете передать
это условие в качестве первого параметра:

.. code-block:: python

    @pytest.mark.xfail(sys.platform == "win32", reason="bug in a 3rd party library")
    def test_function():
        ...

Обратите внимание, что вы также должны указать причину(см. описание параметра в
:ref:`pytest.mark.xfail ref`).

Параметр ``reason``
~~~~~~~~~~~~~~~~~~~~

Вы можете указать причину ожидаемого сбоя с помощью параметра ``reason``:

.. code-block:: python

    @pytest.mark.xfail(reason="known parser issue")
    def test_function():
        ...


Параметр ``raises``
~~~~~~~~~~~~~~~~~~~~

Если вы хотите уточнить причину сбоя теста, вы можете указать одно исключение или кортеж исключений в
аргументе ``raises``.

.. code-block:: python

    @pytest.mark.xfail(raises=RuntimeError)
    def test_function():
        ...

В этом случае тест будет объявлен в отчете, как обычный сбой,
если он не выполняется с исключением, упомянутом в параметре ``raises``.

Параметр ``run``
~~~~~~~~~~~~~~~~~

Если тест должен быть помечен и учитываться в отчете как маркированный ``xfail``,
но при этом даже не должен выполняться, можно установить параметр ``run`` в значение ``False``:

.. code-block:: python

    @pytest.mark.xfail(run=False)
    def test_function():
        ...

Это особенно полезно для тестов ``xfail``, которые приводят к сбою интерпретатора
и должны быть исследованы позже.

.. _`xfail strict tutorial`:

Параметр ``strict``
~~~~~~~~~~~~~~~~~~~~

Как ``XFAIL``, так и ``XPASS`` не проваливает набор тестов по умолчанию.
Вы можете изменить это, установив параметр ``strict`` только для ключевых слов в ``True``:

.. code-block:: python

    @pytest.mark.xfail(strict=True)
    def test_function():
        ...


Это приведет к тому, что результаты этого теста ``XPASS`` («неожиданно успешно пройдены») не пройдут
набор тестов.

Вы можете изменить значение по умолчанию для параметра ``strict`` используя ini опцию
``xfail_strict``:

.. code-block:: ini

    [pytest]
    xfail_strict=true


Игнорирование ``xfail``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Указав в командной строке:

.. code-block:: bash

    pytest --runxfail

вы можете принудительно запустить тест с пометкой ``xfail`` и сообщить о нем, как если бы он вообще не был
помечен. Это также приводит к тому, что :func:`pytest.xfail` не производит никакого эффекта.

Примеры
~~~~~~~~

Простой тест с несколькими примерами:

.. literalinclude:: /example/xfail_demo.py

Запустив его с параметром ``-rx`` (report-on-xfail), получим следующий отчет:

.. code-block:: pytest

    example $ pytest -rx xfail_demo.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR/example
    collected 7 items

    xfail_demo.py xxxxxxx                                                [100%]

    ========================= short test summary info ==========================
    XFAIL xfail_demo.py::test_hello
    XFAIL xfail_demo.py::test_hello2
      reason: [NOTRUN]
    XFAIL xfail_demo.py::test_hello3
      condition: hasattr(os, 'sep')
    XFAIL xfail_demo.py::test_hello4
      bug 110
    XFAIL xfail_demo.py::test_hello5
      condition: pytest.__version__[0] != "17"
    XFAIL xfail_demo.py::test_hello6
      reason: reason
    XFAIL xfail_demo.py::test_hello7
    ============================ 7 xfailed in 0.12s ============================

.. _`skip/xfail with parametrize`:

``Skip``/``xfail`` с параметризацией
---------------------------------------------

При использовании параметризации можно применять маркеры, такие как ``skip/xfail``, к отдельным
тестовым экземплярам:

.. code-block:: python

    import sys
    import pytest


    @pytest.mark.parametrize(
        ("n", "expected"),
        [
            (1, 2),
            pytest.param(1, 0, marks=pytest.mark.xfail),
            pytest.param(1, 3, marks=pytest.mark.xfail(reason="some bug")),
            (2, 3),
            (3, 4),
            (4, 5),
            pytest.param(
                10, 11, marks=pytest.mark.skipif(sys.version_info >= (3, 0), reason="py2k")
            ),
        ],
    )
    def test_increment(n, expected):
        assert n + 1 == expected
