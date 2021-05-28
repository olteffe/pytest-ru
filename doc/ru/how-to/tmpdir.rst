
.. _`tmpdir handling`:
.. _tmpdir:

Использование временных каталогов и файлов в тестах
=====================================================

Фикстура ``tmp_path``
------------------------

Вы можете использовать фикстуру ``tmp_path``, которая предоставит временный каталог, уникальный для
вызова теста, созданный в `base temporary directory`_.

``tmp_path`` это объект :class:`pathlib.Path`. Вот пример использования теста:

.. code-block:: python

    # cлистинг test_tmp_path.py
    CONTENT = "content"


    def test_create_file(tmp_path):
        d = tmp_path / "sub"
        d.mkdir()
        p = d / "hello.txt"
        p.write_text(CONTENT)
        assert p.read_text() == CONTENT
        assert len(list(tmp_path.iterdir())) == 1
        assert 0

Запуск приведет к пройденному тесту, за исключением последней строки ``assert 0``, которую мы
используем для просмотра значений:

.. code-block:: pytest

    $ pytest test_tmp_path.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 1 item

    test_tmp_path.py F                                                   [100%]

    ================================= FAILURES =================================
    _____________________________ test_create_file _____________________________

    tmp_path = PosixPath('PYTEST_TMPDIR/test_create_file0')

        def test_create_file(tmp_path):
            d = tmp_path / "sub"
            d.mkdir()
            p = d / "hello.txt"
            p.write_text(CONTENT)
            assert p.read_text() == CONTENT
            assert len(list(tmp_path.iterdir())) == 1
    >       assert 0
    E       assert 0

    test_tmp_path.py:11: AssertionError
    ========================= short test summary info ==========================
    FAILED test_tmp_path.py::test_create_file - assert 0
    ============================ 1 failed in 0.12s =============================

.. _`tmp_path_factory example`:

Фикстура ``tmp_path_factory``
--------------------------------

``tmp_path_factory`` - это фикстура с ограниченным сеансом, которую можно использовать для создания
произвольных временных каталогов из любой другой фикстуры или теста.

Например, предположим, что вашему набору тестов требуется большой образ на диске, который создается
процедурно. Вместо того, чтобы вычислять один и тот же образ для каждого теста, который использует
его в своем собственном ``tmp_path``, вы можете сгенерировать его один раз за сеанс, чтобы сэкономить
время:

.. code-block:: python

    # листинг conftest.py
    import pytest


    @pytest.fixture(scope="session")
    def image_file(tmp_path_factory):
        img = compute_expensive_image()
        fn = tmp_path_factory.mktemp("data") / "img.png"
        img.save(fn)
        return fn


    # листинг test_image.py
    def test_histogram(image_file):
        img = load_image(image_file)
        # вычисление и проверка гистограммы

См. :ref:`tmp_path_factory API <tmp_path_factory factory api>` для подробностей.

.. _`tmpdir and tmpdir_factory`:

Фикстуры ``tmpdir`` и ``tmpdir_factory``
---------------------------------------------------

Фикстуры ``tmpdir`` и ``tmpdir_factory`` похожи на ``tmp_path``
и ``tmp_path_factory``, но используют/возвращают устаревшие объекты `py.path.local`_
а не стандартные объекты :class:`pathlib.Path`. В наши дни предпочитают использовать ``tmp_path`` и
``tmp_path_factory``.

См. :fixture:`tmpdir <tmpdir>` :fixture:`tmpdir_factory <tmpdir_factory>` API для подробностей.


.. _`base temporary directory`:

Базовый временный каталог по умолчанию
-----------------------------------------------

Временные каталоги по умолчанию создаются как подкаталоги временного системного каталога. Базовое имя
будет ``pytest-NUM``, где ``NUM`` будет увеличиваться с каждым запуском теста. Кроме того, будут
удалены записи старше трех временных каталогов.

Вы можете переопределить настройку временного каталога по умолчанию следующим образом:

.. code-block:: bash

    pytest --basetemp=mydir

.. warning::

    Содержание ``mydir`` будет полностью удален, поэтому обязательно используйте каталог только для
    этой цели.

При распространении тестов на локальном компьютере с использованием ``pytest-xdist`` необходимо
автоматически настроить каталог basetemp для подпроцессов, чтобы все временные данные располагались
ниже одного каталога basetemp для каждого запуска теста.

.. _`py.path.local`: https://py.readthedocs.io/en/latest/path.html
