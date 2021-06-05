
.. _`unittest.TestCase`:
.. _`unittest`:

Как использовать тесты на основе ``unittest`` с помощью pytest
================================================================

``pytest`` из коробки поддерживает запуск тестов на основе ``unittest``. Он предназначен для использования
существующих наборов тестов на основе ``unittest`` для использования pytest в качестве средства запуска
тестов, а также позволяет постепенно адаптировать набор тестов, чтобы в полной мере использовать
возможности pytest.

Для запуска с помощью ``pytest`` наборов тестов в стиле ``unittest`` , напечатайте:

.. code-block:: bash

    pytest tests


``pytest`` автоматически соберет  подклассы ``unittest.TestCase``и их тестовые методы ``test``
в файлах ``test_*.py`` или ``*_test.py``.

Поддерживаются почти все особенности ``unittest``:

* ``@unittest.skip`` декораторы стиля;
* ``setUp/tearDown``;
* ``setUpClass/tearDownClass``;
* ``setUpModule/tearDownModule``;

.. _`load_tests protocol`: https://docs.python.org/3/library/unittest.html#load-tests-protocol
.. _`subtests`: https://docs.python.org/3/library/unittest.html#distinguishing-test-iterations-using-subtests

На текущий момент pytest не поддерживает следующие функции:

* `load_tests protocol`_;
* `subtests`_;

Преимущества из коробки
--------------------------

Запустив свой набор тестов с помощью pytest, вы можете использовать несколько функций, в большинстве
случаев без изменения существующего кода:

* Получение:ref:`more informative tracebacks <tbreportdemo>`;
* захват :ref:`stdout and stderr <captures>` ;
* :ref:`Test selection options <select-tests>` используя флаги ``-k`` и ``-m``;
* :ref:`maxfail`;
* :ref:`--pdb <pdb-option>` опция командной строки для отладки при сбоях теста
  (см. :ref:`note <pdb-unittest-note>` ниже);
* Распределение тестов на несколько процессоров, используя плагин `pytest-xdist <https://pypi.org/project/pytest-xdist/>`_ ;
* Использование :ref:`plain assert-statements <assert>` взамен функции ``self.assert*``(`unittest2pytest
  <https://pypi.org/project/unittest2pytest/>`__ очень помогает в этом);


Особенности pytest в подклассах ``unittest.TestCase``
------------------------------------------------------

Следующие функции pytest работают в подклассах ``unittest.TestCase``:

* :ref:`Marks <mark>`: :ref:`skip <skip>`, :ref:`skipif <skipif>`, :ref:`xfail <xfail>`;
* :ref:`Auto-use fixtures <mixing-fixtures>`;

Следующие функции pytest **НЕ** работают и, вероятно, никогда не будут работать из-за другой философии дизайна:

* :ref:`Fixtures <fixture>` (за исключением фикстур ``autouse``, см. :ref:`below <mixing-fixtures>`);
* :ref:`Parametrization <parametrize>`;
* :ref:`Custom hooks <writing-plugins>`;


Сторонние плагины могут работать, а могут и не работать, в зависимости от плагина и набора тестов.

.. _mixing-fixtures:

Смешивание фикстур pytest с подклассами ``unittest.TestCase`` с использованием меток
--------------------------------------------------------------------------------------

Запуск unittest с помощью ``pytest`` позволяет использовать механику фикстур
:ref:`fixture mechanism <fixture>` с тестами стиля ``unittest.TestCase``. Предполагая, что вы хотя бы
ознакомились с функциями фикстур pytest, давайте перейдем к примеру, который интегрирует фикстуру
pytest ``db_class``, настраивает объект базы данных в кэше классов, а затем ссылается на него из
теста в стиле unittest:

.. code-block:: python

    # листинг conftest.py

    # мы определяем функцию фикстуры ниже, и она будет "использоваться"
    # ссылаясь на свое название из тестов

    import pytest


    @pytest.fixture(scope="class")
    def db_class(request):
        class DummyDB:
            pass

        # установка атрибута класса в вызывающем тестовом контексте
        request.cls.db = DummyDB()

Это определяет функцию фикстуры ``db_class``, которая, если она используется, вызывается один раз для
каждого тестового класса и устанавливает атрибут ``db`` уровня класса в экземпляр ``DummyDB``. Функция фикстуры
достигает этого, получая специальный объект ``request``, который дает доступ к :ref:`the requesting test context <request-context>`,
например, атрибуту ``cls``, обозначающему класс, из которого фикстура используется. Эта архитектура
отделяет написание фикстуры от реального тестового кода и позволяет повторно использовать фикстуру по
минимальной ссылке - имени фикстуры. Итак, давайте напишем реальный класс ``unittest.TestCase``, используя
наше определение фикстуры:

.. code-block:: python

    # листинг test_unittest_db.py

    import unittest
    import pytest


    @pytest.mark.usefixtures("db_class")
    class MyTest(unittest.TestCase):
        def test_method1(self):
            assert hasattr(self, "db")
            assert 0, self.db  # падение теста для демонстрационных целей

        def test_method2(self):
            assert 0, self.db  # падение теста для демонстрационных целей

Класс-декоратор ``@pytest.mark.usefixtures("db_class")`` гарантирует, что функция фикстуры pytest
``db_class`` вызывается один раз для каждого класса.
Из-за преднамеренного падения assert мы можем взглянуть на значения ``self.db`` в трассировке:

.. code-block:: pytest

    $ pytest test_unittest_db.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 2 items

    test_unittest_db.py FF                                               [100%]

    ================================= FAILURES =================================
    ___________________________ MyTest.test_method1 ____________________________

    self = <test_unittest_db.MyTest testMethod=test_method1>

        def test_method1(self):
            assert hasattr(self, "db")
    >       assert 0, self.db  # fail for demo purposes
    E       AssertionError: <conftest.db_class.<locals>.DummyDB object at 0xdeadbeef>
    E       assert 0

    test_unittest_db.py:10: AssertionError
    ___________________________ MyTest.test_method2 ____________________________

    self = <test_unittest_db.MyTest testMethod=test_method2>

        def test_method2(self):
    >       assert 0, self.db  # fail for demo purposes
    E       AssertionError: <conftest.db_class.<locals>.DummyDB object at 0xdeadbeef>
    E       assert 0

    test_unittest_db.py:13: AssertionError
    ========================= short test summary info ==========================
    FAILED test_unittest_db.py::MyTest::test_method1 - AssertionError: <conft...
    FAILED test_unittest_db.py::MyTest::test_method2 - AssertionError: <conft...
    ============================ 2 failed in 0.12s =============================

Эта трассировка pytest по умолчанию показывает, что два тестовых метода используют один и тот же
экземпляр ``self.db``, что было нашим намерением при написании функции фикстуры с привязкой к классу выше.


Использование фикстур autouse и доступ к другим фикстурам
--------------------------------------------------------------

Хотя обычно лучше явно декларировать использование фикстур, необходимых для данного теста, иногда вам
могут понадобиться фикстуры, которые автоматически используются в данном контексте. В конце
концов, традиционный стиль установки unittest требует использования этого неявного написания фикстуры,
и, скорее всего, вы к этому привыкли или вам это нравится.

Вы можете пометить функции фикстуры с помощью ``@pytest.fixture(autouse=True)``
и определить функцию фикстуры в контексте, в котором вы хотите ее использовать.
Давайте посмотрим на фикстуру ``initdir``, которая заставляет все тестовые методы класса ``TestCase``
выполняться во временном каталоге с предварительно инициализированным файлом ``samplefile.ini``.
Наша фикстура ``initdir`` самостоятельно использует встроенную в pytest фикстуру :fixture:`tmp_path`
для делегирования создания временного каталога для каждого теста:

.. code-block:: python

    # листинг test_unittest_cleandir.py
    import os
    import pytest
    import unittest


    class MyTest(unittest.TestCase):
        @pytest.fixture(autouse=True)
        def initdir(self, tmp_path, monkeypatch):
            monkeypatch.chdir(tmp_path)  # изменение временной директории, предоставленной pytest
            tmp_path.joinpath("samplefile.ini").write_text("# testdata")

        def test_method(self):
            with open("samplefile.ini") as f:
                s = f.read()
            assert "testdata" in s

Из-за флага ``autouse`` функция фикстуры ``initdir`` будет использоваться для всех методов класса, в
котором она определена.  Это ярлык для использования маркера ``@pytest.mark.usefixtures("initdir")``
в классе, как в предыдущем примере.

Запуск этого тестового модуля...:

.. code-block:: pytest

    $ pytest -q test_unittest_cleandir.py
    .                                                                    [100%]
    1 passed in 0.12s

... дает нам один пройденный тест, потому что функция фикстуры ``initdir`` была выполнена перед
``test_method``.

.. note::

   методы ``unittest.TestCase`` не могут напрямую получать аргументы фикстуры как реализацию, которая
   может повлиять на возможность запуска общего набора тестов unittest.TestCase.

   Вышеприведенные примеры ``usefixtures`` и ``autouse`` должны помочь соединении фикстур pytest с
   наборами unittest.

   Вы также можете постепенно перейти от создания подклассов ``unittest.TestCase`` к *простым assert*,
   а затем постепенно извлекать пользу из использования полного набора функций pytest.

.. _pdb-unittest-note:

.. note::

   Из-за архитектурных различий между двумя фреймворками, этапы ``setup`` и ``teardown`` для тестов на
   основе ``unittest`` выполняется во время фазы ``вызова`` тестирования, а не в стандартной ``setup`` и
   ``teardown`` ``pytest``. Это может быть важно понимать в некоторых ситуациях, особенно при
   рассмотрении ошибок. Например, если пакет на основе ``unittest`` обнаруживает ошибки во время
   ``setup``, ``pytest`` не будет сообщать об ошибках во время фазы ``setup`` и вместо этого вызовет
   ошибку во время ``вызова``.
