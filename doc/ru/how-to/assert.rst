.. _`assert`:

Как писать "проверки"(assertions) в тестах и выводить их
================================================================

.. _`assert with the assert statement`:

Проверка с помощью оператора ``assert``
---------------------------------------------------------

``pytest`` позволяет использовать стандартный  оператор языка Python - ``assert`` -
для проверки соответствия ожидаемых результатов фактическим. Или, например, можно
написать следующее:

.. code-block:: python

    # листинг test_assert1.py
    def f():
        return 3


    def test_function():
        assert f() == 4

чтобы убедиться, что ваша функция возвращает определенное значение. Если это проверка
не выполняется, вы увидите возвращаемое значение вызванной функции:

.. code-block:: pytest

    $ pytest test_assert1.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_assert1.py F                                                    [100%]

    ================================= FAILURES =================================
    ______________________________ test_function _______________________________

        def test_function():
    >       assert f() == 4
    E       assert 3 == 4
    E        +  where 3 = f()

    test_assert1.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_assert1.py::test_function - assert 3 == 4
    ============================ 1 failed in 0.12s =============================

``pytest`` поддерживает отображение наиболее распространенных подвыражений,
включая вызовы, атрибуты, сравнения, а также бинарные и унарные операторы.
(См. :ref:`tbreportdemo`). Что позволяет использовать характерные для этого конструкции
``python`` без шаблонного кода, не теряя при этом информацию.

Однако, если вы укажете сообщение с такой проверкой:

.. code-block:: python

    assert a % 2 == 0, "значение нечетное, должно быть четным"

то никакая аналитическая информация выводиться не будет, и это сообщение будет просто показано
в трассировке.

См. :ref:`assert-details` для получения дополнительной информации.

.. _`assertraises`:

Проверка ожидаемых исключений
------------------------------------------

Чтобы убедиться в том, что вызвано ожидаемое исключение, используется
:func:`pytest.raises` в следующем контексте:

.. code-block:: python

    import pytest


    def test_zero_division():
        with pytest.raises(ZeroDivisionError):
            1 / 0

и если нужен доступ к фактической информации об исключении, используется:

.. code-block:: python

    def test_recursion_depth():
        with pytest.raises(RuntimeError) as excinfo:

            def f():
                f()

            f()
        assert "maximum recursion" in str(excinfo.value)

``excinfo`` в примере :class:`~pytest.ExceptionInfo`, который является оберткой вокруг
возникшего фактического исключения.  Основными атрибутами являются: ``.type``, ``.value``
и ``.traceback``.

Можно передать ключевой параметр ``match`` в диспетчер контекста, чтобы проверить
соответствие регулярного выражения строковому представлению исключения (аналогично методу
``TestCase.assertRaisesRegexp`` из ``unittest``):

.. code-block:: python

    import pytest


    def myfunc():
        raise ValueError("Exception 123 raised")


    def test_match():
        with pytest.raises(ValueError, match=r".* 123 .*"):
            myfunc()

Регулярное выражение параметра ``match``  совпадает с функцией ``re.search``,
так что в приведенном выше примере ``match='123'`` также сработает.

Есть и альтернативный вариант использования ``pytest.raises``, когда вы передаете функцию, которая
которая должна выполняться с заданными ``*args`` и ``**kwargs``
и проверять, что вызвано указанное исключение:

.. code-block:: python

    pytest.raises(ExpectedException, func, *args, **kwargs)

``pytest`` выведет полезные результаты в случае сбоев, таких как *отсутствие
исключения*(no exception) или *неверное исключение*(wrong exception).

Обратите внимание, что также можно указать параметр "raises" для ``pytest.mark.xfail``, который
проверяет падение теста более точным способом, чем просто возникновение какого-либо
исключения:

.. code-block:: python

    @pytest.mark.xfail(raises=IndexError)
    def test_f():
        f()

Использование :func:`pytest.raises` скорее всего пригодится, когда вы тестируете исключения, генерируемые
собственным кодом, а вот маркировка тестовой функции маркером ``@pytest.mark.xfail``, наверное,
лучше подойдет для документирования незафиксированных (когда тест описывает то, что "должно бы"
происходить) или зависимых от чего-либо багов.

.. _`assertwarns`:

Проверка ожидаемых предупреждений
-----------------------------------------

Вы можете проверить, вызывает ли код конкретное предупреждение, используя
:ref:`pytest.warns <warns>`.


.. _newreport:

Использование контекстно-зависимых сравнений
-------------------------------------------------

В ``pytest`` есть широкая поддержка для предоставления контекстно-зависимой информации
при сравнении. Например:

.. code-block:: python

    # cлистинг test_assert2.py
    def test_set_comparison():
        set1 = set("1308")
        set2 = set("8035")
        assert set1 == set2

если вы запустите этот модуль:

.. code-block:: pytest

    $ pytest test_assert2.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_assert2.py F                                                    [100%]

    ================================= FAILURES =================================
    ___________________________ test_set_comparison ____________________________

        def test_set_comparison():
            set1 = set("1308")
            set2 = set("8035")
    >       assert set1 == set2
    E       AssertionError: assert {'0', '1', '3', '8'} == {'0', '3', '5', '8'}
    E         Extra items in the left set:
    E         '1'
    E         Extra items in the right set:
    E         '5'
    E         Use -v to get the full diff

    test_assert2.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_assert2.py::test_set_comparison - AssertionError: assert {'0'...
    ============================ 1 failed in 0.12s =============================

Специальные сравнения сделаны для отдельных случаев:

* сравнение длинных строк: показано сравнение контекста;
* сравнение длинных последовательностей: показан индекс первого несоответствия;
* сравнение словарей: показаны различающиеся элементы.


См. :ref:`reporting demo <tbreportdemo>` для других примеров.

Определение собственных сообщений к упавшим ``assert``
--------------------------------------------------------

Можно добавить свои собственные подробные пояснения, реализовав хук
``pytest_assertrepr_compare``.

.. autofunction:: _pytest.hookspec.pytest_assertrepr_compare
   :noindex:

Для примера рассмотрим добавление в файл :ref:`conftest.py <conftest.py>` хука, который
устанавливает наше сообщение для сравниваемых объектов ``Foo``:

.. code-block:: python

   # листинг conftest.py
   from test_foocompare import Foo


   def pytest_assertrepr_compare(op, left, right):
       if isinstance(left, Foo) and isinstance(right, Foo) and op == "==":
           return [
               "Comparing Foo instances:",
               "   vals: {} != {}".format(left.val, right.val),
           ]

теперь, учитывая этот тестовый модуль:

.. code-block:: python

   # листинг test_foocompare.py
   class Foo:
       def __init__(self, val):
           self.val = val

       def __eq__(self, other):
           return self.val == other.val


   def test_compare():
       f1 = Foo(1)
       f2 = Foo(2)
       assert f1 == f2

вы можете запустить тестовый модуль и получить пользовательский вывод, определенный в
файле ``conftest.py``:

.. code-block:: pytest

   $ pytest -q test_foocompare.py
   F                                                                    [100%]
   ================================= FAILURES =================================
   _______________________________ test_compare _______________________________

       def test_compare():
           f1 = Foo(1)
           f2 = Foo(2)
   >       assert f1 == f2
   E       assert Comparing Foo instances:
   E            vals: 1 != 2

   test_foocompare.py:12: AssertionError
   ========================= short test summary info ==========================
   FAILED test_foocompare.py::test_compare - assert Comparing Foo instances:
   1 failed in 0.12s

.. _assert-details:
.. _`assert introspection`:

Детальный анализ неудачных проверок (assertion introspection)
------------------------------------------------------------------

Детальный анализ упавших проверок достигается переопределением операторов ``assert`` перед запуском.
Переопределенные ``assert`` помещают аналитическую информацию в сообщение о неудачной проверке.
``pytest`` переопределяет только тестовые модули, обнаруженные им в процессе сборки (collecting)
тестов, поэтому **``assert``-ы в поддерживающих модулях, которые сами по себе не являются тестами,
переопределены не будут**.

Можно вручную включить возможность переопределения ``assert`` для импортируемого модуля, вызвав
`register_assert_rewrite <https://docs.pytest.org/en/stable/writing_plugins.html#assertion-rewriting>`_ перед его импортом
(лучше это сделать в корневом файле``conftest.py``).

Дополнительную информацию можно найти в статье Бенджамина Петерсона:
`Behind the scenes of pytest's new assertion rewriting <http://pybites.blogspot.com/2011/07/behind-scenes-of-pytests-new-assertion.html>`_.

Кэширование переопределенных файлов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``pytest`` кэширует переопределенные модули на диск. Можно отключить такое поведение (например,
чтобы избежать устаревших ``.pyc`` файлов в проектах, которые задействуют множество файлов),
добавив в ваш корневой файл ``conftest.py``:

.. code-block:: python

   import sys

   sys.dont_write_bytecode = True

Обратите внимание, что это не влияет на анализ упавших проверок, единственное отличие заключается в том,
что ``.pyc``-файлы не будут кэшироваться на диск.

Кроме того, кэширование при переопределении будет автоматически отключаться, если не получается записать
новые ``.pyc``- файлы, т. е. для read-only файлов или zip-архивов.


Отключение переопределения ``assert``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``pytest`` перезаписывает тестовые модули при импорте, используя хук импорта для записи новых
файлов ``pyc``. В большинстве случаев это работает прозрачно. Тем не менее, при работе с механизмом импорта,
такой способ может создавать проблемы.

В этом случае у вас есть два варианта:

* Отключить переопределение для отдельного модуля, добавив строку ``PYTEST_DONT_REWRITE`` в
  docstring (строковую переменную для документирования модуля).

* Отключить переопределение для всех модулей с помощью ``--assert=plain``.
