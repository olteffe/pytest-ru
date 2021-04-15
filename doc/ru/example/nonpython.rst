
.. _`non-python tests`:

Работа с тестами не на ``python``
====================================================

.. _`yaml plugin`:

Простой пример указания тестов в файлах Yaml
--------------------------------------------------------------

.. _`pytest-yamlwsgi`: http://bitbucket.org/aafshar/pytest-yamlwsgi/src/tip/pytest_yamlwsgi.py
.. _`PyYAML`: https://pypi.org/project/PyYAML/

Вот пример файла ``conftest.py`` (полученного из плагина Ali Afshar-а `pytest-yamlwsgi`_ ).
Этот ``conftest.py`` собирает файлы вида ``test*.yaml`` и выполнит содержимое в
формате yaml как пользовательские тесты:


.. include:: nonpython/conftest.py
    :literal:

Вы можете создать простой пример файла:

.. include:: nonpython/test_simple.yaml
    :literal:

и если вы установите `PyYAML`_ или совместимый YAML-парсер, то сможете
запустить тесты:

.. code-block:: pytest

    nonpython $ pytest test_simple.yaml
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR/nonpython
    collected 2 items

    test_simple.yaml F.                                                  [100%]

    ================================= FAILURES =================================
    ______________________________ usecase: hello ______________________________
    usecase execution failed
       spec failed: 'some': 'other'
       no further details known at this point.
    ========================= short test summary info ==========================
    FAILED test_simple.yaml::hello
    ======================= 1 failed, 1 passed in 0.12s ========================

.. regendoc:wipe

У нас получился один прошедший тест на проверку ``sub1: sub1`` и один упавший.
Очевидно, что в приведенном выше файле ``conftest.py`` вы захотите реализовать более
интересную интерпретацию значений yaml.  Таким образом вы можете легко написать свой
собственный "язык тестирования" для конкретной предметной области.

.. note::

    ``repr_failure(excinfo)`` вызывается для представления упавших тестов.
    Если вы сами будете генерировать узлы, то сможете возвращать
    любое строковое представление ошибки по вашему выбору. В отчете
    оно будет выделено красным.

``reportinfo()`` используется для представления расположения теста, а также
в режиме ``verbose``:

.. code-block:: pytest

    nonpython $ pytest -v
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR/nonpython
    collecting ... collected 2 items

    test_simple.yaml::hello FAILED                                       [ 50%]
    test_simple.yaml::ok PASSED                                          [100%]

    ================================= FAILURES =================================
    ______________________________ usecase: hello ______________________________
    usecase execution failed
       spec failed: 'some': 'other'
       no further details known at this point.
    ========================= short test summary info ==========================
    FAILED test_simple.yaml::hello
    ======================= 1 failed, 1 passed in 0.12s ========================

.. regendoc:wipe

При разработке собственного поиска и выполнения тестов интересно
будет взглянуть на дерево собранных тестов:

.. code-block:: pytest

    nonpython $ pytest --collect-only
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR/nonpython
    collected 2 items

    <Package nonpython>
      <YamlFile test_simple.yaml>
        <YamlItem hello>
        <YamlItem ok>

    ======================== 2 tests collected in 0.12s ========================
