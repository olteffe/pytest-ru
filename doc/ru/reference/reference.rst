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

**руководство**: :ref:`skipping`

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

**Tutorial**: :ref:`assertraises`.

.. autofunction:: pytest.raises(expected_exception: Exception [, *, match])
    :with: excinfo

pytest.deprecated_call
~~~~~~~~~~~~~~~~~~~~~~

**Tutorial**: :ref:`ensuring_function_triggers`.

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

**Tutorial**: :ref:`record_property example`.

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

**Tutorial**: :ref:`available installable plugins`

Can be declared at the **global** level in *test modules* and *conftest.py files* to register additional plugins.
Can be either a ``str`` or ``Sequence[str]``.

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



    Sets a :py:func:`time.strftime`-compatible string that will be used when formatting dates for logging capture.

    .. code-block:: ini

        [pytest]
        log_date_format = %Y-%m-%d %H:%M:%S

    For more information, see :ref:`logging`.


.. confval:: log_file



    Sets a file name relative to the ``pytest.ini`` file where log messages should be written to, in addition
    to the other logging facilities that are active.

    .. code-block:: ini

        [pytest]
        log_file = logs/pytest-logs.txt

    For more information, see :ref:`logging`.


.. confval:: log_file_date_format



    Sets a :py:func:`time.strftime`-compatible string that will be used when formatting dates for the logging file.

    .. code-block:: ini

        [pytest]
        log_file_date_format = %Y-%m-%d %H:%M:%S

    For more information, see :ref:`logging`.

.. confval:: log_file_format



    Sets a :py:mod:`logging`-compatible string used to format logging messages redirected to the logging file.

    .. code-block:: ini

        [pytest]
        log_file_format = %(asctime)s %(levelname)s %(message)s

    For more information, see :ref:`logging`.

.. confval:: log_file_level



    Sets the minimum log message level that should be captured for the logging file. The integer value or
    the names of the levels can be used.

    .. code-block:: ini

        [pytest]
        log_file_level = INFO

    For more information, see :ref:`logging`.


.. confval:: log_format



    Sets a :py:mod:`logging`-compatible string used to format captured logging messages.

    .. code-block:: ini

        [pytest]
        log_format = %(asctime)s %(levelname)s %(message)s

    For more information, see :ref:`logging`.


.. confval:: log_level



    Sets the minimum log message level that should be captured for logging capture. The integer value or
    the names of the levels can be used.

    .. code-block:: ini

        [pytest]
        log_level = INFO

    For more information, see :ref:`logging`.


.. confval:: markers

    When the ``--strict-markers`` or ``--strict`` command-line arguments are used,
    only known markers - defined in code by core pytest or some plugin - are allowed.

    You can list additional markers in this setting to add them to the whitelist,
    in which case you probably want to add ``--strict-markers`` to ``addopts``
    to avoid future regressions:

    .. code-block:: ini

        [pytest]
        addopts = --strict-markers
        markers =
            slow
            serial

    .. note::
        The use of ``--strict-markers`` is highly preferred. ``--strict`` was kept for
        backward compatibility only and may be confusing for others as it only applies to
        markers and not to other options.

.. confval:: minversion

   Specifies a minimal pytest version required for running tests.

   .. code-block:: ini

        # content of pytest.ini
        [pytest]
        minversion = 3.0  # will fail if we run with pytest-2.8


.. confval:: norecursedirs

   Set the directory basename patterns to avoid when recursing
   for test discovery.  The individual (fnmatch-style) patterns are
   applied to the basename of a directory to decide if to recurse into it.
   Pattern matching characters::

        *       matches everything
        ?       matches any single character
        [seq]   matches any character in seq
        [!seq]  matches any char not in seq

   Default patterns are ``'*.egg'``, ``'.*'``, ``'_darcs'``, ``'build'``,
   ``'CVS'``, ``'dist'``, ``'node_modules'``, ``'venv'``, ``'{arch}'``.
   Setting a ``norecursedirs`` replaces the default.  Here is an example of
   how to avoid certain directories:

   .. code-block:: ini

        [pytest]
        norecursedirs = .svn _build tmp*

   This would tell ``pytest`` to not look into typical subversion or
   sphinx-build directories or into any ``tmp`` prefixed directory.

   Additionally, ``pytest`` will attempt to intelligently identify and ignore a
   virtualenv by the presence of an activation script.  Any directory deemed to
   be the root of a virtual environment will not be considered during test
   collection unless ``‑‑collect‑in‑virtualenv`` is given.  Note also that
   ``norecursedirs`` takes precedence over ``‑‑collect‑in‑virtualenv``; e.g. if
   you intend to run tests in a virtualenv with a base directory that matches
   ``'.*'`` you *must* override ``norecursedirs`` in addition to using the
   ``‑‑collect‑in‑virtualenv`` flag.


.. confval:: python_classes

   One or more name prefixes or glob-style patterns determining which classes
   are considered for test collection. Search for multiple glob patterns by
   adding a space between patterns. By default, pytest will consider any
   class prefixed with ``Test`` as a test collection.  Here is an example of how
   to collect tests from classes that end in ``Suite``:

   .. code-block:: ini

        [pytest]
        python_classes = *Suite

   Note that ``unittest.TestCase`` derived classes are always collected
   regardless of this option, as ``unittest``'s own collection framework is used
   to collect those tests.


.. confval:: python_files

   One or more Glob-style file patterns determining which python files
   are considered as test modules. Search for multiple glob patterns by
   adding a space between patterns:

   .. code-block:: ini

        [pytest]
        python_files = test_*.py check_*.py example_*.py

   Or one per line:

   .. code-block:: ini

        [pytest]
        python_files =
            test_*.py
            check_*.py
            example_*.py

   By default, files matching ``test_*.py`` and ``*_test.py`` will be considered
   test modules.


.. confval:: python_functions

   One or more name prefixes or glob-patterns determining which test functions
   and methods are considered tests. Search for multiple glob patterns by
   adding a space between patterns. By default, pytest will consider any
   function prefixed with ``test`` as a test.  Here is an example of how
   to collect test functions and methods that end in ``_test``:

   .. code-block:: ini

        [pytest]
        python_functions = *_test

   Note that this has no effect on methods that live on a ``unittest
   .TestCase`` derived class, as ``unittest``'s own collection framework is used
   to collect those tests.

   See :ref:`change naming conventions` for more detailed examples.


.. confval:: required_plugins

   A space separated list of plugins that must be present for pytest to run.
   Plugins can be listed with or without version specifiers directly following
   their name. Whitespace between different version specifiers is not allowed.
   If any one of the plugins is not found, emit an error.

   .. code-block:: ini

       [pytest]
       required_plugins = pytest-django>=3.0.0,<4.0.0 pytest-html pytest-xdist>=1.0.0


.. confval:: testpaths



   Sets list of directories that should be searched for tests when
   no specific directories, files or test ids are given in the command line when
   executing pytest from the :ref:`rootdir <rootdir>` directory.
   Useful when all project tests are in a known location to speed up
   test collection and to avoid picking up undesired tests by accident.

   .. code-block:: ini

        [pytest]
        testpaths = testing doc

   This tells pytest to only look for tests in ``testing`` and ``doc``
   directories when executing from the root directory.


.. confval:: usefixtures

    List of fixtures that will be applied to all test functions; this is semantically the same to apply
    the ``@pytest.mark.usefixtures`` marker to all test functions.


    .. code-block:: ini

        [pytest]
        usefixtures =
            clean_db


.. confval:: xfail_strict

    If set to ``True``, tests marked with ``@pytest.mark.xfail`` that actually succeed will by default fail the
    test suite.
    For more information, see :ref:`xfail strict tutorial`.


    .. code-block:: ini

        [pytest]
        xfail_strict = True


.. _`command-line-flags`:

Command-line Flags
------------------

All the command-line flags can be obtained by running ``pytest --help``::

    $ pytest --help
    usage: pytest [options] [file_or_dir] [file_or_dir] [...]

    positional arguments:
      file_or_dir

    general:
      -k EXPRESSION         only run tests which match the given substring
                            expression. An expression is a python evaluatable
                            expression where all names are substring-matched
                            against test names and their parent classes.
                            Example: -k 'test_method or test_other' matches all
                            test functions and classes whose name contains
                            'test_method' or 'test_other', while -k 'not
                            test_method' matches those that don't contain
                            'test_method' in their names. -k 'not test_method
                            and not test_other' will eliminate the matches.
                            Additionally keywords are matched to classes and
                            functions containing extra names in their
                            'extra_keyword_matches' set, as well as functions
                            which have names assigned directly to them. The
                            matching is case-insensitive.
      -m MARKEXPR           only run tests matching given mark expression.
                            For example: -m 'mark1 and not mark2'.
      --markers             show markers (builtin, plugin and per-project ones).
      -x, --exitfirst       exit instantly on first error or failed test.
      --fixtures, --funcargs
                            show available fixtures, sorted by plugin appearance
                            (fixtures with leading '_' are only shown with '-v')
      --fixtures-per-test   show fixtures per test
      --pdb                 start the interactive Python debugger on errors or
                            KeyboardInterrupt.
      --pdbcls=modulename:classname
                            start a custom interactive Python debugger on
                            errors. For example:
                            --pdbcls=IPython.terminal.debugger:TerminalPdb
      --trace               Immediately break when running each test.
      --capture=method      per-test capturing method: one of fd|sys|no|tee-sys.
      -s                    shortcut for --capture=no.
      --runxfail            report the results of xfail tests as if they were
                            not marked
      --lf, --last-failed   rerun only the tests that failed at the last run (or
                            all if none failed)
      --ff, --failed-first  run all tests, but run the last failures first.
                            This may re-order tests and thus lead to repeated
                            fixture setup/teardown.
      --nf, --new-first     run tests from new files first, then the rest of the
                            tests sorted by file mtime
      --cache-show=[CACHESHOW]
                            show cache contents, don't perform collection or
                            tests. Optional argument: glob (default: '*').
      --cache-clear         remove all cache contents at start of test run.
      --lfnf={all,none}, --last-failed-no-failures={all,none}
                            which tests to run with no previously (known)
                            failures.
      --sw, --stepwise      exit on test failure and continue from last failing
                            test next time
      --sw-skip, --stepwise-skip
                            ignore the first failing test but stop on the next
                            failing test

    reporting:
      --durations=N         show N slowest setup/test durations (N=0 for all).
      --durations-min=N     Minimal duration in seconds for inclusion in slowest
                            list. Default 0.005
      -v, --verbose         increase verbosity.
      --no-header           disable header
      --no-summary          disable summary
      -q, --quiet           decrease verbosity.
      --verbosity=VERBOSE   set verbosity. Default is 0.
      -r chars              show extra test summary info as specified by chars:
                            (f)ailed, (E)rror, (s)kipped, (x)failed, (X)passed,
                            (p)assed, (P)assed with output, (a)ll except passed
                            (p/P), or (A)ll. (w)arnings are enabled by default
                            (see --disable-warnings), 'N' can be used to reset
                            the list. (default: 'fE').
      --disable-warnings, --disable-pytest-warnings
                            disable warnings summary
      -l, --showlocals      show locals in tracebacks (disabled by default).
      --tb=style            traceback print mode
                            (auto/long/short/line/native/no).
      --show-capture={no,stdout,stderr,log,all}
                            Controls how captured stdout/stderr/log is shown on
                            failed tests. Default is 'all'.
      --full-trace          don't cut any tracebacks (default is to cut).
      --color=color         color terminal output (yes/no/auto).
      --code-highlight={yes,no}
                            Whether code should be highlighted (only if --color
                            is also enabled)
      --pastebin=mode       send failed|all info to bpaste.net pastebin service.
      --junit-xml=path      create junit-xml style report file at given path.
      --junit-prefix=str    prepend prefix to classnames in junit-xml output

    pytest-warnings:
      -W PYTHONWARNINGS, --pythonwarnings=PYTHONWARNINGS
                            set which warnings to report, see -W option of
                            python itself.
      --maxfail=num         exit after first num failures or errors.
      --strict-config       any warnings encountered while parsing the `pytest`
                            section of the configuration file raise errors.
      --strict-markers      markers not registered in the `markers` section of
                            the configuration file raise errors.
      --strict              (deprecated) alias to --strict-markers.
      -c file               load configuration from `file` instead of trying to
                            locate one of the implicit configuration files.
      --continue-on-collection-errors
                            Force test execution even if collection errors
                            occur.
      --rootdir=ROOTDIR     Define root directory for tests. Can be relative
                            path: 'root_dir', './root_dir',
                            'root_dir/another_dir/'; absolute path:
                            '/home/user/root_dir'; path with variables:
                            '$HOME/root_dir'.

    collection:
      --collect-only, --co  only collect tests, don't execute them.
      --pyargs              try to interpret all arguments as python packages.
      --ignore=path         ignore path during collection (multi-allowed).
      --ignore-glob=path    ignore path pattern during collection (multi-
                            allowed).
      --deselect=nodeid_prefix
                            deselect item (via node id prefix) during collection
                            (multi-allowed).
      --confcutdir=dir      only load conftest.py's relative to specified dir.
      --noconftest          Don't load any conftest.py files.
      --keep-duplicates     Keep duplicate tests.
      --collect-in-virtualenv
                            Don't ignore tests in a local virtualenv directory
      --import-mode={prepend,append,importlib}
                            prepend/append to sys.path when importing test
                            modules and conftest files, default is to prepend.
      --doctest-modules     run doctests in all .py modules
      --doctest-report={none,cdiff,ndiff,udiff,only_first_failure}
                            choose another output format for diffs on doctest
                            failure
      --doctest-glob=pat    doctests file matching pattern, default: test*.txt
      --doctest-ignore-import-errors
                            ignore doctest ImportErrors
      --doctest-continue-on-failure
                            for a given doctest, continue to run after the first
                            failure

    test session debugging and configuration:
      --basetemp=dir        base temporary directory for this test run.(warning:
                            this directory is removed if it exists)
      -V, --version         display pytest version and information about
                            plugins.When given twice, also display information
                            about plugins.
      -h, --help            show help message and configuration info
      -p name               early-load given plugin module name or entry point
                            (multi-allowed).
                            To avoid loading of plugins, use the `no:` prefix,
                            e.g. `no:doctest`.
      --trace-config        trace considerations of conftest.py files.
      --debug               store internal tracing debug information in
                            'pytestdebug.log'.
      -o OVERRIDE_INI, --override-ini=OVERRIDE_INI
                            override ini option with "option=value" style, e.g.
                            `-o xfail_strict=True -o cache_dir=cache`.
      --assert=MODE         Control assertion debugging tools.
                            'plain' performs no assertion debugging.
                            'rewrite' (the default) rewrites assert statements
                            in test modules on import to provide assert
                            expression information.
      --setup-only          only setup fixtures, do not execute tests.
      --setup-show          show setup of fixtures while executing tests.
      --setup-plan          show what fixtures and tests would be executed but
                            don't execute anything.

    logging:
      --log-level=LEVEL     level of messages to catch/display.
                            Not set by default, so it depends on the root/parent
                            log handler's effective level, where it is "WARNING"
                            by default.
      --log-format=LOG_FORMAT
                            log format as used by the logging module.
      --log-date-format=LOG_DATE_FORMAT
                            log date format as used by the logging module.
      --log-cli-level=LOG_CLI_LEVEL
                            cli logging level.
      --log-cli-format=LOG_CLI_FORMAT
                            log format as used by the logging module.
      --log-cli-date-format=LOG_CLI_DATE_FORMAT
                            log date format as used by the logging module.
      --log-file=LOG_FILE   path to a file when logging will be written to.
      --log-file-level=LOG_FILE_LEVEL
                            log file logging level.
      --log-file-format=LOG_FILE_FORMAT
                            log format as used by the logging module.
      --log-file-date-format=LOG_FILE_DATE_FORMAT
                            log date format as used by the logging module.
      --log-auto-indent=LOG_AUTO_INDENT
                            Auto-indent multiline messages passed to the logging
                            module. Accepts true|on, false|off or an integer.

    [pytest] ini-options in the first pytest.ini|tox.ini|setup.cfg file found:

      markers (linelist):   markers for test functions
      empty_parameter_set_mark (string):
                            default marker for empty parametersets
      norecursedirs (args): directory patterns to avoid for recursion
      testpaths (args):     directories to search for tests when no files or
                            directories are given in the command line.
      filterwarnings (linelist):
                            Each line specifies a pattern for
                            warnings.filterwarnings. Processed after
                            -W/--pythonwarnings.
      usefixtures (args):   list of default fixtures to be used with this
                            project
      python_files (args):  glob-style file patterns for Python test module
                            discovery
      python_classes (args):
                            prefixes or glob names for Python test class
                            discovery
      python_functions (args):
                            prefixes or glob names for Python test function and
                            method discovery
      disable_test_id_escaping_and_forfeit_all_rights_to_community_support (bool):
                            disable string escape non-ascii characters, might
                            cause unwanted side effects(use at your own risk)
      console_output_style (string):
                            console output: "classic", or with additional
                            progress information ("progress" (percentage) |
                            "count").
      xfail_strict (bool):  default for the strict parameter of xfail markers
                            when not given explicitly (default: False)
      enable_assertion_pass_hook (bool):
                            Enables the pytest_assertion_pass hook.Make sure to
                            delete any previously generated pyc cache files.
      junit_suite_name (string):
                            Test suite name for JUnit report
      junit_logging (string):
                            Write captured log messages to JUnit report: one of
                            no|log|system-out|system-err|out-err|all
      junit_log_passing_tests (bool):
                            Capture log information for passing tests to JUnit
                            report:
      junit_duration_report (string):
                            Duration time to report: one of total|call
      junit_family (string):
                            Emit XML for schema: one of legacy|xunit1|xunit2
      doctest_optionflags (args):
                            option flags for doctests
      doctest_encoding (string):
                            encoding used for doctest files
      cache_dir (string):   cache directory path.
      log_level (string):   default value for --log-level
      log_format (string):  default value for --log-format
      log_date_format (string):
                            default value for --log-date-format
      log_cli (bool):       enable log display during test run (also known as
                            "live logging").
      log_cli_level (string):
                            default value for --log-cli-level
      log_cli_format (string):
                            default value for --log-cli-format
      log_cli_date_format (string):
                            default value for --log-cli-date-format
      log_file (string):    default value for --log-file
      log_file_level (string):
                            default value for --log-file-level
      log_file_format (string):
                            default value for --log-file-format
      log_file_date_format (string):
                            default value for --log-file-date-format
      log_auto_indent (string):
                            default value for --log-auto-indent
      faulthandler_timeout (string):
                            Dump the traceback of all threads if a test takes
                            more than TIMEOUT seconds to finish.
      addopts (args):       extra command line options
      minversion (string):  minimally required pytest version
      required_plugins (args):
                            plugins that must be present for pytest to run

    environment variables:
      PYTEST_ADDOPTS           extra command line options
      PYTEST_PLUGINS           comma-separated plugins to load during startup
      PYTEST_DISABLE_PLUGIN_AUTOLOAD set to disable plugin auto-loading
      PYTEST_DEBUG             set to enable debug tracing of pytest's internals


    to see available markers type: pytest --markers
    to see available fixtures type: pytest --fixtures
    (shown according to specified file_or_dir or current dir if not specified; fixtures with leading '_' are only shown with the '-v' option
