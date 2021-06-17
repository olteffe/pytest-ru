:orphan:

===============================================
ПРЕДЛОЖЕНИЕ: Параметризация с помощью фикстур
===============================================

.. warning::

    Этот документ описывает предложение по использованию фикстур в качестве входных данных
    параметризованных тестов или фикстур.

Проблемы
---------

Как пользователь, у меня есть функциональные тесты, которые я хотел бы провести в
различных сценариях.

В этом конкретном примере мы хотим создать новый проект на основе шаблона cookiecutter.
Мы хотим протестировать значения по умолчанию, а также данные, имитирующие ввод данных
пользователем.

- использование значения по умолчанию

- имитировать ввод пользователя

  - указываем 'author'

  - указываем 'project_slug'

  - указываем 'author' и 'project_slug'

Вот так может выглядеть функциональный тест:

.. code-block:: python

    import pytest


    @pytest.fixture
    def default_context():
        return {"extra_context": {}}


    @pytest.fixture(
        params=[
            {"author": "alice"},
            {"project_slug": "helloworld"},
            {"author": "bob", "project_slug": "foobar"},
        ]
    )
    def extra_context(request):
        return {"extra_context": request.param}


    @pytest.fixture(params=["default", "extra"])
    def context(request):
        if request.param == "default":
            return request.getfuncargvalue("default_context")
        else:
            return request.getfuncargvalue("extra_context")


    def test_generate_project(cookies, context):
        """Вызываем cookiecutter API для создания нового проекта из шаблона."""
        result = cookies.bake(extra_context=context)

        assert result.exit_code == 0
        assert result.exception is None
        assert result.project.isdir()


Проблемы
---------

* Используя ``request.getfuncargvalue()`` мы полагаемся на фактическое выполнение функции
  фикстуры, чтобы знать, какие фикстуры задействованы, из-за ее динамической сущности.
* Что еще более важно, ``request.getfuncargvalue()`` не может быть объединенным с параметризованной фикстурой,
  такой как ``extra_context``.
* Это очень неудобно, если вы хотите расширить существующий набор тестов с помощью
  определенных параметров для фикстуры, которые уже используются тестами.

pytest версии 3.0 сообщает об ошибке, если вы попытаетесь запустить вышеуказанный код::

    Failed: The requested fixture has no parameter defined for the current
    test.

    Requested fixture 'extra_context'


Предложенное решение
--------------------

Новая функция, которую можно использовать в модулях, может использоваться для динамического определения
фикстур из существующих.

.. code-block:: python

    pytest.define_combined_fixture(
        name="context", fixtures=["default_context", "extra_context"]
    )

Новая фикстура ``context`` наследует область видимости от используемых фикстур и дает
следующие значения.

- ``{}``

- ``{'author': 'alice'}``

- ``{'project_slug': 'helloworld'}``

- ``{'author': 'bob', 'project_slug': 'foobar'}``

Альтернативный подход
-----------------------

Новая вспомогательная функция с именем ``fixture_request`` будет сообщать pytest, что нужно выдать
все параметры, помеченные как фикстура.

.. note::

    `pytest-lazy-fixture <https://pypi.org/project/pytest-lazy-fixture/>`_ плагин реализует решение,
    очень похожее на предложение ниже, обязательно ознакомьтесь с ним.

.. code-block:: python

    @pytest.fixture(
        params=[
            pytest.fixture_request("default_context"),
            pytest.fixture_request("extra_context"),
        ]
    )
    def context(request):
        """Возвращает все значения для ``default_context``, одно за другим, перед тем как
        делает то же самое для ``extra_context``.

        request.param:
            - {}
            - {'author': 'alice'}
            - {'project_slug': 'helloworld'}
            - {'author': 'bob', 'project_slug': 'foobar'}
        """
        return request.param

Эта же вспомогательная функция может быть использован в сочетании с ``pytest.mark.parametrize``.

.. code-block:: python


    @pytest.mark.parametrize(
        "context, expected_response_code",
        [
            (pytest.fixture_request("default_context"), 0),
            (pytest.fixture_request("extra_context"), 0),
        ],
    )
    def test_generate_project(cookies, context, exit_code):
        """Вызов cookiecutter API для генерации нового проекта из шаблона.
        """
        result = cookies.bake(extra_context=context)

        assert result.exit_code == exit_code
