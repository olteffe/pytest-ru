.. _`warnings`:

Фиксирование предупреждений
===============================

Начиная с версии ``3.1``, pytest автоматически перехватывает предупреждения во время
выполнения теста и отображает их в конце сеанса:

.. code-block:: python

    # листинг test_show_warnings.py
    import warnings


    def api_v1():
        warnings.warn(UserWarning("api v1, should use functions from v2"))
        return 1


    def test_one():
        assert api_v1() == 1

При запуске pytest получим следующий результат:

.. code-block:: pytest

    $ pytest test_show_warnings.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_show_warnings.py .                                              [100%]

    ============================= warnings summary =============================
    test_show_warnings.py::test_one
      $REGENDOC_TMPDIR/test_show_warnings.py:5: UserWarning: api v1, should use functions from v2
        warnings.warn(UserWarning("api v1, should use functions from v2"))

    -- Docs: https://docs.pytest.org/en/stable/warnings.html
    ======================= 1 passed, 1 warning in 0.12s =======================

Флаг ``-W`` может быть передан для управления тем, какие предупреждения будут отображаться
или даже обратить их в ошибки:

.. code-block:: pytest

    $ pytest -q test_show_warnings.py -W error::UserWarning
    F                                                                    [100%]
    ================================= FAILURES =================================
    _________________________________ test_one _________________________________

        def test_one():
    >       assert api_v1() == 1

    test_show_warnings.py:10:
    _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

        def api_v1():
    >       warnings.warn(UserWarning("api v1, should use functions from v2"))
    E       UserWarning: api v1, should use functions from v2

    test_show_warnings.py:5: UserWarning
    ========================= short test summary info ==========================
    FAILED test_show_warnings.py::test_one - UserWarning: api v1, should use ...
    1 failed in 0.12s

Тот же параметр можно установить в файле ``pytest.ini`` или ``pyproject.toml`` с помощью
ini-параметра ``filterwarnings``. Например, приведенные ниже настройки игнорируют все
пользовательские предупреждения и определенные предупреждения об устаревании,
соответствующие регулярному выражению, но преобразует все другие предупреждения в ошибки.

.. code-block:: ini

    # pytest.ini
    [pytest]
    filterwarnings =
        error
        ignore::UserWarning
        ignore:function ham\(\) is deprecated:DeprecationWarning

.. code-block:: toml

    # pyproject.toml
    [tool.pytest.ini_options]
    filterwarnings = [
        "error",
        "ignore::UserWarning",
        # note the use of single quote below to denote "raw" strings in TOML
        'ignore:function ham\(\) is deprecated:DeprecationWarning',
    ]


Когда предупреждение соответствует более чем одному параметру в списке, выполняется
действие для последнего совпадающего параметра.

Опция командной строки ``-W`` и настройка ``filterwarnings`` в ini-файле основаны на встроенные в Python
`-W option`_ и `warnings.simplefilter`_, поэтому обратитесь к этим разделам в документации Python для
других примеров и использования.

.. _`filterwarnings`:

``@pytest.mark.filterwarnings``
-------------------------------



Вы можете использовать ``@pytest.mark.filterwarnings`` для добавления фильтров предупреждений к конкретным
элементам теста, что позволяет более точно контролировать, какие предупреждения следует фиксировать на
уровне теста, класса или даже модуля:

.. code-block:: python

    import warnings


    def api_v1():
        warnings.warn(UserWarning("api v1, should use functions from v2"))
        return 1


    @pytest.mark.filterwarnings("ignore:api v1")
    def test_one():
        assert api_v1() == 1


Фильтры, примененные с помощью метки, имеют приоритет над фильтрами, переданными в командной строке или
настроенными с помощью ini-параметра ``filterwarnings``.

Вы можете применить фильтр ко всем тестам класса, используя метку ``filterwarnings`` в качестве декоратора
класса, или ко всем тестам в модуле, установив переменную :globalvar:`pytestmark`:

.. code-block:: python

    # превращает все предупреждения в ошибки для этого модуля
    pytestmark = pytest.mark.filterwarnings("error")



*Смотрите эталонную реализацию Florian Schulze в модуле* `pytest-warnings`_

.. _`-W option`: https://docs.python.org/3/using/cmdline.html#cmdoption-w
.. _warnings.simplefilter: https://docs.python.org/3/library/warnings.html#warnings.simplefilter
.. _`pytest-warnings`: https://github.com/fschulze/pytest-warnings

Отключение сводки предупреждений
----------------------------------

Хотя это не рекомендуется, вы можете использовать параметр командной строки ``--disable-warnings``, чтобы
полностью исключить суммарные предупреждения из выходных данных тестового запуска.

Полное отключение сбора предупреждений
---------------------------------------

Этот плагин включен по умолчанию, но его можно полностью отключить в файле ``pytest.ini`` с помощью:

    .. code-block:: ini

        [pytest]
        addopts = -p no:warnings

Или передав в командной строке ``-p no:warnings``. Это может быть полезно, если ваши наборы тестов
обрабатывают предупреждения с помощью внешней системы.


.. _`deprecation-warnings`:

DeprecationWarning and PendingDeprecationWarning
------------------------------------------------




По умолчанию pytest отобразит предупреждения ``DeprecationWarning`` и ``PendingDeprecationWarning``  из
пользовательского кода и сторонних библиотек, как рекомендуется в `PEP-0565 <https://www.python.org/dev/peps/pep-0565>`_.
Это помогает пользователям поддерживать свой код в современном стиле и избегать поломок, когда
DeprecationWarning эффективно удаляются.

Иногда полезно скрыть некоторые конкретные DeprecationWarning, которые появляются в коде, который вы не
контролируете (например, сторонние библиотеки), и в этом случае вы можете использовать
параметры фильтров предупреждений (ini или метки), чтобы игнорировать эти предупреждения.

Например:

.. code-block:: ini

    [pytest]
    filterwarnings =
        ignore:.*U.*mode is deprecated:DeprecationWarning


Это проигнорирует все предупреждения типа ``DeprecationWarning``, где начало сообщения соответствует регулярному
выражению ``".*U.*mode is deprecated"``.

.. note::

    Если предупреждения настроены на уровне интерпретатора, использование
    `PYTHONWARNINGS <https://docs.python.org/3/using/cmdline.html#envvar-PYTHONWARNINGS>`_ переменная
    окружения или параметр командной строки ``-W``, pytest не будет настраивать какие-либо фильтры по
    умолчанию.

    Также pytest не следует ``PEP-0506`` о сбросе всех фильтров предупреждений, потому что
    это может нарушить комплекты тестов, которые сами настраивают фильтры предупреждений
    вызывая ``warnings.simplefilter`` (см. issue `#2430 <https://github.com/pytest-dev/pytest/issues/2430>`_
    для примера).


.. _`ensuring a function triggers a deprecation warning`:

.. _ensuring_function_triggers:

Обеспечение того, чтобы код запускал предупреждение об устаревании
--------------------------------------------------------------------

Вы также можете использовать :func:`pytest.deprecated_call` для проверки того, что
вызов определенной функции запускает ``DeprecationWarning`` или ``PendingDeprecationWarning``:

.. code-block:: python

    import pytest


    def test_myfunction_deprecated():
        with pytest.deprecated_call():
            myfunction(17)

Этот тест не пройдёт, если ``myfunction`` не выдает предупреждение об устаревании при вызове с
аргументом ``17``.

По умолчанию, ``DeprecationWarning`` и ``PendingDeprecationWarning`` не будет захвачен при использовании
:func:`pytest.warns` или :ref:`recwarn <recwarn>`, потому что фильтр предупреждений Python по умолчанию
скрывает их. Если вы хотите записать их в свой код, используйте ``warnings.simplefilter('always')``:

.. code-block:: python

    import warnings
    import pytest


    def test_deprecation(recwarn):
        warnings.simplefilter("always")
        myfunction(17)
        assert len(recwarn) == 1
        assert recwarn.pop(DeprecationWarning)


Фикстура :ref:`recwarn <recwarn>` автоматически обеспечивает сброс фильтра предупреждений в конце теста,
так что не происходит утечки глобального состояния.

.. _`asserting warnings`:

.. _assertwarnings:

.. _`asserting warnings with the warns function`:

.. _warns:

Утверждение предупреждений с помощью функции предупреждений
-------------------------------------------------------------



Вы можете проверить, вызывает ли код конкретное предупреждение, используя func:`pytest.warns`,
который работает аналогично :ref:`raises <assertraises>`:

.. code-block:: python

    import warnings
    import pytest


    def test_warning():
        with pytest.warns(UserWarning):
            warnings.warn("my warning", UserWarning)

Тест упадет, если соответствующее предупреждение не появится. Аргумент ``match``,
чтобы подтвердить, что исключение соответствует тексту или регулярному выражению::

    >>> with warns(UserWarning, match='must be 0 or None'):
    ...     warnings.warn("value must be 0 or None", UserWarning)

    >>> with warns(UserWarning, match=r'must be \d+$'):
    ...     warnings.warn("value must be 42", UserWarning)

    >>> with warns(UserWarning, match=r'must be \d+$'):
    ...     warnings.warn("this is not here", UserWarning)
    Traceback (most recent call last):
      ...
    Failed: DID NOT WARN. No warnings of type ...UserWarning... was emitted...

Вы также можете вызвать func:`pytest.warns` в строке функции или кода:

.. code-block:: python

    pytest.warns(expected_warning, func, *args, **kwargs)
    pytest.warns(expected_warning, "func(*args, **kwargs)")

Функция также возвращает список всех выданных предупреждений (такие как объекты
``warnings.WarningMessage``), которые вы можете запросить для получения дополнительной информации:

.. code-block:: python

    with pytest.warns(RuntimeWarning) as record:
        warnings.warn("another warning", RuntimeWarning)

    # проверка, что было только одно предупреждение
    assert len(record) == 1
    # проверка, что сообщение соответствует
    assert record[0].message.args[0] == "another warning"

Кроме того, вы можете подробно изучить возникшие предупреждения, используя фикстуру
:ref:`recwarn <recwarn>`(см. ниже).

.. note::
    ``DeprecationWarning`` и ``PendingDeprecationWarning`` обрабатываются по разному;
    см. :ref:`ensuring_function_triggers`.

.. _`recording warnings`:

.. _recwarn:

Регистрирующие предупреждения
------------------------------

Вы можете записывать возникшие предупреждения, используя func:`pytest.warns` или с фикстурой ``recwarn``.

Для записи с помощью func: `pytest.warns` без утверждения каких-либо предупреждений, передайте
``None`` в качестве ожидаемого типа предупреждения:

.. code-block:: python

    with pytest.warns(None) as record:
        warnings.warn("user", UserWarning)
        warnings.warn("runtime", RuntimeWarning)

    assert len(record) == 2
    assert str(record[0].message) == "user"
    assert str(record[1].message) == "runtime"

Фикстура ``recwarn`` будет записывать предупреждения для всей функции:

.. code-block:: python

    import warnings


    def test_hello(recwarn):
        warnings.warn("hello", UserWarning)
        assert len(recwarn) == 1
        w = recwarn.pop(UserWarning)
        assert issubclass(w.category, UserWarning)
        assert str(w.message) == "hello"
        assert w.filename
        assert w.lineno

И ``recwarn``, и func:`pytest.warns` возвращают один и тот же интерфейс для записанных предупреждений:
экземпляр WarningsRecorder. Чтобы просмотреть записанные предупреждения, вы можете перебрать этот
экземпляр, вызвать для него ``len``, чтобы получить количество записанных предупреждений, или
проиндексировать его, чтобы получить конкретное записанное предупреждение.

.. currentmodule:: _pytest.warnings

Full API: :class:`~_pytest.recwarn.WarningsRecorder`.

.. _custom_failure_messages:

Пользовательские сообщения об ошибках
--------------------------------------

Запись предупреждений дает возможность создавать настраиваемые сообщения об ошибках тестирования,
когда предупреждения не выдаются или выполняются другие условия.

.. code-block:: python

    def test():
        with pytest.warns(Warning) as record:
            f()
            if not record:
                pytest.fail("Expected a warning!")

Если при вызове ``f`` не выдаются предупреждения, то ``not record`` будет оцениваться как ``True``.
Затем вы можете вызвать :func:`pytest.fail` с настраиваемым сообщением об ошибке.

.. _internal-warnings:

Внутренние предупреждения pytest
------------------------------------

pytest может генерировать собственные предупреждения в некоторых ситуациях, таких как неправильное
использование или устаревшие функции.

Например, pytest выдаст предупреждение, если он встретит класс, который соответствует
:confval:`python_classes`, но также определяет конструктор ``__init__``, поскольку это предотвращает
создание экземпляра класса:

.. code-block:: python

    # листинг test_pytest_warnings.py
    class Test:
        def __init__(self):
            pass

        def test_foo(self):
            assert 1 == 1

.. code-block:: pytest

    $ pytest test_pytest_warnings.py -q

    ============================= warnings summary =============================
    test_pytest_warnings.py:1
      $REGENDOC_TMPDIR/test_pytest_warnings.py:1: PytestCollectionWarning: cannot collect test class 'Test' because it has a __init__ constructor (from: test_pytest_warnings.py)
        class Test:

    -- Docs: https://docs.pytest.org/en/stable/warnings.html
    1 warning in 0.12s

Эти предупреждения могут быть отфильтрованы с использованием тех же встроенных механизмов, которые
используются для фильтрации других типов предупреждений.

Пожалуйста, прочтите наш :ref:`backwards-compatibility` чтобы узнать, как мы поступаем с прекращением
поддержки и, в конечном итоге, удалением функций.

Полный список предупреждений приведен в :ref:`the reference documentation <warnings ref>`.
