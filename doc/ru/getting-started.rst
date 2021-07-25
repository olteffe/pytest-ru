.. _get-started:

Первые шаги
===================================

.. _`getstarted`:
.. _`installation`:

Установка ``pytest``
----------------------------------------

Для ``pytest`` требуется: Python 3.6, 3.7, 3.8, 3.9, или PyPy3.

1. Выполните следующую команду в командной строке:

.. code-block:: bash

    pip install -U pytest

2. Убедитесь, что вы установили правильную версию:

.. code-block:: bash

    $ pytest --version
    pytest 6.2.3

.. _`simpletest`:

Создание первого теста
----------------------------------------------------------

Создайте новый файл с именем ``test_sample.py``, содержащий функцию и тест:

.. code-block:: python

    # содержимое test_sample.py
    def func(x):
        return x + 1


    def test_answer():
        assert func(3) == 5

Тестирование:

.. code-block:: pytest

    $ pytest
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_sample.py F                                                     [100%]

    ================================= FAILURES =================================
    _______________________________ test_answer ________________________________

        def test_answer():
    >       assert func(3) == 5
    E       assert 4 == 5
    E        +  where 4 = func(3)

    test_sample.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_sample.py::test_answer - assert 4 == 5
    ============================ 1 failed in 0.12s =============================

Значение ``[100%]`` относится к общему прогрессу выполнения всех тестовых кейсов. После завершения pytest
выдает сообщение о падении, поскольку ``func(3)`` не возвращает ``5``.

.. note::

    Вы можете использовать оператор ``assert`` для проверки ожиданий теста.
    `Продвинутая интроспекция утверждений pytest <http://docs.python.org/reference/simple_stmts.html#the-assert-statement>`_
    будет интеллектуально сообщать промежуточные значения выражения assert, чтобы вы могли избежать множества
    имен `унаследованных методов JUnit <http://docs.python.org/library/unittest.html#test-cases>`_.

Запуск нескольких тестов
----------------------------------------------------------

``pytest`` запустит все файлы вида test_*.py или \*_test.py в текущем каталоге и его
подкаталогах. В более общем случае, он следует :ref:`стандартным правилам обнаружения тестов <test discovery>`.


Утверждать, что определенное исключение вызвано
--------------------------------------------------------------

Используйте помощник :ref:`raises <assertraises>` для утверждения, что некоторый код вызывает исключение:

.. code-block:: python

    # содержимое test_sysexit.py
    import pytest


    def f():
        raise SystemExit(1)


    def test_mytest():
        with pytest.raises(SystemExit):
            f()

Запуск тестовой функции в «тихом» режиме:

.. code-block:: pytest

    $ pytest -q test_sysexit.py
    .                                                                    [100%]
    1 passed in 0.12s

.. note::

    Флаг ``-q/--quiet`` сохраняет краткость вывода в этом и следующих примерах.

Группировка нескольких тестов в классе
--------------------------------------------------------------

.. regendoc:wipe

После разработки нескольких тестов вы можете захотеть сгруппировать их в класс. pytest позволяет легко
создать класс, содержащий более одного теста:

.. code-block:: python

    # листинг test_class.py
    class TestClass:
        def test_one(self):
            x = "this"
            assert "h" in x

        def test_two(self):
            x = "hello"
            assert hasattr(x, "check")

``pytest`` обнаруживает все тесты, следуя своему :ref:`Соглашению для обнаружения тестов Python <test discovery>`,
поэтому он находит обе функции с префиксом ``test_``. Нет необходимости создавать подклассы, но убедитесь,
что ваш класс имеет префикс ``Test``, иначе класс будет пропущен. Мы можем просто запустить модуль,
передав ему имя файла:

.. code-block:: pytest

    $ pytest -q test_class.py
    .F                                                                   [100%]
    ================================= FAILURES =================================
    ____________________________ TestClass.test_two ____________________________

    self = <test_class.TestClass object at 0xdeadbeef>

        def test_two(self):
            x = "hello"
    >       assert hasattr(x, "check")
    E       AssertionError: assert False
    E        +  where False = hasattr('hello', 'check')

    test_class.py:8: AssertionError
    ========================= short test summary info ==========================
    FAILED test_class.py::TestClass::test_two - AssertionError: assert False
    1 failed, 1 passed in 0.12s

Первый тест прошел, а второй - нет. Вы можете легко увидеть промежуточные значения в утверждении,
которые помогут вам понять причину падения.

Группирование тестов в класс может быть полезно по следующим причинам:

 * ОСтруктурирование тестирования.
 * Совместное использование фикстур для тестов только в данном конкретном классе.
 * Применение меток на уровне класса и их неявное применение ко всем тестам.

При группировке тестов внутри классов следует помнить о том, что каждый тест имеет уникальный экземпляр класса.
Если каждый тест будет использовать один и тот же экземпляр класса, это будет очень вредно для изоляции тестов
и будет способствовать плохой практике тестирования.
Это описано ниже:

.. regendoc:wipe

.. code-block:: python

    # листинг test_class_demo.py
    class TestClassDemoInstance:
        def test_one(self):
            assert 0

        def test_two(self):
            assert 0


.. code-block:: pytest

    $ pytest -k TestClassDemoInstance -q
    FF                                                                   [100%]
    ================================= FAILURES =================================
    ______________________ TestClassDemoInstance.test_one ______________________

    self = <test_class_demo.TestClassDemoInstance object at 0xdeadbeef>

        def test_one(self):
    >       assert 0
    E       assert 0

    test_class_demo.py:3: AssertionError
    ______________________ TestClassDemoInstance.test_two ______________________

    self = <test_class_demo.TestClassDemoInstance object at 0xdeadbeef>

        def test_two(self):
    >       assert 0
    E       assert 0

    test_class_demo.py:6: AssertionError
    ========================= short test summary info ==========================
    FAILED test_class_demo.py::TestClassDemoInstance::test_one - assert 0
    FAILED test_class_demo.py::TestClassDemoInstance::test_two - assert 0
    2 failed in 0.12s

Обратите внимание, что атрибуты, добавленные на уровне класса, являются атрибутами класса, поэтому они
будут совместно использоваться тестами.

Запрос уникального временного каталога для функциональных тестов
-----------------------------------------------------------------

``pytest`` предоставляет ``аргументы встроенной функции/фикстуры <https://docs.pytest.org/en/stable/builtin.html>`_.
для запроса произвольных ресурсов, например, уникального временного каталога:

.. code-block:: python

    # листинг test_tmp_path.py
    def test_needsfiles(tmp_path):
        print(tmp_path)
        assert 0

Укажите имя ``tmp_path`` в сигнатуре тестовой функции, и ``pytest`` будет искать и вызывать фабрику фикстур
для создания ресурса перед выполнением вызова тестовой функции. Перед запуском теста ``pytest``
создает уникальный для каждого теста временный каталог:

.. code-block:: pytest

    $ pytest -q test_tmp_path.py
    F                                                                    [100%]
    ================================= FAILURES =================================
    _____________________________ test_needsfiles ______________________________

    tmp_path = Path('PYTEST_TMPDIR/test_needsfiles0')

        def test_needsfiles(tmp_path):
            print(tmp_path)
    >       assert 0
    E       assert 0

    test_tmpdir.py:3: AssertionError
    --------------------------- Captured stdout call ---------------------------
    PYTEST_TMPDIR/test_needsfiles0
    ========================= short test summary info ==========================
    FAILED test_tmp_path.py::test_needsfiles - assert 0
    1 failed in 0.12s

Более подробная информация об обработке временного каталога доступна по ссылке
:ref:`Временные каталоги и файлы <tmpdir handling>`.

Выяснить, какие встроенные :ref:`фикстуры в pytest <fixtures>` существуют можно по команде:

.. code-block:: bash

    pytest --fixtures   # shows builtin and custom fixtures

Обратите внимание, что эта команда опускает фикстуры с впередистоящим ``_`` , если не добавлена опция ``-v``.

Дополнительная информация
-------------------------------------

Ознакомьтесь с дополнительными ресурсами pytest, которые помогут вам настроить тесты для вашего уникального рабочего процесса:

* ":ref:`usage`" для примеров вызова командной строки
* ":ref:`existingtestsuite`" для работы с уже существующими тестами
* ":ref:`mark`" для получения информации о механизме ``pytest.mark``
* ":ref:`fixtures`" для обеспечения функциональной основы для ваших тестов
* ":ref:`plugins`" для управления и написания плагинов
* ":ref:`goodpractices`" для virtualenv и тестовых макетов
