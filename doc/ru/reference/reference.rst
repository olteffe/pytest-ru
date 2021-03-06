.. _`api-reference`:

Справочник по API
===================

Эта страница содержит полную справку на API pytest.

.. contents::
    :depth: 3
    :local:

Функции
---------

pytest.approx
~~~~~~~~~~~~~

.. autofunction:: pytest.approx

pytest.fail
~~~~~~~~~~~

**Руководство**: :ref:`skipping`

.. autofunction:: pytest.fail

pytest.skip
~~~~~~~~~~~

.. autofunction:: pytest.skip(msg, [allow_module_level=False])

.. _`pytest.importorskip ref`:

pytest.importorskip
~~~~~~~~~~~~~~~~~~~

.. autofunction:: pytest.importorskip

pytest.xfail
~~~~~~~~~~~~

.. autofunction:: pytest.xfail

pytest.exit
~~~~~~~~~~~

.. autofunction:: pytest.exit

pytest.main
~~~~~~~~~~~

.. autofunction:: pytest.main

pytest.param
~~~~~~~~~~~~

.. autofunction:: pytest.param(*values, [id], [marks])

pytest.raises
~~~~~~~~~~~~~

**Руководство**: :ref:`assertraises`.

.. autofunction:: pytest.raises(expected_exception: Exception [, *, match])
    :with: excinfo

pytest.deprecated_call
~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`ensuring_function_triggers`.

.. autofunction:: pytest.deprecated_call()
    :with:

pytest.register_assert_rewrite
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`assertion-rewriting`.

.. autofunction:: pytest.register_assert_rewrite

pytest.warns
~~~~~~~~~~~~

**Руководство**: :ref:`assertwarnings`

.. autofunction:: pytest.warns(expected_warning: Exception, [match])
    :with:

pytest.freeze_includes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`freezing-pytest`.

.. autofunction:: pytest.freeze_includes

.. _`marks ref`:

Маркировки
------------

Маркировки можно использовать для метаданных *тестовых функций* (но не фикстур), к которым затем могут обращаться
фикстуры или плагины.




.. _`pytest.mark.filterwarnings ref`:

pytest.mark.filterwarnings
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`filterwarnings`.

Добавление предупреждающих фильтров к маркированным элементам теста.

.. py:function:: pytest.mark.filterwarnings(filter)

    :keyword str filter:
        *Строка спецификации предупреждения*, которая состоит из содержимого кортежа ``(action, message, category, module, lineno)``
        как указано в разделе ``Фильтр предупреждений'' <https://docs.python.org/3/library/warnings.html#warning-filter>`_
        документации Python, разделенных знаком ``":"``. Необязательные поля могут быть опущены.
        Имена модулей, переданные для фильтрации, не экранируются с помощью регулярных выражений.

        Например:

        .. code-block:: python

            @pytest.mark.filterwarnings("ignore:.*usage will be deprecated.*:DeprecationWarning")
            def test_foo():
                ...


.. _`pytest.mark.parametrize ref`:

pytest.mark.parametrize
~~~~~~~~~~~~~~~~~~~~~~~

:ref:`parametrize`.

Эта маркировка имеет ту же сигнатуру, что и :py:meth:`pytest.Metafunc.parametrize`; см. здесь.


.. _`pytest.mark.skip ref`:

pytest.mark.skip
~~~~~~~~~~~~~~~~

:ref:`skip`.

Безоговорочно пропустить тестовую функцию.

.. py:function:: pytest.mark.skip(reason=None)

    :keyword str reason: Причина, по которой функция тестирования пропускается.


.. _`pytest.mark.skipif ref`:

pytest.mark.skipif
~~~~~~~~~~~~~~~~~~

:ref:`skipif`.

Пропустить тестовую функцию, если условие ``True``.

.. py:function:: pytest.mark.skipif(condition, *, reason=None)

    :type condition: bool или str
    :param condition: ``True/False`` если условие следует пропустить или :ref:`строка условия <string conditions>`.
    :keyword str reason: Причина, по которой функция тестирования пропускается.


.. _`pytest.mark.usefixtures ref`:

pytest.mark.usefixtures
~~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`usefixtures`.

Маркировка тестовой функции как использующую указанные имена фикстур.

.. py:function:: pytest.mark.usefixtures(*names)

    :param args: Имена используемых фикстур в виде строк.

.. note::

    Когда используете `usefixtures` в хуках, они могут загружать фикстуры только тогда, когда применяется к функции тестирования перед настройкой теста
    (например в хуке `pytest_collection_modifyitems`.

    Также обратите внимание, что эта маркировка не действует при применении к фикстурам.



.. _`pytest.mark.xfail ref`:

pytest.mark.xfail
~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`xfail`.

Маркировка тестовой функции как *ожидаемое падение*.

.. py:function:: pytest.mark.xfail(condition=None, *, reason=None, raises=None, run=True, strict=False)

    :type condition: bool или str
    :param condition:
        Условие маркировки тестовой функции как xfail (``True/False`` или
        :ref:`строка условия <string conditions>`). Если bool, вы также должны
        указать ``причину`` (см. :ref:`строка условия <string conditions>`).
    :keyword str reason:
        Причина, по которой функция тестирования отмечена как xfail.
    :keyword Type[Exception] raises:
        Подкласс исключения, который должен быть вызван тестовой функцией; другие исключения не пройдут тест.
    :keyword bool run:
        Если тестовая функция действительно должна выполняться. Если ``False``, функция всегда будет xfail и не будет
        выполнена (полезно, если функция вызывает падение).
    :keyword bool strict:
        * Если ``False`` (по умолчанию), функция будет показана в выводе терминала как ``xfailed`` в случае падения
          и как ``xpass``, если она прошла. В обоих случаях это не приведет к сбою тестового набора в целом. Это
          особенно полезно для пометки *flaky* тестов (тестов, которые отказывают случайным образом), чтобы заняться ими позже.
        * Если ``True``, функция будет показана в выводе терминала как ``xfailed``, если она не прошла, но если она
          неожиданно пройдет, то она **провалит** набор тестов. Это особенно полезно для пометки функций
          которые постоянно падают, и должно быть четкое указание, если они неожиданно начинают проходить (например
          в новом выпуске библиотеки исправлена известная ошибка).


Пользовательские маркировки
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Маркировки создаются динамически с помощью объекта-фабрики ``pytest.mark`` и применяются как декораторы.

Например:

.. code-block:: python

    @pytest.mark.timeout(10, "slow", method="thread")
    def test_function():
        ...

Создадим и прикрепим объект :class:`Mark <pytest.Mark>` к собранным
:class:`Item <pytest.Item>`, к которым затем можно получить доступ с помощью фикстур или хуков с помощью
:meth:`Node.iter_markers <_pytest.nodes.Node.iter_markers>`. У объекта ``mark`` будут следующие атрибуты:

.. code-block:: python

    mark.args == (10, "slow")
    mark.kwargs == {"method": "thread"}

Пример использования нескольких пользовательских маркеров:

.. code-block:: python

    @pytest.mark.timeout(10, "slow", method="thread")
    @pytest.mark.slow
    def test_function():
        ...

Когда :meth:`Node.iter_markers <_pytest.nodes.Node.iter_markers>` или :meth:`Node.iter_markers <_pytest.nodes.Node.iter_markers_with_node>`
используется с несколькими маркерами, маркер, ближайший к функции, будет итерироваться сначала.
Приведенный выше пример приведет к ``@pytest.mark.slow``, за которым последует ``@pytest.mark.timeout(...)``.

.. _`fixtures-api`:

Фикстуры
----------

**Руководство**: :ref:`fixture`.

Фикстуры запрашиваются тестовыми функциями или другими фикстурами, объявляя их как имена аргументов.


Пример теста, требующего фикстуру:

.. code-block:: python

    def test_output(capsys):
        print("hello")
        out, err = capsys.readouterr()
        assert out == "hello\n"


Пример фикстуры, требующей другую фикстуру:

.. code-block:: python

    @pytest.fixture
    def db_session(tmp_path):
        fn = tmp_path / "db.file"
        return connect(fn)

Для получения более подробной информации обратитесь к полной :ref:`документации по фикстурам <fixture>`.


.. _`pytest.fixture-api`:

@pytest.fixture
~~~~~~~~~~~~~~~

.. autofunction:: pytest.fixture
    :decorator:


.. fixture:: cache

config.cache
~~~~~~~~~~~~

**Руководство**: :ref:`cache`.

Объект ``config.cache`` позволяет другим плагинам и фикстурам
хранить и извлекать значения во время выполнения тестов. Чтобы получить доступ к нему из фикстуры
запросите ``pytestconfig`` в вашей фикстуре и получите его с помощью ``pytestconfig.cache``.

Под капотом, плагин кэша использует простой
API ``dumps``/ ``loads`` модуля :py:mod:`json`` stdlib.

``config.cache`` это пример :class:`pytest.Cache`:

.. autoclass:: pytest.Cache()
   :members:


.. fixture:: capsys

capsys
~~~~~~

:ref:`captures`.

.. autofunction:: _pytest.capture.capsys()
    :no-auto-options:

    Возвращает экземпляр :class:`CaptureFixture[str] <pytest.CaptureFixture>`.

    Например:

    .. code-block:: python

        def test_output(capsys):
            print("hello")
            captured = capsys.readouterr()
            assert captured.out == "hello\n"

.. autoclass:: pytest.CaptureFixture()
    :members:


.. fixture:: capsysbinary

capsysbinary
~~~~~~~~~~~~

:ref:`captures`.

.. autofunction:: _pytest.capture.capsysbinary()
    :no-auto-options:

    Возвращает экземпляр :class:`CaptureFixture[bytes] <pytest.CaptureFixture>`.

    Например:

    .. code-block:: python

        def test_output(capsysbinary):
            print("hello")
            captured = capsysbinary.readouterr()
            assert captured.out == b"hello\n"


.. fixture:: capfd

capfd
~~~~~~

:ref:`captures`.

.. autofunction:: _pytest.capture.capfd()
    :no-auto-options:

    Возвращает экземпляр :class:`CaptureFixture[str] <pytest.CaptureFixture>`.

    Например:

    .. code-block:: python

        def test_system_echo(capfd):
            os.system('echo "hello"')
            captured = capfd.readouterr()
            assert captured.out == "hello\n"


.. fixture:: capfdbinary

capfdbinary
~~~~~~~~~~~~

:ref:`captures`.

.. autofunction:: _pytest.capture.capfdbinary()
    :no-auto-options:

    Возвращает экземпляр :class:`CaptureFixture[bytes] <pytest.CaptureFixture>`.

    Например:

    .. code-block:: python

        def test_system_echo(capfdbinary):
            os.system('echo "hello"')
            captured = capfdbinary.readouterr()
            assert captured.out == b"hello\n"


.. fixture:: doctest_namespace

doctest_namespace
~~~~~~~~~~~~~~~~~

:ref:`doctest`.

.. autofunction:: _pytest.doctest.doctest_namespace()

    Обычно эта фикстура используется вместе с другими ``autouse`` фикстурами:

    .. code-block:: python

        @pytest.fixture(autouse=True)
        def add_np(doctest_namespace):
            doctest_namespace["np"] = numpy

    Больше информации см.: :ref:`doctest_namespace`.


.. fixture:: request

request
~~~~~~~

:ref:`request example`.

Фикстура ``request`` fixture - это специальная фикстура, предоставляющая информацию о запрашиваемой функции тестирования.

.. autoclass:: pytest.FixtureRequest()
    :members:


.. fixture:: pytestconfig

pytestconfig
~~~~~~~~~~~~

.. autofunction:: _pytest.fixtures.pytestconfig()


.. fixture:: record_property

record_property
~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`record_property example`.

.. autofunction:: _pytest.junitxml.record_property()


.. fixture:: record_testsuite_property

record_testsuite_property
~~~~~~~~~~~~~~~~~~~~~~~~~

**Руководство**: :ref:`record_testsuite_property example`.

.. autofunction:: _pytest.junitxml.record_testsuite_property()


.. fixture:: caplog

caplog
~~~~~~

:ref:`logging`.

.. autofunction:: _pytest.logging.caplog()
    :no-auto-options:

    Возвращает экземпляр :class:`pytest.LogCaptureFixture`.

.. autoclass:: pytest.LogCaptureFixture()
    :members:


.. fixture:: monkeypatch

monkeypatch
~~~~~~~~~~~

:ref:`monkeypatching`.

.. autofunction:: _pytest.monkeypatch.monkeypatch()
    :no-auto-options:

    Возвращает экземпляр :class:`~pytest.MonkeyPatch`.

.. autoclass:: pytest.MonkeyPatch
    :members:


.. fixture:: pytester

pytester
~~~~~~~~

.. versionadded:: 6.2

Предоставляет экземпляр :class:`~pytest.Pytester`, который можно использовать для запуска и тестирования самого pytest.

Предоставляет пустой каталог, в котором pytest может выполняться изолированно, и содержит средства
для написания тестов, конфигурационных файлов и сопоставления с ожидаемым результатом.

Чтобы использовать, поместите в самый верхний файл ``conftest.py``:

.. code-block:: python

    pytest_plugins = "pytester"



.. autoclass:: pytest.Pytester()
    :members:

.. autoclass:: _pytest.pytester.RunResult()
    :members:

.. autoclass:: _pytest.pytester.LineMatcher()
    :members:
    :special-members: __str__

.. autoclass:: _pytest.pytester.HookRecorder()
    :members:

.. fixture:: testdir

testdir
~~~~~~~

Идентичен :fixture:`pytester`, но предоставляет экземпляр, чьи методы возвращают
унаследованные объекты ``py.path.local``, когда это применимо.

Новый код должен избегать использования :fixture:`testdir` в пользу :fixture:`pytester`.

.. autoclass:: pytest.Testdir()
    :members:


.. fixture:: recwarn

recwarn
~~~~~~~

**Руководство**: :ref:`assertwarnings`

.. autofunction:: _pytest.recwarn.recwarn()
    :no-auto-options:

.. autoclass:: pytest.WarningsRecorder()
    :members:

Каждое записанное предупреждение является экземпляром :class:`warnings.WarningMessage`.

.. note::
    ``DeprecationWarning`` и ``PendingDeprecationWarning`` рассматриваются
    по-разному; см. :ref:`ensuring_function_triggers`.


.. fixture:: tmp_path

tmp_path
~~~~~~~~

:ref:`tmpdir`

.. autofunction:: _pytest.tmpdir.tmp_path()
    :no-auto-options:


.. fixture:: tmp_path_factory

tmp_path_factory
~~~~~~~~~~~~~~~~

:ref:`tmp_path_factory example`

.. _`tmp_path_factory factory api`:

``tmp_path_factory`` - это экземпляр :class:`~pytest.TempPathFactory`:

.. autoclass:: pytest.TempPathFactory()


.. fixture:: tmpdir

tmpdir
~~~~~~

:ref:`tmpdir and tmpdir_factory`

.. autofunction:: _pytest.tmpdir.tmpdir()
    :no-auto-options:


.. fixture:: tmpdir_factory

tmpdir_factory
~~~~~~~~~~~~~~

:ref:`tmpdir and tmpdir_factory`

``tmp_path_factory`` это экземпляр :class:`~pytest.TempdirFactory`:

.. autoclass:: pytest.TempdirFactory()


.. _`hook-reference`:

Хуки
-----

:ref:`writing-plugins`.

.. currentmodule:: _pytest.hookspec

Ссылка на все хуки, которые могут быть реализованы :ref:`conftest.py files <localplugin>` и :ref:`plugins <plugins>`.

Загрузочные хуки
~~~~~~~~~~~~~~~~~~~

Хукм начальной загрузки вызываются для плагинов, зарегистрированных достаточно *рано* (внутренние плагины и плагины setuptools).

.. autofunction:: pytest_load_initial_conftests
.. autofunction:: pytest_cmdline_preparse
.. autofunction:: pytest_cmdline_parse
.. autofunction:: pytest_cmdline_main

.. _`initialization-hooks`:

Хуки инициализации
~~~~~~~~~~~~~~~~~~~~

Инициализационные хуки вызываются для плагинов и файлов ``conftest.py``.

.. autofunction:: pytest_addoption
.. autofunction:: pytest_addhooks
.. autofunction:: pytest_configure
.. autofunction:: pytest_unconfigure
.. autofunction:: pytest_sessionstart
.. autofunction:: pytest_sessionfinish

.. autofunction:: pytest_plugin_registered

Хуки для сбора
~~~~~~~~~~~~~~~~

``pytest`` вызывает следующие хуки для сбора файлов и каталогов

.. autofunction:: pytest_collection
.. autofunction:: pytest_ignore_collect
.. autofunction:: pytest_collect_file
.. autofunction:: pytest_pycollect_makemodule

Для воздействия на коллекцию объектов в модулях Python
можно использовать следующий хук:

.. autofunction:: pytest_pycollect_makeitem
.. autofunction:: pytest_generate_tests
.. autofunction:: pytest_make_parametrize_id

После завершения сбора можно изменить порядок расположения
элементов, удалить или иным образом изменить элементы теста:

.. autofunction:: pytest_collection_modifyitems

.. note::
    Если этот хук реализован в файлах ``conftest.py``, он всегда получает все собранные элементы, а не только те,
    которые находятся в ``conftest.py``, где он реализован.

.. autofunction:: pytest_collection_finish

Хуки для запуска тестов (runtest)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Все связанные с runtest хуки получают объект :py:class:`pytest.Item <pytest.Item>`.

.. autofunction:: pytest_runtestloop
.. autofunction:: pytest_runtest_protocol
.. autofunction:: pytest_runtest_logstart
.. autofunction:: pytest_runtest_logfinish
.. autofunction:: pytest_runtest_setup
.. autofunction:: pytest_runtest_call
.. autofunction:: pytest_runtest_teardown
.. autofunction:: pytest_runtest_makereport

Для более глубокого понимания вы можете посмотреть на стандартную реализацию
этих хуков в ``_pytest.runner`` и, возможно, также
в ``_pytest.pdb``, который взаимодействует с ``_pytest.capture``
и его перехватом ввода/вывода, чтобы немедленно перейти
в интерактивную отладку, когда происходит падение теста.

.. autofunction:: pytest_pyfunc_call

Хуки для составления отчетов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Связанные с сеансом обработчики отчетов:

.. autofunction:: pytest_collectstart
.. autofunction:: pytest_make_collect_report
.. autofunction:: pytest_itemcollected
.. autofunction:: pytest_collectreport
.. autofunction:: pytest_deselected
.. autofunction:: pytest_report_header
.. autofunction:: pytest_report_collectionfinish
.. autofunction:: pytest_report_teststatus
.. autofunction:: pytest_terminal_summary
.. autofunction:: pytest_fixture_setup
.. autofunction:: pytest_fixture_post_finalizer
.. autofunction:: pytest_warning_captured
.. autofunction:: pytest_warning_recorded

Центральный хук для отчетов о выполнении теста:

.. autofunction:: pytest_runtest_logreport

Хуки, связанные с ассертами:

.. autofunction:: pytest_assertrepr_compare
.. autofunction:: pytest_assertion_pass


Хуки отладки/взаимодействия
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Есть несколько хуков, которые могут быть использованы для специальных
отчетов или взаимодействия с исключениями:

.. autofunction:: pytest_internalerror
.. autofunction:: pytest_keyboard_interrupt
.. autofunction:: pytest_exception_interact
.. autofunction:: pytest_enter_pdb


Объекты
--------

Полная ссылка на объекты, доступные из :ref:`фикстур <fixture>` или :ref:`хуков <hook-reference>`.


CallInfo
~~~~~~~~

.. autoclass:: pytest.CallInfo()
    :members:


Class
~~~~~

.. autoclass:: pytest.Class()
    :members:
    :show-inheritance:

Collector
~~~~~~~~~

.. autoclass:: pytest.Collector()
    :members:
    :show-inheritance:

CollectReport
~~~~~~~~~~~~~

.. autoclass:: _pytest.reports.CollectReport()
    :members:
    :show-inheritance:
    :inherited-members:

Config
~~~~~~

.. autoclass:: _pytest.config.Config()
    :members:

ExceptionInfo
~~~~~~~~~~~~~

.. autoclass:: pytest.ExceptionInfo()
    :members:


ExitCode
~~~~~~~~

.. autoclass:: pytest.ExitCode
    :members:

File
~~~~

.. autoclass:: pytest.File()
    :members:
    :show-inheritance:


FixtureDef
~~~~~~~~~~

.. autoclass:: _pytest.fixtures.FixtureDef()
    :members:
    :show-inheritance:

FSCollector
~~~~~~~~~~~

.. autoclass:: _pytest.nodes.FSCollector()
    :members:
    :show-inheritance:

Function
~~~~~~~~

.. autoclass:: pytest.Function()
    :members:
    :show-inheritance:

FunctionDefinition
~~~~~~~~~~~~~~~~~~

.. autoclass:: _pytest.python.FunctionDefinition()
    :members:
    :show-inheritance:

Item
~~~~

.. autoclass:: pytest.Item()
    :members:
    :show-inheritance:

MarkDecorator
~~~~~~~~~~~~~

.. autoclass:: pytest.MarkDecorator()
    :members:


MarkGenerator
~~~~~~~~~~~~~

.. autoclass:: pytest.MarkGenerator()
    :members:


Mark
~~~~

.. autoclass:: pytest.Mark()
    :members:


Metafunc
~~~~~~~~

.. autoclass:: pytest.Metafunc()
    :members:

Module
~~~~~~

.. autoclass:: pytest.Module()
    :members:
    :show-inheritance:

Node
~~~~

.. autoclass:: _pytest.nodes.Node()
    :members:

Parser
~~~~~~

.. autoclass:: _pytest.config.argparsing.Parser()
    :members:


PytestPluginManager
~~~~~~~~~~~~~~~~~~~

.. autoclass:: _pytest.config.PytestPluginManager()
    :members:
    :undoc-members:
    :inherited-members:
    :show-inheritance:

Session
~~~~~~~

.. autoclass:: pytest.Session()
    :members:
    :show-inheritance:

TestReport
~~~~~~~~~~

.. autoclass:: _pytest.reports.TestReport()
    :members:
    :show-inheritance:
    :inherited-members:

_Result
~~~~~~~

Result, используемый внутри :ref:`обертки хуков <hookwrapper>`.

.. autoclass:: pluggy.callers._Result
.. automethod:: pluggy.callers._Result.get_result
.. automethod:: pluggy.callers._Result.force_result

Глобальные переменные
-----------------------

pytest обрабатывает некоторые глобальные переменные особым образом, когда они определены в тестовом модуле или в
файлах ``conftest.py``.


.. globalvar:: collect_ignore

**Руководство**: :ref:`customizing-test-collection`

Могут быть объявлены в файлах *conftest.py* для исключения тестовых каталогов или модулей.
Должны представлять собой список путей (``str``, :class:`pathlib.Path`` или любой :class:`os.PathLike``).

.. code-block:: python

  collect_ignore = ["setup.py"]


.. globalvar:: collect_ignore_glob

**Руководство**: :ref:`customizing-test-collection`

Могут быть объявлены в файлах *conftest.py* для исключения тестовых каталогов или модулей
с помощью подстановочных знаков в стиле оболочки Unix. Должны быть ``списком[str]``, где ``str`` может
содержать шаблоны поиска.

.. code-block:: python

  collect_ignore_glob = ["*_ignore.py"]


.. globalvar:: pytest_plugins

**Руководство**: :ref:`available installable plugins`

Может быть объявлено на **глобальном** уровне в *тестовых модулях* и *conftest.py файлах* для регистрации дополнительных плагинов.
Может быть либо ``str``, либо ``Sequence[str]``.

.. code-block:: python

    pytest_plugins = "myapp.testsupport.myplugin"

.. code-block:: python

    pytest_plugins = ("myapp.testsupport.tools", "myapp.testsupport.regression")


.. globalvar:: pytestmark

**Руководство**: :ref:`scoped-marking`

Могут быть объявлены на **глобальном** уровне в *тестовых модулях* для применения одной или нескольких :ref:`иаркировок <marks ref>` ко всем
тестовым функциям и методам. Это может быть как одна маркировка, так и список маркировок (применяемых в порядке слева направо).

.. code-block:: python

    import pytest

    pytestmark = pytest.mark.webtest


.. code-block:: python

    import pytest

    pytestmark = [pytest.mark.integration, pytest.mark.slow]


Переменные среды
---------------------

Переменные среды, которые можно использовать для изменения поведения pytest.

.. envvar:: PYTEST_ADDOPTS

Содержит командную строку (разбираемую модулем py:mod:`shlex`), которая будет **дополнена** к командной строке, заданной
пользователем, более подробную информацию смотрите в :ref:`adding default options`.

.. envvar:: PYTEST_CURRENT_TEST

Этот параметр не предназначен для установки пользователями, но устанавливается внутри pytest с именем текущего теста, поэтому другие
процессы могут изменить его, смотрите :ref:`pytest current test env` для дополнительной информации.

.. envvar:: PYTEST_DEBUG

Когда установлено, pytest будет печатать трассировку и отладочную информацию..

.. envvar:: PYTEST_DISABLE_PLUGIN_AUTOLOAD

Если установлено, отключает автоматическую загрузку плагина через точки входа setuptools. Будут загружены только явно указанные плагины.

.. envvar:: PYTEST_PLUGINS

Содержит разделенный запятыми список модулей, которые должны быть загружены как плагины:

.. code-block:: bash

    export PYTEST_PLUGINS=mymodule.plugin,xdist

.. envvar:: PY_COLORS

Если установлено значение ``1``, pytest будет использовать цветовой выводе в терминале.
При значении ``0`` pytest не будет использовать цветовой вывод.
``PY_COLORS`` имеет приоритет над ``NO_COLOR`` и ``FORCE_COLOR``.

.. envvar:: NO_COLOR

Если установлено(независимо от значения), pytest не будет использовать цветовой вывод в терминале.
``PY_COLORS`` имеет приоритет над ``NO_COLOR``, который имеет приоритет над ``FORCE_COLOR``.
Смотрите `no-color.org <https://no-color.org/>`__ для других библиотек, поддерживающих этот стандарт сообщества.

.. envvar:: FORCE_COLOR

Если установлено(независимо от значения), pytest будет использовать цветовой вывод в терминале.
``PY_COLORS`` и ``NO_COLOR`` иметь приоритет над ``FORCE_COLOR``.

Исключения
-----------

.. autoclass:: pytest.UsageError()
    :show-inheritance:

.. _`warnings ref`:

Предупреждения
---------------

Пользовательские предупреждения, генерируемые в некоторых ситуациях, таких как неправильное использование или устаревшие функции.

.. autoclass:: pytest.PytestWarning
   :show-inheritance:

.. autoclass:: pytest.PytestAssertRewriteWarning
   :show-inheritance:

.. autoclass:: pytest.PytestCacheWarning
   :show-inheritance:

.. autoclass:: pytest.PytestCollectionWarning
   :show-inheritance:

.. autoclass:: pytest.PytestConfigWarning
   :show-inheritance:

.. autoclass:: pytest.PytestDeprecationWarning
   :show-inheritance:

.. autoclass:: pytest.PytestExperimentalApiWarning
   :show-inheritance:

.. autoclass:: pytest.PytestUnhandledCoroutineWarning
   :show-inheritance:

.. autoclass:: pytest.PytestUnknownMarkWarning
   :show-inheritance:

.. autoclass:: pytest.PytestUnraisableExceptionWarning
   :show-inheritance:

.. autoclass:: pytest.PytestUnhandledThreadExceptionWarning
   :show-inheritance:


Обратитесь к разделу :ref:`internal-warnings` в документации для получения дополнительной информации.


.. _`ini options ref`:

Опции конфигурации
-----------------------

Здесь приведен список встроенных опций конфигурации, которые могут быть записаны в файлы ``pytest.ini``, ``pyproject.toml``, ``tox.ini`` или ``setup.cfg``,
обычно расположенные в корне вашего репозитория. Для подробного ознакомления с каждым форматом файла см.
:ref:`config file formats`.

.. warning::
    Использование ``setup.cfg`` не рекомендуется, за исключением очень простых случаев. ``.cfg``
    файлы используют парсер, отличный от ``pytest.ini`` и ``tox.ini``, что может привести к трудно отслеживаемым проблемам.
    Когда это возможно, рекомендуется использовать вышеперечисленные файлы, или ``pyproject.toml``, для хранения конфигурации pytest.

Параметры конфигурации могут быть перезаписаны командной строкой с помощью ``-o/--override-ini``, которые также могут быть
передаваться несколько раз. Ожидаемый формат - ``имя = значение``. Например::

   pytest -o console_output_style=classic -o cache_dir=/tmp/mycache


.. confval:: addopts

   Добавляет указанные ``OPTS`` к набору аргументов командной строки, как если бы они были
   указаны пользователем. Пример: при следующем содержимое ini файла:

   .. code-block:: ini

        # листинг pytest.ini
        [pytest]
        addopts = --maxfail=2 -rf  # exit after 2 failures, report fail info

   выдача ``pytest test_hello.py`` на самом деле означает:

   .. code-block:: bash

        pytest --maxfail=2 -rf test_hello.py

   По умолчанию параметры не добавляются.


.. confval:: cache_dir

   Устанавливает директорию, в которой хранится содержимое плагина кэша. По умолчанию это каталог
   ``.pytest_cache``, который создается в :ref:`rootdir <rootdir>`. У каталога может быть
   относительный или абсолютный путь. Если задан относительный путь, то каталог создается
   относительно :ref:`rootdir <rootdir>`. Кроме того, путь может содержать переменные
   окружения, которые будут расширены. Для получения дополнительной информации о плагине кэширования
   обратитесь к :ref:`cache_provider`.


.. confval:: confcutdir

   Задает директорию, в которой прекращается поиск *снизу-вверх* файлов ``conftest.py``.
   По умолчанию pytest прекращает поиск файлов ``conftest.py`` вверх по каталогу
   от ``pytest.ini``/ ``tox.ini``/ ``setup.cfg`` проекта, если таковой имеется,
   или до корня файловой системы.


.. confval:: console_output_style



   Устанавливает стиль вывода консоли при запуске тестов:

   * ``classic``: классический вывод pytest.
   * ``progress``: как классический вывод pytest, но с индикатором прогресса.
   * ``count``: как ``progress``, но показывает прогресс в виде количества выполненных тестов, а не в процентах.

   По умолчанию используется ``progress``, но вы можете вернуться к ``classic``, если вы предпочитаете или
   новый режим вызывает неожиданные проблемы:

   .. code-block:: ini

        # листинг pytest.ini
        [pytest]
        console_output_style = classic


.. confval:: doctest_encoding



   Кодировка по умолчанию, используемая для декодирования текстовых файлов со строками документации.
   :ref:`Посмотрите, как pytest обрабатывает doctests <doctest>`.


.. confval:: doctest_optionflags

   Одно или несколько имен флагов doctest из стандартного модуля ``doctest``.
   :ref:`Посмотрите, как pytest обрабатывает doctests <doctest>`.


.. confval:: empty_parameter_set_mark



    Позволяет выбрать действие для пустых наборов параметров в параметризации

    * ``skip`` пропускает тесты с пустым набором параметров (по умолчанию)
    * ``xfail`` отмечает тесты с пустым набором параметров как xfail(run=False)
    * ``fail_at_collect`` вызывает исключение, если параметризация собирает пустой набор параметров

    .. code-block:: ini

      # листинг pytest.ini
      [pytest]
      empty_parameter_set_mark = xfail

    .. note::

      Значение по умолчанию этой опции планируется изменить на ``xfail`` в будущих релизах
      так как это считается менее подверженным ошибкам, см. `#3155 <https://github.com/pytest-dev/pytest/issues/3155>`_
      для более подробной информации.


.. confval:: faulthandler_timeout

   Выгружает трассировку всех потоков, если тест выполняется более ``X`` секунд (включая
   установку и удаление фикстур). Реализовано с помощью функции `faulthandler.dump_traceback_later`_,
   поэтому все предостережения применимы.

   .. code-block:: ini

        # листинг pytest.ini
        [pytest]
        faulthandler_timeout=5

   Для получения дополнительной информации, пожалуйста, обратитесь к :ref:`faulthandler`.

.. _`faulthandler.dump_traceback_later`: https://docs.python.org/3/library/faulthandler.html#faulthandler.dump_traceback_later


.. confval:: filterwarnings



   Устанавливает список фильтров и действий, которые должны быть предприняты для совпавших
   предупреждений. По умолчанию все предупреждения, выданные во время сеанса тестирования
   будут отображаться в сводке в конце сеанса тестирования.

   .. code-block:: ini

        # листинг pytest.ini
        [pytest]
        filterwarnings =
            error
            ignore::DeprecationWarning

   Это указывает pytest игнорировать предупреждения об устаревании и превращать все остальные предупреждения
   в ошибки. Более подробную информацию можно найти в :ref:`warnings`.


.. confval:: junit_duration_report

    .. versionadded:: 4.1

    Настраивает способ записи продолжительности в отчет JUnit XML:

    * ``total`` (по умолчанию): указанное время включает время установки, вызова и очистки.
    * ``call``: сообщаемое время продолжительности включает только время вызова, исключая настройку и очистку.

    .. code-block:: ini

        [pytest]
        junit_duration_report = call


.. confval:: junit_family

    .. versionadded:: 4.2
    .. versionchanged:: 6.1
        Значение по умолчанию изменено на ``xunit2``.

    Настраивает формат создаваемого XML-файла JUnit. Возможные варианты:

    * ``xunit1`` (или ``legacy``): производит вывод в старом стиле, совместимый с форматом xunit 1.0.
    * ``xunit2``: производит `вывод в стиле xunit 2.0 <https://github.com/jenkinsci/xunit-plugin/blob/xunit-2.3.2/src/main/resources/org/jenkinsci/plugins/xunit/types/model/xsd/junit-10.xsd>`__,
      который должен быть более совместим с последними версиями Jenkins. **Значение по умолчанию**.

    .. code-block:: ini

        [pytest]
        junit_family = xunit2


.. confval:: junit_logging

    .. versionadded:: 3.5
    .. versionchanged:: 5.4
        добавлены опции ``log``, ``all``, ``out-err``.

    Настраивает, следует ли записывать захваченный вывод в файл JUnit XML. Допустимые значения:

    * ``log``: записывать только захваченный вывод ``logging``.
    * ``system-out``: записать захваченное содержимое ``stdout``.
    * ``system-err``: записать захваченное содержимое ``stderr``.
    * ``out-err``: записать захваченное содержимое ``stdout`` и ``stderr``.
    * ``all``: записать захваченное содержимое ``logging``, ``stdout`` и ``stderr``.
    * ``no`` (по умолчанию): захваченный вывод не записывается.

    .. code-block:: ini

        [pytest]
        junit_logging = system-out


.. confval:: junit_log_passing_tests

    .. versionadded:: 4.6

    Если ``junit_logging != "no"``, настраивает, должен ли перехваченный вывод записываться
    в XML-файл JUnit для **успешных** тестов. По умолчанию ``True``.

    .. code-block:: ini

        [pytest]
        junit_log_passing_tests = False


.. confval:: junit_suite_name

    Чтобы задать имя корневого элемента xml тестового набора, вы можете настроить опцию ``junit_suite_name`` в вашем конфигурационном файле:

    .. code-block:: ini

        [pytest]
        junit_suite_name = my_suite

.. confval:: log_auto_indent

    Разрешить выборочный автоматический отступ для многострочных сообщений журнала.

    Поддерживает опцию командной строки ``--log-auto-indent [value]``
    и опцию конфигурации ``log_auto_indent = [value]`` для установки
    авто-отступа для всех журналов.

    ``[value]`` может быть:
        * True или "On" - Многострочные сообщения журнала с динамическим автоматическим отступом
        * False или "Off" или 0 - Не использовать автоматический отступ для многострочных сообщений журнала (поведение по умолчанию)
        * [positive integer] - автоматический отступ многострочных сообщений журнала на [value] пробелов

    .. code-block:: ini

        [pytest]
        log_auto_indent = False

    Поддерживает передачу kwarg ``extra={"auto_indent": [value]}`` в
    в вызовы ``logging.log()`` для указания поведения авто-отступов для
    конкретной записи в журнале. kwarg ``extra`` переопределяет значение, указанное
    в командной строке или в конфигурации.

.. confval:: log_cli

    Включить отображение журнала во время тестового запуска (также известное как :ref:`"логирование в реальном времени" <live_logs>`).
    По умолчанию значение ``False``.

    .. code-block:: ini

        [pytest]
        log_cli = True

.. confval:: log_cli_date_format



    Устанавливает :py:func:`time.strftime`-совместимая строка, которая будет использоваться при форматировании дат для записи в реальном времени.

    .. code-block:: ini

        [pytest]
        log_cli_date_format = %Y-%m-%d %H:%M:%S

    Для получения дополнительной информации см. :ref:`live_logs`.

.. confval:: log_cli_format



    Устанавливает совместимую с :py:mod:`logging` строку, используемую для форматирования сообщений журнала в реальном времени.

    .. code-block:: ini

        [pytest]
        log_cli_format = %(asctime)s %(levelname)s %(message)s

    Для получения дополнительной информации см. :ref:`live_logs`.


.. confval:: log_cli_level



    Устанавливает минимальный уровень сообщений журнала, который должен быть захвачен для ведения журнала в реальном времени. Можно использовать целочисленное значение или
    имена уровней.

    .. code-block:: ini

        [pytest]
        log_cli_level = INFO

    Для получения дополнительной информации см. :ref:`live_logs`.


.. confval:: log_date_format



    Устанавливает :py:func:`time.strftime`-совместимую строку, которая будет использоваться при форматировании дат для захвата в журнал.

    .. code-block:: ini

        [pytest]
        log_date_format = %Y-%m-%d %H:%M:%S

    For more information, see :ref:`logging`.


.. confval:: log_file



    Задает имя файла относительно файла ``pytest.ini``, в который должны записываться сообщения журнала, в дополнение к
    к другим активным средствам протоколирования.

    .. code-block:: ini

        [pytest]
        log_file = logs/pytest-logs.txt

    For more information, see :ref:`logging`.


.. confval:: log_file_date_format



    Устанавливает :py:func:`time.strftime`-совместимую строку, которая будет использоваться при форматировании дат для файла журнала.

    .. code-block:: ini

        [pytest]
        log_file_date_format = %Y-%m-%d %H:%M:%S

    For more information, see :ref:`logging`.

.. confval:: log_file_format



    Устанавливает :py:mod:`logging`-совместимую строку, используемую для форматирования сообщений регистрации, перенаправляемых в файл регистрации.

    .. code-block:: ini

        [pytest]
        log_file_format = %(asctime)s %(levelname)s %(message)s

    Больше информации см. в :ref:`logging`.

.. confval:: log_file_level



    Устанавливает минимальный уровень сообщений журнала, который должен быть захвачен для файла журнала. Можно использовать целочисленное значение или
    можно использовать имена уровней.

    .. code-block:: ini

        [pytest]
        log_file_level = INFO

    Больше информации см. в :ref:`logging`.


.. confval:: log_format



    Устанавливает :py:mod:`logging`-совместимую строку, используемую для форматирования перехваченных сообщений регистрации.

    .. code-block:: ini

        [pytest]
        log_format = %(asctime)s %(levelname)s %(message)s

    Больше информации см. в :ref:`logging`.


.. confval:: log_level



    Устанавливает минимальный уровень сообщений журнала, который должен быть захвачен для перехвата журнала. Можно использовать целочисленное значение или
    можно использовать имена уровней.

    .. code-block:: ini

        [pytest]
        log_level = INFO

    Больше информации см. в :ref:`logging`.


.. confval:: markers

    При использовании аргументов командной строки ``--strict-markers`` или ``--strict``,
    разрешены только известные маркеры - определенные в коде основным pytest или каким-либо плагином.

    Вы можете перечислить дополнительные маркеры в этом параметре, чтобы добавить их в белый список,
    в этом случае вы, вероятно, захотите добавить ``--strict-markers`` в ``addopts``.
    чтобы избежать будущих регрессий:

    .. code-block:: ini

        [pytest]
        addopts = --strict-markers
        markers =
            slow
            serial

    .. note::
        Использование ``--strict-markers`` является весьма предпочтительным. ``--strict`` был сохранен только для
        обратной совместимости и может сбивать с толку других, поскольку применяется только к
        маркерам, а не к другим опциям.

.. confval:: minversion

   Задает минимальную версию pytest, необходимую для запуска тестов.

   .. code-block:: ini

        # листинг pytest.ini
        [pytest]
        minversion = 3.0  # will fail if we run with pytest-2.8


.. confval:: norecursedirs

   Устанавливает шаблоны базовых имен каталогов, которые следует избегать при повторном поиске
   для обнаружения тестов.  Отдельные шаблоны (в стиле fnmatch) применяются
   к основному имени каталога, чтобы решить, следует ли в него обращаться.
   Символы сопоставления шаблонов::

        *       соответствует всему
        ?       соответствует любому одиночному символу
        [seq]   соответствует любому символу в seq
        [!seq]  соответствует любому символу не в seq

   Шаблоны по умолчанию: ``'*.egg'``, ``'*'``, ``'_darcs'``, ``'build'``,
   ``'CVS'``, ``'dist'``, ``'node_modules'``, ``'venv'``, ``'{arch}'``.
   Установка ``norecursedirs`` заменяет значение по умолчанию.  Вот пример того,
   как избежать определенных каталогов:

   .. code-block:: ini

        [pytest]
        norecursedirs = .svn _build tmp*

   Это указывает ``pytest`` не искать в типичных каталогах subversion или
   sphinx-build или в любой каталог с префиксом ``tmp``.

   Кроме того, ``pytest`` будет пытаться интеллектуально определить и игнорировать
   virtualenv по наличию скрипта активации.  Любой каталог, считающийся
   корнем виртуального окружения, не будет рассматриваться во время сбора тестов,
   если не задан параметр ``‑‑collect‑in‑virtualenv``. Обратите также внимание, что
   ``norecursedirs`` имеет приоритет над ``collect-in-virtualenv``; например, если
   вы собираетесь запускать тесты в virtualenv с базовым каталогом, который соответствует
   ``.*``, вы *должны* переопределить ``norecursedirs`` в дополнение к использованию опции
   флага ``--collect-in-virtualenv``.


.. confval:: python_classes

   Один или несколько префиксов имен или шаблонов в стиле glob, определяющих, какие классы
   рассматриваются для сбора тестов. Для поиска нескольких шаблонов glob
   добавляется пробел между шаблонами. По умолчанию pytest будет рассматривать любой
   класс с префиксом ``Test`` в качестве коллекции тестов. Вот пример того, как
   собирать тесты из классов, которые заканчиваются на ``Suite``:

   .. code-block:: ini

        [pytest]
        python_classes = *Suite

   Обратите внимание, что производные классы ``unittest.TestCase`` всегда собираются
   независимо от этой опции, так как используется собственный механизм сборки ``unittest``
   для сбора этих тестов.


.. confval:: python_files

   Один или несколько шаблонов файлов в стиле Glob, определяющих, какие файлы python
   рассматриваются как тестовые модули. Поиск по нескольким шаблонам glob осуществляется путем
   добавления пробела между шаблонами:

   .. code-block:: ini

        [pytest]
        python_files = test_*.py check_*.py example_*.py

   Или по одному в строке:

   .. code-block:: ini

        [pytest]
        python_files =
            test_*.py
            check_*.py
            example_*.py

   По умолчанию, файлы, соответствующие ``test_*.py`` и ``*_test.py`` будут считаться
   тестовыми модулями.


.. confval:: python_functions

   Один или несколько префиксов имен или glob-шаблонов, определяющих, какие тестовые функции
   и методы считаются тестами. Для поиска нескольких шаблонов glob
   добавляется пробел между шаблонами. По умолчанию pytest будет считать тестом любую
   функцию с префиксом ``test`` как тест.  Вот пример того, как
   собирать тестовые функции и методы, которые заканчиваются на ``_test``:

   .. code-block:: ini

        [pytest]
        python_functions = *_test

   Обратите внимание, что это не влияет на методы, которые живут в ``unittest.TestCase`` производного класса,
   так как используется собственная структура коллекции ``unittest`` для сбора этих тестов.

   См. :ref:`change naming conventions` для более подробных примеров.


.. confval:: required_plugins

   Список плагинов, которые должны присутствовать для запуска pytest, разделенный пробелами.
   Плагины могут быть перечислены с указателями версии или без них непосредственно после
   их именем. Пробелы между разными спецификаторами версий не допускаются.
   Если какой-либо из плагинов не найден, выдается ошибка.

   .. code-block:: ini

       [pytest]
       required_plugins = pytest-django>=3.0.0,<4.0.0 pytest-html pytest-xdist>=1.0.0


.. confval:: testpaths



   Задает список каталогов, в которых следует искать тесты, когда
   в командной строке не указаны конкретные каталоги, файлы или идентификаторы тестов, когда
   выполняется pytest из каталога :ref:`rootdir <rootdir>`.
   Полезно, когда все тесты проекта находятся в известном месте, чтобы ускорить
   сбор тестов и избежать случайной выборки нежелательных тестов.

   .. code-block:: ini

        [pytest]
        testpaths = testing doc

   Это указывает pytest искать тесты только в каталогах ``testing`` и ``doc``
   при выполнении из корневого каталога.


.. confval:: usefixtures

    Список фикстур, которые будут применяться ко всем тестовым функциям; семантически это то же самое, что применять маркер
    ``@pytest.mark.usefixtures`` ко всем тестовым функциям.


    .. code-block:: ini

        [pytest]
        usefixtures =
            clean_db


.. confval:: xfail_strict

    Если установлено значение ``True``, тесты, помеченные ``@pytest.mark.xfail``, которые на самом деле успешны, по умолчанию упадут.
    Для получения дополнительной информации см. :ref:`xfail strict tutorial`.


    .. code-block:: ini

        [pytest]
        xfail_strict = True


.. _`command-line-flags`:

Command-line Flags
------------------

Все флаги командной строки можно получить, запустив ``pytest --help``::

    $ pytest --help
    использование: pytest [опции] [файл_или_директория] [файл_или_директория] [...]

    позиционные аргументы:
      файл_или_директория

    общие:
      -k EXPRESSION         запускать только тесты, которые соответствуют заданному подстрочному
                            выражение. Выражение - это вычисляемое в python
                            выражение, в котором все имена сопоставляются с подстрокой
                            с именами тестов и их родительских классов.
                            Пример: -k 'test_method or test_other' соответствует всем
                            тестовым функциям и классам, имя которых содержит
                            'test_method' или 'test_other', в то время как -k 'not
                            test_method" соответствует тем, которые не содержат
                            'test_method' в своих именах. -k 'not test_method
                            and not test_other" исключает совпадения.
                            Дополнительно ключевые слова сопоставляются с классами и
                            функциям, содержащим дополнительные имена в их
                            'extra_keyword_matches', а также с функциями,
                            которые имеют имена, присвоенные непосредственно им.
                            Сопоставление не чувствительно к регистру.
      -m MARKEXPR           запускать только тесты, соответствующие заданному отмеченному выражению.
                            Например: -m 'mark1 and not mark2'.
      --markers             показать маркеры (встроенные, подключаемые и индивидуальные для каждого проекта).
      -x, --exitfirst       немедленно выйти при первой ошибке или упавшем тесте.
      --fixtures, --funcargs
                            показать доступные фикстуры, отсортированные по видимости плагина
                            (фикстуры с впереди стоящим '_' будут показаны только с '-v')
      --fixtures-per-test   показать фикстуры для каждого испытания
      --pdb                 запускать интерактивный отладчик Python при ошибках или при прерывании
                            KeyboardInterrupt.
      --pdbcls=modulename:classname
                            запускать пользовательский интерактивный отладчик Python при
                            ошибки. Например:
                            --pdbcls=IPython.terminal.debugger:TerminalPdb
      --trace               вызовет отладчик Python в начале каждого теста.
      --capture=method      метод захвата теста: один из fd|sys|no|tee-sys.
      -s                    сокращенный вариант для --capture=no.
      --runxfail            сообщать результаты тестов xfail так, как если бы они были
                            не помечены
      --lf, --last-failed   повторно запускать только те тесты, которые упали при последнем запуске (или
                            все, если все прошли)
      --ff, --failed-first  запустить все тесты, но сначала запустить последние упавшие.
                            Это может изменить порядок тестов и, следовательно, привести к повторному запуску
                            фикстур setup/teardown.
      --nf, --new-first     сначала запускать тесты из новых файлов, затем остальные
                            тесты, отсортированные по mtime файла
      --cache-show=[CACHESHOW]
                            показать содержимое кэша, не выполнять сборку или
                            проверки. Необязательный аргумент: glob (по умолчанию: '*').
      --cache-clear         удалить все содержимое кеша при запуске теста.
      --lfnf={all,none}, --last-failed-no-failures={all,none}
                            какие тесты запускать без ранее (известных)
                            падений.
      --sw, --stepwise      выйти при падении теста и продолжить с последнего упавшего теста в
                            следующий раз
      --sw-skip, --stepwise-skip
                            игнорировать первый упавший тест, но останавливаться на следующем
                            упавшем тесте

    составление отчетов:
      --durations=N         показать N самых медленных длительностей setup/test (N = 0 для всех).
      --durations-min=N     Минимальная продолжительность в секундах для включения в список самых медленных.
                            По умолчанию 0.005
      -v, --verbose         увеличить объем вывода информации.
      --no-header           отключить заголовок.
      --no-summary          отключить сводку
      -q, --quiet           уменьшение объема вывода информации.
      --verbosity=VERBOSE   установка verbosity. По умолчанию 0.
      -r chars              показать дополнительную краткую информацию о тесте, заданную символами:
                            (f)ailed, (E)rror, (s)kipped, (x)failed, (X)passed,
                            (p)assed, (P)assed с выходом, (a)ll кроме пройденного
                            (p/P), или (A)ll. (w)arnings-предупреждения включены по умолчанию
                            (см. --disable-warnings), 'N' может быть использовано для сброса
                            списка(по умолчанию: 'fE').
      --disable-warnings, --disable-pytest-warnings
                            отключить сводку предупреждений
      -l, --showlocals      показывать локали в трассировке(отключено по умолчанию).
      --tb=style            режим печати с отслеживанием
                            (auto/long/short/line/native/no).
      --show-capture={no,stdout,stderr,log,all}
                            Управляет тем, как перехваченный stdout/stderr/log будет показан на
                            упавших тестах. По умолчанию - 'all'.
      --full-trace          не вырезать никаких трассировок (по умолчанию вырезано).
      --color=color         цветной вывод терминала(yes/no/auto).
      --code-highlight={yes,no}
                            Следует ли выделять код (только если --color
                            также включен)
      --pastebin=mode       отправлять failed|all информацию в сервис bpaste.net.
      --junit-xml=path      создание файл отчета в стиле junit-xml по заданному пути.
      --junit-prefix=str    добавить префикс к именам классов в выходных данных junit-xml.

    pytest-предупреждения:
      -W PYTHONWARNINGS, --pythonwarnings=PYTHONWARNINGS
                            установить, о каких предупреждениях сообщать, см. опцию -W
                            самого python.
      --maxfail=num         выйти после num числа падений или ошибок.
      --strict-config       любые предупреждения, возникающие при разборе раздела `pytest`
                            в разделе конфигурационного файла, вызывающие ошибки.
      --strict-markers      Маркеры, не зарегистрированные в разделе `markers`
                            конфигурационного файла, вызывают ошибки.
      --strict              (устарело) алиас для --strict-markers.
      -c file               загружать конфигурацию из `file` вместо того, чтобы пытаться
                            найти один из неявных конфигурационных файлов.
      --continue-on-collection-errors
                            Принудительное выполнение теста даже при ошибках сборки.
      --rootdir=ROOTDIR     Определите корневой каталог для тестов. Может быть относительным путем:
                            'root_dir', './root_dir',
                            'root_dir/another_dir/'; абсолютным путем:
                            '/home/user/root_dir'; путь с переменными:
                            '$HOME/root_dir'.

    сборка:
      --collect-only, --co  только собирать тесты, не выполнять их.
      --pyargs              попробовать интерпретировать все аргументы как пакеты python.
      --ignore=path         игнорировать путь во время сборки (разрешено несколько).
      --ignore-glob=path    игнорировать шаблон пути во время сборки(разрешено
                            несколько).
      --deselect=nodeid_prefix
                            отменить выбор(через префикс идентификатора узла) во время сборки
                            (разрешено несколько).
      --confcutdir=dir      загружать только conftest.py относительно указанного каталога.
      --noconftest          Не загружать любой файл conftest.py.
      --keep-duplicates     Сохраняйть повторяющиеся тесты.
      --collect-in-virtualenv
                            Не игнорировать тесты в локальном каталоге virtualenv
      --import-mode={prepend,append,importlib}
                            добавление/дополнение к sys.path при импорте тестовых
                            модулей и файлов conftest, по умолчанию - добавлять(prepend).
      --doctest-modules     запустить doctests во всех модулях .py
      --doctest-report={none,cdiff,ndiff,udiff,only_first_failure}
                            выберите другой формат вывода диффов doctest
                            при падении
      --doctest-glob=pat    шаблон соответствия файла doctests, по умолчанию: test*.txt
      --doctest-ignore-import-errors
                            игнорировать ImportErrors doctest-а
      --doctest-continue-on-failure
                            для данного doctest, продолжить выполнение после первого падения.

    Отладка и настройка тестовой сессии:
      --basetemp=dir        базовый временный каталог для этого тестового прогона(внимание:
                            этот каталог удаляется, если он существует)
      -V, --version         отображать версию pytest и информацию о плагинах.
                            Если указано дважды, также отображать информацию
                            о плагинах.
      -h, --help            показать помощь и конфигурационную информацию
      -p name               предварительная загрузка заданного имени модуля плагина или точки входа
                            (разрешено несколько).
                            Чтобы избежать загрузки плагинов, используйте префикс `no:`,
                            например, `no:doctest`.
      --trace-config        трассировка файлов conftest.py.
      --debug               хранить информацию об отладке внутренней трассировки в
                            'pytestdebug.log'.
      -o OVERRIDE_INI, --override-ini=OVERRIDE_INI
                            переопределить опцию ini со стилем "option=value", например:
                            `-o xfail_strict=True -o cache_dir=cache`.
      --assert=MODE         Управление инструментами отладки утверждений.
                            'plain' не производит отладку утверждений.
                            'rewrite' (по умолчанию) переписывает утверждения утверждений
                            в тестовых модулях при импорте для предоставления информации об утверждениях
                            информацию о выражении assert.
      --setup-only          только настройка фикстур, тесты не выполняются.
      --setup-show          показать настройку фикстур при выполнении тестов.
      --setup-plan          показать, какие фикстуры и тесты будут выполнены, но
                            ничего не выполнять.

    логирование:
      --log-level=LEVEL     уровень сообщений для перехвата/отображения.
                            По умолчанию не установлено, поэтому зависит от корневого/родительского
                            эффективного уровня обработчика журнала, где он равен "WARNING"
                            по умолчанию.
      --log-format=LOG_FORMAT
                            формат журнала, используемый модулем ведения журнала.
      --log-date-format=LOG_DATE_FORMAT
                            формат даты журнала, используемый модулем ведения журнала.
      --log-cli-level=LOG_CLI_LEVEL
                            уровень ведения журнала cli.
      --log-cli-format=LOG_CLI_FORMAT
                            формат журнала, используемый модулем ведения журнала.
      --log-cli-date-format=LOG_CLI_DATE_FORMAT
                            формат даты журнала, используемый модулем ведения журнала.
      --log-file=LOG_FILE   путь к файлу, куда лог будет записан.
      --log-file-level=LOG_FILE_LEVEL
                            уровень ведения файла лога.
      --log-file-format=LOG_FILE_FORMAT
                            формат журнала, используемый модулем ведения журнала.
      --log-file-date-format=LOG_FILE_DATE_FORMAT
                            формат даты журнала, используемый модулем ведения журнала.
      --log-auto-indent=LOG_AUTO_INDENT
                            Авто отступ многострочных сообщений, передаваемых модулю протоколирования.
                            Принимает значения true|on, false|off или целое число.

    [pytest] ini-опции в первом найденном файле pytest.ini|tox.ini|setup.cfg:

      markers (linelist):   маркеры для тестовой функции
      empty_parameter_set_mark (string):
                            маркер по умолчанию для пустых наборов параметров
      norecursedirs (args): шаблоны каталогов, которых следует избегать при рекурсии
      testpaths (args):     каталоги для поиска тестов, когда нет файлов или
                            каталогов, указанных в командной строке.
      filterwarnings (linelist):
                            Каждая строка определяет шаблон для
                            warnings.filterwarnings. Обрабатывается после
                            -W/--pythonwarnings.
      usefixtures (args):   список фикстур по умолчанию, которые будут использоваться с этим
                            проектом
      python_files (args):  шаблоны файлов в стиле glob для обнаружения тестового модуля Python
                            модуля Python
      python_classes (args):
                            префиксы или глобальные имена для обнаружения тестового
                            класса Python
      python_functions (args):
                            префиксы или глобальные имена для функции тестирования Python и
                            обнаружения методов
      disable_test_id_escaping_and_forfeit_all_rights_to_community_support (bool):
                            отключить строковые escape-символы, отличные от ascii, могут вызвать нежелательные
                            побочные эффекты (используйте на свой страх и риск)
      console_output_style (string):
                            вывод в консоль: «классический» или с дополнительной информацией о ходе выполнения
                            ("progress" (в процентах) | "count").
      xfail_strict (bool):  по умолчанию для строгого параметра маркеров xfail,
                            если он не указан явно (по умолчанию: False)
      enable_assertion_pass_hook (bool):
                            Включает хук pytest_assertion_pass.Убедитесь, что
                            удалили все ранее созданные файлы кэша .pyc.
      junit_suite_name (string):
                            Название набора тестов для отчета JUnit.
      junit_logging (string):
                            Записывать захваченные сообщения журнала в отчет JUnit: одно из
                            no|log|system-out|system-err|out-err|all
      junit_log_passing_tests (bool):
                            Сбор информации журнала для прохождения тестов в JUnit
                            отчет:
      junit_duration_report (string):
                            Продолжительность отчета: одно из total|call
      junit_family (string):
                            Выделить XML для схемы: один из legacy|xunit1|xunit2
      doctest_optionflags (args):
                            флаги опций для doctests
      doctest_encoding (string):
                            кодировка, используемая для файлов doctest
      cache_dir (string):   путь к каталогу кеша.
      log_level (string):   значение по умолчанию для --log-level
      log_format (string):  значение по умолчанию для --log-format
      log_date_format (string):
                            значение по умолчанию для --log-date-format
      log_cli (bool):       включить отображение журнала во время тестового запуска (также известно как
                            "логирование в реальном времени").
      log_cli_level (string):
                            значение по умолчанию для --log-cli-level
      log_cli_format (string):
                            значение по умолчанию для --log-cli-format
      log_cli_date_format (string):
                            значение по умолчанию для --log-cli-date-format
      log_file (string):    значение по умолчанию для --log-file
      log_file_level (string):
                            значение по умолчанию для --log-file-level
      log_file_format (string):
                            значение по умолчанию для --log-file-format
      log_file_date_format (string):
                            значение по умолчанию для --log-file-date-format
      log_auto_indent (string):
                            значение по умолчанию для --log-auto-indent
      faulthandler_timeout (string):
                            Выгрузка трассировки всех потоков, если для завершения теста
                            требуется более TIMEOUT секунд.
      addopts (args):       дополнительные параметры командной строки.
      minversion (string):  минимально необходимая версия pytest.
      required_plugins (args):
                            необходимые плагины, которые должны присутствовать для запуска pytest.

    переменные среды:
      PYTEST_ADDOPTS           дополнительные параметры командной строки.
      PYTEST_PLUGINS           плагины через запятую для загрузки во время запуска
      PYTEST_DISABLE_PLUGIN_AUTOLOAD установка, чтобы отключить автозагрузку плагина
      PYTEST_DEBUG             установка, чтобы включить отслеживание отладки внутренних компонентов pytest


    чтобы увидеть доступные типы маркеров: pytest --markers
    чтобы увидеть доступные типы фикстур: pytest --fixtures
    (отображается в соответствии с указанным file_or_dir или текущим каталогом, если не указано; фикстуры с впередистоящим '_' отображаются только с опцией '-v'
