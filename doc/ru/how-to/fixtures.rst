.. _how-to-fixtures:

Как использовать фикстуры
==========================

.. seealso:: :ref:`about-fixtures`
.. seealso:: :ref:`Fixtures reference <reference-fixtures>`


"Запрашивающие" фикстуры
-------------------------

На базовом уровне тестовые функции запрашивают необходимые им фикстуры, объявляя их в качестве аргументов.

Когда pytest запускает тест, он просматривает параметры в сигнатуре этой тестовой функции, а затем ищет
фикстуры, имена которых совпадают с именами этих параметров. Как только pytest находит их, он запускает
эти фикстуры, фиксирует то, что они вернули (если есть), и передает эти объекты в тестовую функцию в
качестве аргументов.

Пример
^^^^^^^^^^^^^

.. code-block:: python

    import pytest


    class Fruit:
        def __init__(self, name):
            self.name = name
            self.cubed = False

        def cube(self):
            self.cubed = True


    class FruitSalad:
        def __init__(self, *fruit_bowl):
            self.fruit = fruit_bowl
            self._cube_fruit()

        def _cube_fruit(self):
            for fruit in self.fruit:
                fruit.cube()


    # Настройка
    @pytest.fixture
    def fruit_bowl():
        return [Fruit("apple"), Fruit("banana")]


    def test_fruit_salad(fruit_bowl):
        # Выполнение
        fruit_salad = FruitSalad(*fruit_bowl)

        # Проверка
        assert all(fruit.cubed for fruit in fruit_salad.fruit)

В этом примере, ``test_fruit_salad`` "**запрашивает**" ``fruit_bowl`` (т.е.
``def test_fruit_salad(fruit_bowl):``), и когда pytest увидит это, он выполнит фикстурную функцию
 ``fruit_bowl`` и передаст объект, в который он возвращается ``test_fruit_salad`` как аргумент
``fruit_bowl``.

Вот примерно что случилось бы, если бы мы сделали это вручную:

.. code-block:: python

    def fruit_bowl():
        return [Fruit("apple"), Fruit("banana")]


    def test_fruit_salad(fruit_bowl):
        # Выполнение
        fruit_salad = FruitSalad(*fruit_bowl)

        # Проверка
        assert all(fruit.cubed for fruit in fruit_salad.fruit)


    # Настройка
    bowl = fruit_bowl()
    test_fruit_salad(fruit_bowl=bowl)


Фикстуры могут **запрашивать** другие фикстуры
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Одной из самых сильных сторон pytest является его чрезвычайно гибкая система фикстур. Это позволяет
нам свести сложные требования к тестам к более простым и организованным функциям, где нам нужно только,
чтобы каждая из них описывала то, от чего они зависят. Мы рассмотрим это подробнее ниже, а пока вот
небольшой пример, демонстрирующий как фикстуры могут использовать другие фикстуры:

.. code-block:: python

    # листинг test_append.py
    import pytest


    # Настройка
    @pytest.fixture
    def first_entry():
        return "a"


    # Настройка
    @pytest.fixture
    def order(first_entry):
        return [first_entry]


    def test_string(order):
        # Выполнение
        order.append("b")

        # Проверка
        assert order == ["a", "b"]


Обратите внимание, что это тот же пример что и выше, но с очень небольшими изменениями. Фикстуры в
pytest **запрашивают** фикстуры так же, как и тесты. Все те же правила запроса применяются к фикстурам,
предназначенным для тестов. Вот как работал бы этот пример, если бы мы делали это вручную:

.. code-block:: python

    def first_entry():
        return "a"


    def order(first_entry):
        return [first_entry]


    def test_string(order):
        # Выполнение
        order.append("b")

        # Проверка
        assert order == ["a", "b"]


    entry = first_entry()
    the_list = order(first_entry=entry)
    test_string(order=the_list)

Фикстуры можно переиспользовать
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Одна из вещей, которая делает систему фикстур pytest такой мощной, заключается в том, что она дает нам
возможность определять общий шаг настройки, который можно повторно использовать снова и снова, как если
бы использовалась обычная функция. Два разных теста могут запрашивать одно и ту же фикстуру, и
pytest дает каждому тесту свой результат из этой фикстуры.

Это чрезвычайно полезно, чтобы убедиться, что тесты не влияют друг на друга. Мы можем использовать
эту систему, чтобы убедиться, что каждый тест получает свой собственный свежий пакет данных и
запускается с чистого состояния, чтобы обеспечить согласованные, повторяемые результаты.

Вот пример того, как это можно использовать:

.. code-block:: python

    # листинг test_append.py
    import pytest


    # Настройка
    @pytest.fixture
    def first_entry():
        return "a"


    # Настройка
    @pytest.fixture
    def order(first_entry):
        return [first_entry]


    def test_string(order):
        # Выполнение
        order.append("b")

        # Проверка
        assert order == ["a", "b"]


    def test_int(order):
        # Выполнение
        order.append(2)

        # Проверка
        assert order == ["a", 2]


Каждому тесту здесь предоставляется собственная копия этого объекта ``list``, что означает, что фикстура
``order`` выполняется дважды (то же самое верно и для фикстуры ``first_entry``). Если бы мы тоже делали это
вручную, это выглядело бы примерно так:

.. code-block:: python

    def first_entry():
        return "a"


    def order(first_entry):
        return [first_entry]


    def test_string(order):
        # выполнение
        order.append("b")

        # проверка
        assert order == ["a", "b"]


    def test_int(order):
        # выполнение
        order.append(2)

        # проверка
        assert order == ["a", 2]


    entry = first_entry()
    the_list = order(first_entry=entry)
    test_string(order=the_list)

    entry = first_entry()
    the_list = order(first_entry=entry)
    test_int(order=the_list)

Тест/фикстура может **запрашивать** более одной фикстуры одновременно
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Тесты и фикстуры не ограничиваются запросом одной фикстуры за раз. Они могут запросить столько,
сколько захотят. Вот еще один простой пример:

.. code-block:: python

    # листинг test_append.py
    import pytest


    # настройка
    @pytest.fixture
    def first_entry():
        return "a"


    # настройка
    @pytest.fixture
    def second_entry():
        return 2


    # настройка
    @pytest.fixture
    def order(first_entry, second_entry):
        return [first_entry, second_entry]


    # настройка
    @pytest.fixture
    def expected_list():
        return ["a", 2, 3.0]


    def test_string(order, expected_list):
        # выполнение
        order.append(3.0)

        # проверка
        assert order == expected_list

Фикстуры могут быть запрошены более одного раза за тест (возвращаемые значения кешируются)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Фикстуры также могут быть **запрошены** более одного раза во время одного и того же теста, и pytest не
будет выполнять их повторно для этого теста. Это означает, что мы можем **запрашивать** фикстуры в нескольких
фикстурах, которые зависят от них (и даже в самом тесте), без того, чтобы эти фикстуры выполнялись
более одного раза.

.. code-block:: python

    # листинг test_append.py
    import pytest


    # настройка
    @pytest.fixture
    def first_entry():
        return "a"


    # настройка
    @pytest.fixture
    def order():
        return []


    # выполнение
    @pytest.fixture
    def append_first(order, first_entry):
        return order.append(first_entry)


    def test_string_only(append_first, order, first_entry):
        # проверка
        assert order == [first_entry]

Если **запрошенная** фикстура выполнилась один раз для каждого раза, когда она была **запршена** во время
теста, то этот тест завершится неудачно, потому что и ``append_first`` и ``test_string_only`` увидят
``order`` как пустой список (т.е. ``[]``), но поскольку возвращаемое значение ``order`` было кэшировано
(вместе с любыми сторонним эффектами, которые могли быть) после первого вызова, и тест, и ``append_first``
ссылались на тот же объект, и тест увидел эффект ``append_first`` на этом объекте.

.. _`autouse`:
.. _`autouse fixtures`:

Автоамтически(Autouse) выполняемые фикстуры(которые не нужно запрашивать)
--------------------------------------------------------------------------

Иногда вам может понадобиться фикстура(или даже несколько), от которой, как вы знаете, будут зависеть
все ваши тесты. «Autouse» - это удобный способ сделать так, чтобы все тесты автоматически
**запрашивали** их. Это может сократить количество избыточных **запросов** и даже обеспечить более продвинутое
использование фикстур(подробнее об этом ниже).

Мы можем сделать фикстуру "autouse", передав параметр ``autouse=True`` в декоратор фикстуры.
Вот простой пример того, как их можно использовать:

.. code-block:: python

    # листинг test_append.py
    import pytest


    @pytest.fixture
    def first_entry():
        return "a"


    @pytest.fixture
    def order(first_entry):
        return []


    @pytest.fixture(autouse=True)
    def append_first(order, first_entry):
        return order.append(first_entry)


    def test_string_only(order, first_entry):
        assert order == [first_entry]


    def test_string_and_int(order, first_entry):
        order.append(2)
        assert order == [first_entry, 2]

В этом примере фикстура ``append_first`` является autouse-фикстурой. Поскольку это происходит автоматически,
это влияет на оба теста, даже если ни один из тестов этого не **запрашивал**. Это не значит, что их *нельзя*
**запрашивать**; просто в этом нет необходимости.

.. _smtpshared:

Scope: совместное использование фикстур между классами, модулями, пакетами или сессией
------------------------------------------------------------------------------------------

.. regendoc:wipe

Фикстуры, требующие доступа к сети, зависят от возможности подключения и обычно требуют больших
затрат времени на создание. Расширяя предыдущий пример, мы можем добавить параметр ``scope="module"`` к
вызову :py:func:`@pytest.fixture <pytest.fixture>`, чтобы вызвать функцию фиксации ``smtp_connection``,
отвечающую за для создания соединения с уже существующим SMTP-сервером, которое будет вызываться
только один раз для каждого тестового *модуля*(по умолчанию вызывается один раз для каждой тестовой
*функции*). Таким образом, несколько тестовых функций в тестовом модуле получат один и тот же экземпляр
фикстуры ``smtp_connection``, что сэкономит время. Возможные значения для ``scope``: ``function``,
``class``, ``module``, ``package`` или ``session``.

В следующем примере функция фикстуры помещается в отдельный файл ``conftest.py``, чтобы тесты из нескольких
тестовых модулей в каталоге могли получить доступ к функции фикстуры:

.. code-block:: python

    # листинг conftest.py
    import pytest
    import smtplib


    @pytest.fixture(scope="module")
    def smtp_connection():
        return smtplib.SMTP("smtp.gmail.com", 587, timeout=5)


.. code-block:: python

    # листинг test_module.py


    def test_ehlo(smtp_connection):
        response, msg = smtp_connection.ehlo()
        assert response == 250
        assert b"smtp.gmail.com" in msg
        assert 0  # для демонстрационных целей


    def test_noop(smtp_connection):
        response, msg = smtp_connection.noop()
        assert response == 250
        assert 0  # для демонстрационных целей

Здесь для ``test_ehlo`` необходимо значение фикстуры ``smtp_connection``. pytest
откроет и вызовет :py:func:`@pytest.fixture <pytest.fixture>`
отмеченная функция фикстуры ``smtp_connection``.  Запуск теста выглядит так:

.. code-block:: pytest

    $ pytest test_module.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collected 2 items

    test_module.py FF                                                    [100%]

    ================================= FAILURES =================================
    ________________________________ test_ehlo _________________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_ehlo(smtp_connection):
            response, msg = smtp_connection.ehlo()
            assert response == 250
            assert b"smtp.gmail.com" in msg
    >       assert 0  # for demo purposes
    E       assert 0

    test_module.py:7: AssertionError
    ________________________________ test_noop _________________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_noop(smtp_connection):
            response, msg = smtp_connection.noop()
            assert response == 250
    >       assert 0  # for demo purposes
    E       assert 0

    test_module.py:13: AssertionError
    ========================= short test summary info ==========================
    FAILED test_module.py::test_ehlo - assert 0
    FAILED test_module.py::test_noop - assert 0
    ============================ 2 failed in 0.12s =============================

Вы видите, что два ``assert 0`` упали, и, что более важно, вы также можете видеть, что точно
такой же объект ``smtp_connection`` был передан в две тестовые функции, потому что pytest показывает
входящие значения аргументов в трассировке. В результате две тестовые функции, использующие
``smtp_connection``, выполняются так же быстро, как и одна, потому что они повторно используют один и тот
же экземпляр.

Если вы решите, что необходим экземпляр ``smtp_connection`` в области сеанса, вы можете просто объявить его:

.. code-block:: python

    @pytest.fixture(scope="session")
    def smtp_connection():
        # возвращенное значение фикстуры будет передано для
        # всех тестов, запрашивающих это
        ...


Области фикстур
^^^^^^^^^^^^^^^^^

Фикстуры создаются при первом запросе теста и удаляются в зависимости от их ``scope``:

* ``function``: область действия по умолчанию, фикстура удаляется в конце теста.
* ``class``: фикстура уничтожается во время очистки(teardown) последнего теста в классе.
* ``module``: фикстура уничтожается  во время очистки последнего теста в модуле.
* ``package``: фикстура уничтожается  во время очистки последнего теста в пакете.
* ``session``: фикстура уничтожается в конце тестовой сессии.

.. note::

    Pytest кэширует только один экземпляр фикстуры за раз, что означает, что при использовании
    параметризованной фикстуры, pytest может вызывать фикстуру более одного раза в заданной области.

.. _dynamic scope:

Динамическая область
^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 5.2

В некоторых случаях может потребоваться изменить область действия фикстуры без изменения кода. Для этого
передайте вызываемый объект в ``scope``. Вызываемый объект должен возвращать строку с допустимой областью
видимости и будет выполнен только один раз - во время определения фикстуры. Он будет вызываться с
двумя ключевыми аргументами - ``fixture_name`` в виде строки и ``config`` с объектом конфигурации.

Это может быть особенно полезно при работе с фикстурами, которым требуется время для настройки, например,
при создании контейнера докера. Вы можете использовать аргумент командной строки для управления
областью порожденных контейнеров для различных сред. См. Пример ниже.

.. code-block:: python

    def determine_scope(fixture_name, config):
        if config.getoption("--keep-containers", None):
            return "session"
        return "function"


    @pytest.fixture(scope=determine_scope)
    def docker_container():
        yield spawn_container()



.. _`finalization`:

Teardown/Cleanup(также известная как финализация фикстуры)
------------------------------------------------------------

Когда мы запускаем наши тесты, мы хотим убедиться, что они убирают за собой, чтобы они не вмешивались
в другие тесты (а также чтобы мы не оставили после себя горы тестовых данных, которые раздувают систему).
Фикстуры в pytest предлагают очень полезную систему очистки, которая позволяет нам определять
конкретные шаги, необходимые для очистки каждой фикстуры после себя.

Эту систему можно использовать двумя способами.

.. _`yield fixtures`:

1. Фикстуры ``yield`` (рекомендовано)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. regendoc: wipe

Используем *выход*-фикстуру ``yield`` вместо ``return``. С помощью этих фикстур мы можем запустить некоторый
код и передать объект обратно запрашивающему fixture/test, как и в случае с другими фикстурами.
Единственные отличия:

1. ``return`` заменен на ``yield``.
2. Любой код очистки для этой фикстуры помещается *после* ``yield``.

Как только pytest определит линейный порядок для фикстур, он будет запускать каждый из них до тех пор,
пока он не вернется или не даст результатов, а затем перейдет к следующей фикстуре в списке,
чтобы сделать то же самое.

После завершения теста pytest вернется вниз по списку фикстур, но в *обратном порядке*, беря каждое из них
и запуская внутри него код, который был *после* оператора ``yield``.

В качестве простого примера рассмотрим этот базовый модуль email:

.. code-block:: python

    # листинг emaillib.py
    class MailAdminClient:
        def create_user(self):
            return MailUser()

        def delete_user(self, user):
            # сделаем некоторую очистку
            pass


    class MailUser:
        def __init__(self):
            self.inbox = []

        def send_email(self, email, other):
            other.inbox.append(email)

        def clear_mailbox(self):
            self.inbox.clear()


    class Email:
        def __init__(self, subject, body):
            self.subject = subject
            self.body = body

Допустим, мы хотим протестировать отправку электронной почты от одного пользователя другому. Нам нужно
сначала создать каждого пользователя, затем отправить электронное письмо от одного пользователя другому
и, наконец, подтвердить, что другой пользователь получил это сообщение в своем почтовом ящике. Если мы
хотим очистить после запуска теста, нам, вероятно, придется убедиться, что почтовый ящик другого
пользователя пуст, прежде чем удалять этого пользователя, иначе система может выдать предупреждение.

Вот как это может выглядеть:

.. code-block:: python

    # листинг test_emaillib.py
    import pytest

    from emaillib import Email, MailAdminClient


    @pytest.fixture
    def mail_admin():
        return MailAdminClient()


    @pytest.fixture
    def sending_user(mail_admin):
        user = mail_admin.create_user()
        yield user
        mail_admin.delete_user(user)


    @pytest.fixture
    def receiving_user(mail_admin):
        user = mail_admin.create_user()
        yield user
        mail_admin.delete_user(user)


    def test_email_received(sending_user, receiving_user):
        email = Email(subject="Hey!", body="How's it going?")
        sending_user.send_email(email, receiving_user)
        assert email in receiving_user.inbox

Поскольку ``receiving_user`` - это последняя фикстура, запускаемая во время установки, она запускается первой
во время очистки.

Существует риск того, что даже наведение порядка при разборке не гарантирует безопасную уборку.
Подробнее об этом можно прочитать в :ref:`safe teardowns`.

.. code-block:: pytest

   $ pytest -q test_emaillib.py
   .                                                                    [100%]
   1 passed in 0.12s

Обработка ошибок для фикстуры yield
"""""""""""""""""""""""""""""""""""""

Если yield-фикстура вызывает исключение перед уступкой, pytest не будет пытаться запустить код
уступки после оператора ``yield`` yield-фикстуры. Но для каждой фикстуры, которая уже была успешно
запущена для этого теста, pytest все равно будет пытаться удалить их, как обычно.

2. Добавление финализаторов напрямую
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Хотя yield-фикстуры считаются более правильным и простым вариантом, есть и другой вариант, а именно
добавление «финализатор»-функций непосредственно к объекту `request-context`_ теста. Это дает тот же
результат, что и yield-фикстура, но требует немного большего объема кода.

Чтобы использовать этот подход, мы должны запросить объект `request-context`_ (точно так же, как мы
запрашиваем другую фикстуру) в фикстуре, для которого нам нужно добавить teardown-код, а затем
передать вызываемый объект, содержащий этот teardown-код, к его методу ``addfinalizer``.

Однако нужно быть осторожным, потому что pytest будет запускать этот финализатор после его добавления,
даже если эта фикстура вызывает исключение после добавления финализатора. Поэтому, чтобы убедиться, что
мы не запускаем код финализатора, когда он нам не нужен, мы добавляем финализатор только после того,
как фикстура сделает что-то, что нам нужно будет очистить.

Вот как будет выглядеть предыдущий пример с использованием метода ``addfinalizer``:

.. code-block:: python

    # листинг test_emaillib.py
    import pytest

    from emaillib import Email, MailAdminClient


    @pytest.fixture
    def mail_admin():
        return MailAdminClient()


    @pytest.fixture
    def sending_user(mail_admin):
        user = mail_admin.create_user()
        yield user
        mail_admin.delete_user(user)


    @pytest.fixture
    def receiving_user(mail_admin, request):
        user = mail_admin.create_user()

        def delete_user():
            mail_admin.delete_user(user)

        request.addfinalizer(delete_user)
        return user


    @pytest.fixture
    def email(sending_user, receiving_user, request):
        _email = Email(subject="Hey!", body="How's it going?")
        sending_user.send_email(_email, receiving_user)

        def empty_mailbox():
            receiving_user.clear_mailbox()

        request.addfinalizer(empty_mailbox)
        return _email


    def test_email_received(receiving_user, email):
        assert email in receiving_user.inbox

Он немного длиннее, чем yield-фикстура, и немного сложнее, но он предлагает некоторые нюансы,
когда вы в затруднительном положении.

.. code-block:: pytest

   $ pytest -q test_emaillib.py
   .                                                                    [100%]
   1 passed in 0.12s

.. _`safe teardowns`:

Безопасные очистки
--------------------

Система фикстур pytest очень мощная, но она все еще запускается компьютером, поэтому он не может понять,
как безопасно очистить все, что мы делаем. Если мы не будем осторожны, ошибка в неправильном
месте может привести к тому, что наши тесты останутся позади, и это может довольно быстро вызвать
дополнительные проблемы.

Например, рассмотрим следующие тесты (основанные на примере выше):

.. code-block:: python

    # листинг test_emaillib.py
    import pytest

    from emaillib import Email, MailAdminClient


    @pytest.fixture
    def setup():
        mail_admin = MailAdminClient()
        sending_user = mail_admin.create_user()
        receiving_user = mail_admin.create_user()
        email = Email(subject="Hey!", body="How's it going?")
        sending_user.send_email(email, receiving_user)
        yield receiving_user, email
        receiving_user.clear_mailbox()
        mail_admin.delete_user(sending_user)
        mail_admin.delete_user(receiving_user)


    def test_email_received(setup):
        receiving_user, email = setup
        assert email in receiving_user.inbox

Эта версия намного компактнее, но ее сложнее читать, у нее нет описательного названия фикстуры, и ни
одна из фикстур не может быть легко использована повторно.

Существует также более серьезная проблема, заключающаяся в том, что если какой-либо из этих шагов в
настройке вызывает исключение, ни один из кодов очистки не запустится.

Одним из вариантов может быть использование метода ``addfinalizer`` вместо yield-фикстур, но это может
оказаться довольно сложным и трудным в обслуживании (и больше не будет компактным).

.. code-block:: pytest

   $ pytest -q test_emaillib.py
   .                                                                    [100%]
   1 passed in 0.12s

.. _`safe fixture structure`:

Безопасная конструкция фикстуры
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Самая безопасная и простая структура фикстуры требует, чтобы фикстуры выполняли только одно действие по
изменению состояния каждое, а затем связывали их вместе с их кодом очистки, как показано в примерах
:ref:`the email examples above <yield fixtures>`.

Вероятность того, что операция изменения состояния может упасть, но все же изменить
состояние, незначительна, поскольку большинство этих операций, как правило, основаны на транзакциях
`transaction <https://en.wikipedia.org/wiki/Transaction_processing>`_ (по крайней мере, на уровне
тестирования, где состояние может быть оставлен позади). Поэтому, если мы убедимся, что любое успешное
действие, изменяющее состояние, прерывается, перемещая его в отдельную фикстурную функцию и отделяя его
от других, потенциально неуспешных действий по изменению состояния, тогда наши тесты будут иметь лучший
шанс оставить тестовую среду в предыдущем состоянии.

Например, предположим, что у нас есть веб-сайт со страницей входа, и у нас есть доступ к административному
API, где мы можем создавать пользователей. Для нашего теста мы хотим:

1. Создаем пользователя через административный API
2. Запускаем браузер с помощью Selenium
3. Перейдем на страницу авторизации на нашем сайте
4. Авторизуемся как созданный нами пользователь
5. Подтверждаем, что их имя указано в заголовке целевой страницы.

Мы не хотели бы оставлять этого пользователя в системе или оставлять этот сеанс браузера запущенным,
поэтому мы хотим убедиться, что фикстуры, которые создают это, очищаются после себя.

Вот как это может выглядеть:

.. note::

    В этом примере некоторые фикстуры(т.е. ``base_url`` и ``admin_credentials``)
    подразумевается, что они существуют где-то еще. Итак, пока давайте
    предполагаем, что они существуют, и мы просто не смотрим на них.

.. code-block:: python

    from uuid import uuid4
    from urllib.parse import urljoin

    from selenium.webdriver import Chrome
    import pytest

    from src.utils.pages import LoginPage, LandingPage
    from src.utils import AdminApiClient
    from src.utils.data_types import User


    @pytest.fixture
    def admin_client(base_url, admin_credentials):
        return AdminApiClient(base_url, **admin_credentials)


    @pytest.fixture
    def user(admin_client):
        _user = User(name="Susan", username=f"testuser-{uuid4()}", password="P4$$word")
        admin_client.create_user(_user)
        yield _user
        admin_client.delete_user(_user)


    @pytest.fixture
    def driver():
        _driver = Chrome()
        yield _driver
        _driver.quit()


    @pytest.fixture
    def login(driver, base_url, user):
        driver.get(urljoin(base_url, "/login"))
        page = LoginPage(driver)
        page.login(user)


    @pytest.fixture
    def landing_page(driver, login):
        return LandingPage(driver)


    def test_name_on_landing_page_after_login(landing_page, user):
        assert landing_page.header == f"Welcome, {user.name}!"

То, как установлены зависимости, означает, что неясно, будет ли фикстура ``user`` выполняться
перед фикстурой ``driver``. Но это нормально, потому что это атомарные операции, и поэтому не имеет
значения, какая из них выполняется первой, потому что последовательность событий для теста по-прежнему
является `linearizable <https: en.wikipedia.orgwikiLinearizability>`_. Но важно то, что независимо от
того, какая из них запускается первой, если одна вызовет исключение, а другая нет, ни одна из них ничего
не оставит. Если ``driver`` выполняется до ``user``, а ``user`` вызывает исключение, driver все равно завершится,
и пользователь никогда не будет создан. И если ``driver`` был тем, кто вызвал исключение, то ``driver``
никогда не был бы запущен, и пользователь никогда не был бы создан.

.. note:

    Хотя фикстура ``user`` на самом деле не обязательно должна быть перед фикстурой
    ``driver``, если мы сделаем с ``driver`` запрос ``user``, это может сэкономить некоторое
    время в том случае, если user вызывает исключение, так как он не будет пытаться
    запустить driver, что является довольно дорогостоящей операцией.


Безопасный запуск нескольких ``assert``
------------------------------------------------------

Иногда вы можете захотеть запустить несколько ``assert`` после выполнения всей настройки,
что имеет смысл, поскольку в более сложных системах одно действие может запускать несколько вариантов
поведения. У pytest есть удобный способ сделать это, и он сочетает в себе многое из того, что мы уже
рассмотрели.

Все, что нужно, - это перейти к большему объему, затем определить шаг **действие** как фикстуру
``autouse`` и, наконец, убедиться, что все фикстуры указывают на более высокий уровень.

Давайте возьмем :ref:`an example from above <safe fixture structure>` и немного подправим его. Предположим,
что помимо проверки приветственного сообщения в заголовке, мы также хотим проверить кнопку выхода и
ссылку на профиль пользователя.

Давайте посмотрим, как мы можем структурировать это, чтобы мы могли запускать несколько утверждений,
не повторяя все эти шаги снова.

.. note::

    Для этого примера, определенные фикстуры(т.е. ``base_url`` и
    ``admin_credentials``) подразумевается, что существуют где-то еще. Итак, пока давайте
    предположим, что они существуют, и мы просто не смотрим на них.

.. code-block:: python

    # листинг tests/end_to_end/test_login.py
    from uuid import uuid4
    from urllib.parse import urljoin

    from selenium.webdriver import Chrome
    import pytest

    from src.utils.pages import LoginPage, LandingPage
    from src.utils import AdminApiClient
    from src.utils.data_types import User


    @pytest.fixture(scope="class")
    def admin_client(base_url, admin_credentials):
        return AdminApiClient(base_url, **admin_credentials)


    @pytest.fixture(scope="class")
    def user(admin_client):
        _user = User(name="Susan", username=f"testuser-{uuid4()}", password="P4$$word")
        admin_client.create_user(_user)
        yield _user
        admin_client.delete_user(_user)


    @pytest.fixture(scope="class")
    def driver():
        _driver = Chrome()
        yield _driver
        _driver.quit()


    @pytest.fixture(scope="class")
    def landing_page(driver, login):
        return LandingPage(driver)


    class TestLandingPageSuccess:
        @pytest.fixture(scope="class", autouse=True)
        def login(self, driver, base_url, user):
            driver.get(urljoin(base_url, "/login"))
            page = LoginPage(driver)
            page.login(user)

        def test_name_in_header(self, landing_page, user):
            assert landing_page.header == f"Welcome, {user.name}!"

        def test_sign_out_button(self, landing_page):
            assert landing_page.sign_out_button.is_displayed()

        def test_profile_link(self, landing_page, user):
            profile_href = urljoin(base_url, f"/profile?id={user.profile_id}")
            assert landing_page.profile_link.get_attribute("href") == profile_href

Обратите внимание, что методы только формально ссылаются на ``self`` в подписи. Никакое состояние не
связано с фактическим тестовым классом, как это может быть в структуре ``unittest.TestCase``. Все
управляется системой фикстур pytest.

Каждый метод должен запрашивать только те фикстуры, которые ему действительно нужны, не беспокоясь о
порядке. Это связано с тем, что фикстура **выполнение** является autouse-фикстурой, и это обеспечило
выполнение всех остальных фикстур перед ней. Больше нет необходимости в изменении состояния, поэтому
тесты могут выполнять столько запросов без изменения состояния, сколько они хотят, не рискуя мешать
другим тестам.

Фикстура ``login`` также определена внутри класса, потому что не все другие тесты в модуле будут ожидать
успешного входа в систему, и действие, возможно, придется обрабатывать немного по-другому для другого
тестового класса. Например, если мы хотим написать еще один тестовый сценарий для отправки неверных
учетных данных, мы могли бы сделать это, добавив следующее в тестовый файл:

.. note:

    Предполагается, что объект страницы(т.е. ``LoginPage``) вызывает настраиваемое
    исключение ``BadCredentialsException``, когда распознает текст, обозначающий это, в форме входа
    после попытки входа в систему.

.. code-block:: python

    class TestLandingPageBadCredentials:
        @pytest.fixture(scope="class")
        def faux_user(self, user):
            _user = deepcopy(user)
            _user.password = "badpass"
            return _user

        def test_raises_bad_credentials_exception(self, login_page, faux_user):
            with pytest.raises(BadCredentialsException):
                login_page.login(faux_user)


.. _`request-context`:

Фикстуры могут анализировать запрашиваемый тестовый контекст
-------------------------------------------------------------

Фикстуры могут принимать объект :py:class:`request <_pytest.fixtures.FixtureRequest>` для интроспекции
«запрашивающей» тестовой функции, класса или контекста модуля. Далее, расширяя предыдущий пример
фикстуры ``smtp_connection``, давайте прочитаем необязательный URL-адрес сервера из тестового модуля,
который использует нашу фикстуру:

.. code-block:: python

    # листинг conftest.py
    import pytest
    import smtplib


    @pytest.fixture(scope="module")
    def smtp_connection(request):
        server = getattr(request.module, "smtpserver", "smtp.gmail.com")
        smtp_connection = smtplib.SMTP(server, 587, timeout=5)
        yield smtp_connection
        print("finalizing {} ({})".format(smtp_connection, server))
        smtp_connection.close()

Мы используем атрибут ``request.module`` чтобы дополнительно получить атрибут ``smtpserver`` из тестового
модуля. Если мы просто запустим еще раз, ничего особенного не изменится:

.. code-block:: pytest

    $ pytest -s -q --tb=no test_module.py
    FFfinalizing <smtplib.SMTP object at 0xdeadbeef> (smtp.gmail.com)

    ========================= short test summary info ==========================
    FAILED test_module.py::test_ehlo - assert 0
    FAILED test_module.py::test_noop - assert 0
    2 failed in 0.12s

Давайте быстро создадим еще один тестовый модуль, который фактически устанавливает URL-адрес сервера в
пространстве имен модуля:

.. code-block:: python

    # листинг test_anothersmtp.py

    smtpserver = "mail.python.org"  # будет прочитано фикстурой smtp


    def test_showhelo(smtp_connection):
        assert 0, smtp_connection.helo()

Запустим:

.. code-block:: pytest

    $ pytest -qq --tb=short test_anothersmtp.py
    F                                                                    [100%]
    ================================= FAILURES =================================
    ______________________________ test_showhelo _______________________________
    test_anothersmtp.py:6: in test_showhelo
        assert 0, smtp_connection.helo()
    E   AssertionError: (250, b'mail.python.org')
    E   assert 0
    ------------------------- Captured stdout teardown -------------------------
    finalizing <smtplib.SMTP object at 0xdeadbeef> (mail.python.org)
    ========================= short test summary info ==========================
    FAILED test_anothersmtp.py::test_showhelo - AssertionError: (250, b'mail....

Вуаля! Фикстура ``smtp_connection`` взяла имя нашего почтового сервера из пространства имен модуля.

.. _`using-markers`:

Использование маркеров для передачи данных в фикстуры
-------------------------------------------------------------

Используя объект :py:class:`request <_pytest.fixtures.FixtureRequest>`, фикстура также может получить
доступ к маркерам, которые применяются к тестовой функции. Это может быть полезно для передачи данных
в фикстуру из теста:

.. code-block:: python

    import pytest


    @pytest.fixture
    def fixt(request):
        marker = request.node.get_closest_marker("fixt_data")
        if marker is None:
            # Каким-то образом обработаем отсутствующий маркер...
            data = None
        else:
            data = marker.args[0]

        # Сделаем что-нибудь с данными
        return data


    @pytest.mark.fixt_data(42)
    def test_fixt(fixt):
        assert fixt == 42

.. _`fixture-factory`:

Фабрики как фикстуры
---------------------------

Шаблон «Фабрика как фикстура» может помочь в ситуациях, когда результат фикстуры требуется
несколько раз в одном тесте. Вместо того, чтобы возвращать данные напрямую, фикстура возвращает функцию,
которая генерирует данные. Затем эту функцию можно вызывать в тесте несколько раз.

Фабрики могут получать параметры по мере необходимости:

.. code-block:: python

    @pytest.fixture
    def make_customer_record():
        def _make_customer_record(name):
            return {"name": name, "orders": []}

        return _make_customer_record


    def test_customer_records(make_customer_record):
        customer_1 = make_customer_record("Lisa")
        customer_2 = make_customer_record("Mike")
        customer_3 = make_customer_record("Meredith")

Если данные, созданные фабрикой, требуют управления, фикстура позаботится об этом:

.. code-block:: python

    @pytest.fixture
    def make_customer_record():

        created_records = []

        def _make_customer_record(name):
            record = models.Customer(name=name, orders=[])
            created_records.append(record)
            return record

        yield _make_customer_record

        for record in created_records:
            record.destroy()


    def test_customer_records(make_customer_record):
        customer_1 = make_customer_record("Lisa")
        customer_2 = make_customer_record("Mike")
        customer_3 = make_customer_record("Meredith")


.. _`fixture-parametrize`:

Параметризация фикстур
-----------------------------------------------------------------

Функции фикстуры можно параметризовать, и в этом случае они будут вызываться несколько раз, каждый раз
выполняя набор зависимых тестов, то есть тестов, которые зависят от этой фикстуры. Тестовым функциям
обычно не нужно знать об их повторном запуске. Параметризация фикстур помогает писать
исчерпывающие функциональные тесты для компонентов, которые сами могут быть настроены различными
способами.

Расширяя предыдущий пример, мы можем пометить прибор, чтобы создать два экземпляра фикстуры
``smtp_connection``, которые заставят все тесты, использующие фикстуру, запускаться дважды. Функция
фикстуры получает доступ к каждому параметру через специальный объект
:py:class:`request <FixtureRequest>`:

.. code-block:: python

    # листинг conftest.py
    import pytest
    import smtplib


    @pytest.fixture(scope="module", params=["smtp.gmail.com", "mail.python.org"])
    def smtp_connection(request):
        smtp_connection = smtplib.SMTP(request.param, 587, timeout=5)
        yield smtp_connection
        print("finalizing {}".format(smtp_connection))
        smtp_connection.close()

Основным изменением является объявление ``params`` с помощью :py:func:`@pytest.fixture <pytest.fixture>`,
списка значений, для каждого из которых функция фикстуры будет выполняться и может получить доступ к
значению через ``request.param``. Код тестовой функции изменять не нужно. Итак, давайте просто сделаем
еще один прогон:

.. code-block:: pytest

    $ pytest -q test_module.py
    FFFF                                                                 [100%]
    ================================= FAILURES =================================
    ________________________ test_ehlo[smtp.gmail.com] _________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_ehlo(smtp_connection):
            response, msg = smtp_connection.ehlo()
            assert response == 250
            assert b"smtp.gmail.com" in msg
    >       assert 0  # for demo purposes
    E       assert 0

    test_module.py:7: AssertionError
    ________________________ test_noop[smtp.gmail.com] _________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_noop(smtp_connection):
            response, msg = smtp_connection.noop()
            assert response == 250
    >       assert 0  # for demo purposes
    E       assert 0

    test_module.py:13: AssertionError
    ________________________ test_ehlo[mail.python.org] ________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_ehlo(smtp_connection):
            response, msg = smtp_connection.ehlo()
            assert response == 250
    >       assert b"smtp.gmail.com" in msg
    E       AssertionError: assert b'smtp.gmail.com' in b'mail.python.org\nPIPELINING\nSIZE 51200000\nETRN\nSTARTTLS\nAUTH DIGEST-MD5 NTLM CRAM-MD5\nENHANCEDSTATUSCODES\n8BITMIME\nDSN\nSMTPUTF8\nCHUNKING'

    test_module.py:6: AssertionError
    -------------------------- Captured stdout setup ---------------------------
    finalizing <smtplib.SMTP object at 0xdeadbeef>
    ________________________ test_noop[mail.python.org] ________________________

    smtp_connection = <smtplib.SMTP object at 0xdeadbeef>

        def test_noop(smtp_connection):
            response, msg = smtp_connection.noop()
            assert response == 250
    >       assert 0  # for demo purposes
    E       assert 0

    test_module.py:13: AssertionError
    ------------------------- Captured stdout teardown -------------------------
    finalizing <smtplib.SMTP object at 0xdeadbeef>
    ========================= short test summary info ==========================
    FAILED test_module.py::test_ehlo[smtp.gmail.com] - assert 0
    FAILED test_module.py::test_noop[smtp.gmail.com] - assert 0
    FAILED test_module.py::test_ehlo[mail.python.org] - AssertionError: asser...
    FAILED test_module.py::test_noop[mail.python.org] - assert 0
    4 failed in 0.12s

Мы видим, что каждая из наших двух тестовых функций запускалась дважды с разными экземплярами
``smtp_connection``. Также обратите внимание, что с подключением ``mail.python.org`` второй тест не
проходит в ``test_ehlo``, потому что ожидается другая строка сервера, отличная от полученной.

pytest построит строку, которая является идентификатором теста для каждого значения фикстуры в
параметризованной фикстуре, например ``test_ehlo[smtp.gmail.com]`` и ``test_ehlo[mail.python.org]`` в
приведенных выше примерах. Эти идентификаторы можно использовать с ``-k`` для выбора конкретных случаев
для запуска, и они также будут определять конкретный случай, когда один из них упадет. Запуск
pytest с параметром ``--collect-only`` покажет сгенерированные идентификаторы.

Числа, строки, логические значения и ``None`` будут иметь обычное строковое представление, используемое
в идентификаторе теста. Для других объектов pytest создаст строку на основе имени аргумента. Можно
настроить строку, используемую в идентификаторе теста для определенного значения фикстуры, используя
аргумент ключевого слова ``ids``:

.. code-block:: python

   # листинг test_ids.py
   import pytest


   @pytest.fixture(params=[0, 1], ids=["spam", "ham"])
   def a(request):
       return request.param


   def test_a(a):
       pass


   def idfn(fixture_value):
       if fixture_value == 0:
           return "eggs"
       else:
           return None


   @pytest.fixture(params=[0, 1], ids=idfn)
   def b(request):
       return request.param


   def test_b(b):
       pass

Выше показано, как ``ids`` может быть либо списком строк для использования, либо функцией, которая будет
вызываться со значением фикстуры, а затем должна возвращать строку для использования. В последнем случае,
если функция возвращает ``None``, будет использоваться автоматически сгенерированный идентификатор pytest.

Выполнение вышеуказанных тестов приводит к использованию следующих идентификаторов тестов:

.. code-block:: pytest

   $ pytest --collect-only
   =========================== test session starts ============================
   platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y
   cachedir: $PYTHON_PREFIX/.pytest_cache
   rootdir: $REGENDOC_TMPDIR
   collected 11 items

   <Module test_anothersmtp.py>
     <Function test_showhelo[smtp.gmail.com]>
     <Function test_showhelo[mail.python.org]>
   <Module test_emaillib.py>
     <Function test_email_received>
   <Module test_ids.py>
     <Function test_a[spam]>
     <Function test_a[ham]>
     <Function test_b[eggs]>
     <Function test_b[1]>
   <Module test_module.py>
     <Function test_ehlo[smtp.gmail.com]>
     <Function test_noop[smtp.gmail.com]>
     <Function test_ehlo[mail.python.org]>
     <Function test_noop[mail.python.org]>

   ======================= 11 tests collected in 0.12s ========================

.. _`fixture-parametrize-marks`:

Использование маркеров с параметризованными фикстурами
--------------------------------------------------------

:func:`pytest.param` могут использоваться для нанесения маркеров в наборах значений параметризованных
фикстур так же, как их можно использовать с :ref:`@pytest.mark.parametrize <@pytest.mark.parametrize>`.

Пример:

.. code-block:: python

    # cлистинг test_fixture_marks.py
    import pytest


    @pytest.fixture(params=[0, 1, pytest.param(2, marks=pytest.mark.skip)])
    def data_set(request):
        return request.param


    def test_data(data_set):
        pass

Запуск этого теста будет *пропускать* вызов ``data_set`` со значением ``2``:

.. code-block:: pytest

    $ pytest test_fixture_marks.py -v
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 3 items

    test_fixture_marks.py::test_data[0] PASSED                           [ 33%]
    test_fixture_marks.py::test_data[1] PASSED                           [ 66%]
    test_fixture_marks.py::test_data[2] SKIPPED (unconditional skip)     [100%]

    ======================= 2 passed, 1 skipped in 0.12s =======================

.. _`interdependent fixtures`:

Модульность: использование фикстур из функции фикстуры
----------------------------------------------------------

Помимо использования фикстур в тестовых функциях, функции фикстур могут сами использовать другие
фикстуры. Это способствует модульному дизайну ваших фикстур и позволяет повторно использовать фикстуры
во многих проектах. В качестве простого примера мы можем расширить предыдущий пример и создать экземпляр
объекта ``app``, в который мы вставим уже определенный ``smtp_connection``:

.. code-block:: python

    # листинг test_appsetup.py

    import pytest


    class App:
        def __init__(self, smtp_connection):
            self.smtp_connection = smtp_connection


    @pytest.fixture(scope="module")
    def app(smtp_connection):
        return App(smtp_connection)


    def test_smtp_connection_exists(app):
        assert app.smtp_connection

Здесь мы объявляем фикстуру ``app``, которая получает ранее определенную фикстуру ``smtp_connection`` и
создает с ней экземпляр объекта ``App``. Запустим:

.. code-block:: pytest

    $ pytest -v test_appsetup.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 2 items

    test_appsetup.py::test_smtp_connection_exists[smtp.gmail.com] PASSED [ 50%]
    test_appsetup.py::test_smtp_connection_exists[mail.python.org] PASSED [100%]

    ============================ 2 passed in 0.12s =============================

Из-за параметризации ``smtp_connection`` тест будет запускаться дважды с двумя разными экземплярами ``App``
и соответствующими серверами smtp. Нет необходимости, чтобы фикстура ``app`` знала о параметризации
``smtp_connection``, потому что pytest полностью проанализирует график зависимостей фикстуры.

Обратите внимание, что фикстура ``app`` имеет область видимости ``module`` и использует фикстуру
``smtp_connection`` в области видимости модуля. Пример все равно работал бы, если бы ``smtp_connection``
был кэширован в области видимости ``сессии``: фикстуры с более широкой областью видимости могут
использовать фикстуры с более широкой областью видимости, но не наоборот: фикстура с привязкой к сеансу
не может использовать модуль в области видимости осмысленным образом.


.. _`automatic per-resource grouping`:

Автоматическая группировка тестов по экземплярам фикстур
----------------------------------------------------------

.. regendoc: wipe

pytest минимизирует количество активных фикстур во время тестовых прогонов. Если у вас есть
параметризованная фикстура, то все тесты, использующие ее, сначала будут выполняться с одним экземпляром,
а затем вызываются финализаторы перед созданием следующего экземпляра фикстуры. Помимо прочего, это
упрощает тестирование приложений, которые создают и используют глобальное состояние.

В следующем примере используются две параметризованных фикстуры, одна из которых ограничена для каждого
модуля, и все функции выполняют вызовы ``print``, чтобы показать последовательность действий
setup/teardown:

.. code-block:: python

    # листинг test_module.py
    import pytest


    @pytest.fixture(scope="module", params=["mod1", "mod2"])
    def modarg(request):
        param = request.param
        print("  SETUP modarg", param)
        yield param
        print("  TEARDOWN modarg", param)


    @pytest.fixture(scope="function", params=[1, 2])
    def otherarg(request):
        param = request.param
        print("  SETUP otherarg", param)
        yield param
        print("  TEARDOWN otherarg", param)


    def test_0(otherarg):
        print("  RUN test0 with otherarg", otherarg)


    def test_1(modarg):
        print("  RUN test1 with modarg", modarg)


    def test_2(otherarg, modarg):
        print("  RUN test2 with otherarg {} and modarg {}".format(otherarg, modarg))


Запустим тесты в подробном режиме и с учетом вывода на печать:

.. code-block:: pytest

    $ pytest -v -s test_module.py
    =========================== test session starts ============================
    platform linux -- Python 3.x.y, pytest-6.x.y, py-1.x.y, pluggy-0.x.y -- $PYTHON_PREFIX/bin/python
    cachedir: $PYTHON_PREFIX/.pytest_cache
    rootdir: $REGENDOC_TMPDIR
    collecting ... collected 8 items

    test_module.py::test_0[1]   SETUP otherarg 1
      RUN test0 with otherarg 1
    PASSED  TEARDOWN otherarg 1

    test_module.py::test_0[2]   SETUP otherarg 2
      RUN test0 with otherarg 2
    PASSED  TEARDOWN otherarg 2

    test_module.py::test_1[mod1]   SETUP modarg mod1
      RUN test1 with modarg mod1
    PASSED
    test_module.py::test_2[mod1-1]   SETUP otherarg 1
      RUN test2 with otherarg 1 and modarg mod1
    PASSED  TEARDOWN otherarg 1

    test_module.py::test_2[mod1-2]   SETUP otherarg 2
      RUN test2 with otherarg 2 and modarg mod1
    PASSED  TEARDOWN otherarg 2

    test_module.py::test_1[mod2]   TEARDOWN modarg mod1
      SETUP modarg mod2
      RUN test1 with modarg mod2
    PASSED
    test_module.py::test_2[mod2-1]   SETUP otherarg 1
      RUN test2 with otherarg 1 and modarg mod2
    PASSED  TEARDOWN otherarg 1

    test_module.py::test_2[mod2-2]   SETUP otherarg 2
      RUN test2 with otherarg 2 and modarg mod2
    PASSED  TEARDOWN otherarg 2
      TEARDOWN modarg mod2


    ============================ 8 passed in 0.12s =============================

Вы можете видеть, что параметризованный ресурс ``modarg`` с областью видимости модуля вызвал такой порядок
выполнения тестов, который приводит к минимальному количеству «активных» ресурсов. Финализатор для
параметризованного ресурса ``mod1`` был выполнен до настройки ресурса ``mod2``.

В частности, обратите внимание, что test_0 полностью независим и заканчивается первым. Затем test_1
выполняется с ``mod1``, затем test_2 с ``mod1``, затем test_1 с ``mod2`` и, наконец, test_2 с ``mod2``.

Параметризованный ресурс ``otherarg`` (имеющий область действия) был настроен раньше и отключался после
каждого теста, в котором он использовался.


.. _`usefixtures`:

Использование фикстуры в классах и модулях с ``usefixtures``
---------------------------------------------------------------

.. regendoc:wipe

Иногда тестовым функциям не нужен прямой доступ к объекту фикстуры. Например, тесты могут потребовать
работы с пустым каталогом в качестве текущего рабочего каталога, но в остальном не заботятся о
конкретном каталоге. Вот как вы можете использовать стандартный `tempfile
<http://docs.python.org/library/tempfile.html>`_ и фикстуру pytest для этого. Мы разделяем создание
фикстуры в файле conftest.py:

.. code-block:: python

    # листинг conftest.py

    import os
    import tempfile

    import pytest


    @pytest.fixture
    def cleandir():
        with tempfile.TemporaryDirectory() as newpath:
            old_cwd = os.getcwd()
            os.chdir(newpath)
            yield
            os.chdir(old_cwd)

и объявить его использование в тестовом модуле с помощью маркера ``usefixtures``:

.. code-block:: python

    # листинг test_setenv.py
    import os
    import pytest


    @pytest.mark.usefixtures("cleandir")
    class TestDirectoryInit:
        def test_cwd_starts_empty(self):
            assert os.listdir(os.getcwd()) == []
            with open("myfile", "w") as f:
                f.write("hello")

        def test_cwd_again_starts_empty(self):
            assert os.listdir(os.getcwd()) == []

Из-за маркера ``usefixtures`` для выполнения каждого метода тестирования потребуется фикстура ``cleandir``,
как если бы вы указали аргумент функции ``cleandir`` для каждого из них. Давайте запустим его, чтобы
убедиться, что наша фикстура активирована и тесты проходят:

.. code-block:: pytest

    $ pytest -q
    ..                                                                   [100%]
    2 passed in 0.12s

Вы можете указать несколько фикстур, например:

.. code-block:: python

    @pytest.mark.usefixtures("cleandir", "anotherfixture")
    def test():
        ...

и вы можете указать использование фикстур на уровне тестового модуля, используя :globalvar:`pytestmark`:

.. code-block:: python

    pytestmark = pytest.mark.usefixtures("cleandir")


Также можно поместить фикстуры, необходимые для всех тестов в вашем проекте, в ini-файл.:

.. code-block:: ini

    # content of pytest.ini
    [pytest]
    usefixtures = cleandir


.. warning::

    Обратите внимание, что этот маркер не влияет на **функции фикстуры**. Например,
    это **не будет работать как ожидалось**:

    .. code-block:: python

        @pytest.mark.usefixtures("my_other_fixture")
        @pytest.fixture
        def my_fixture_that_sadly_wont_use_my_other_fixture():
            ...

    В настоящее время это не будет генерировать никаких ошибок или предупреждений, но это
    будет обработано в `#3664 <https://github.com/pytest-dev/pytest/issues/3664>`_.

.. _`override fixtures`:

Переопределение фикстур на разных уровнях
-------------------------------------------

В относительно большом наборе тестов вам, скорее всего, потребуется ``переопределить`` ``глобальную`
или ``корневую`` фикстуру на ``локально`` определенную, чтобы код теста оставался читаемым
и поддерживаемым.

Переопределение фикстур на уровне папки (conftest)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Учитывая, что структура файла тестов:

::

    tests/
        __init__.py

        conftest.py
            # content of tests/conftest.py
            import pytest

            @pytest.fixture
            def username():
                return 'username'

        test_something.py
            # content of tests/test_something.py
            def test_username(username):
                assert username == 'username'

        subfolder/
            __init__.py

            conftest.py
                # content of tests/subfolder/conftest.py
                import pytest

                @pytest.fixture
                def username(username):
                    return 'overridden-' + username

            test_something.py
                # content of tests/subfolder/test_something.py
                def test_username(username):
                    assert username == 'overridden-username'

Как видите, фикстуру с таким же именем можно переопределить для определенного уровня тестовой папки.
Обратите внимание, что к ``базовой`` или ``супер`` фикстуре можно легко получить доступ из
``переопределяющей`` фикстуры - используется в примере выше.

Переопределение фикстуры на уровне тестового модуля
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Учитывая, что структура тестовых файлов следующая:

::

    tests/
        __init__.py

        conftest.py
            # content of tests/conftest.py
            import pytest

            @pytest.fixture
            def username():
                return 'username'

        test_something.py
            # content of tests/test_something.py
            import pytest

            @pytest.fixture
            def username(username):
                return 'overridden-' + username

            def test_username(username):
                assert username == 'overridden-username'

        test_something_else.py
            # content of tests/test_something_else.py
            import pytest

            @pytest.fixture
            def username(username):
                return 'overridden-else-' + username

            def test_username(username):
                assert username == 'overridden-else-username'

В приведенном выше примере фикстура с таким же именем может быть переопределена для определенного
тестового модуля.


Переопределение фикстуры с прямой параметризацией теста
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Учитывая, что структура тестовых файлов следующая:

::

    tests/
        __init__.py

        conftest.py
            # content of tests/conftest.py
            import pytest

            @pytest.fixture
            def username():
                return 'username'

            @pytest.fixture
            def other_username(username):
                return 'other-' + username

        test_something.py
            # content of tests/test_something.py
            import pytest

            @pytest.mark.parametrize('username', ['directly-overridden-username'])
            def test_username(username):
                assert username == 'directly-overridden-username'

            @pytest.mark.parametrize('username', ['directly-overridden-username-other'])
            def test_username_other(other_username):
                assert other_username == 'other-directly-overridden-username-other'

В приведенном выше примере значение фикстуры заменяется значением параметра теста. Обратите внимание,
что значение фикстуры может быть переопределено таким образом, даже если тест не использует его
напрямую (не упоминает об этом в прототипе функции).


Переопределение параметризованной фикстуры непараметризованной и наоборот
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Учитывая, что структура тестовых файлов:

::

    tests/
        __init__.py

        conftest.py
            # content of tests/conftest.py
            import pytest

            @pytest.fixture(params=['one', 'two', 'three'])
            def parametrized_username(request):
                return request.param

            @pytest.fixture
            def non_parametrized_username(request):
                return 'username'

        test_something.py
            # content of tests/test_something.py
            import pytest

            @pytest.fixture
            def parametrized_username():
                return 'overridden-username'

            @pytest.fixture(params=['one', 'two', 'three'])
            def non_parametrized_username(request):
                return request.param

            def test_username(parametrized_username):
                assert parametrized_username == 'overridden-username'

            def test_parametrized_username(non_parametrized_username):
                assert non_parametrized_username in ['one', 'two', 'three']

        test_something_else.py
            # content of tests/test_something_else.py
            def test_username(parametrized_username):
                assert parametrized_username in ['one', 'two', 'three']

            def test_username(non_parametrized_username):
                assert non_parametrized_username == 'username'

В приведенном выше примере параметризованная фикстура заменяется непараметризованной версией, а
непараметризованная фикстура заменяется параметризованной версией для определенного тестового модуля.
То же самое относится и к уровню тестовой папки, что очевидно.


Использование фикстур из других проектов
-------------------------------------------

Обычно проекты, которые предоставляют поддержку pytest, будут использовать точки входа
:ref:`entry points <setuptools entry points>`, поэтому простая установка этих проектов в среду
сделает эти фикстуры доступными для использования.

Если вы хотите использовать фикстуры из проекта, который не использует точки входа, вы можете
определить :globalvar:`pytest_plugins` в вашем верхнем файле ``conftest.py``, чтобы зарегистрировать
этот модуль как плагин.

Предположим, у вас есть фикстуры в ``mylibrary.fixtures``, и вы хотите повторно использовать их в своем
каталоге ``app/tests``.

Все, что вам нужно сделать, это определить :globalvar:`pytest_plugins` в ``app/tests/conftest.py``
указывая на этот модуль.

.. code-block:: python

    pytest_plugins = "mylibrary.fixtures"

Это регистрирует ``mylibrary.fixtures`` как плагин, делая все его фикстуры и хуки доступными для тестов
в ``app/tests``..

.. note::

    Иногда пользователи *импортируют* фикстуры из других проектов для использования, однако это не
    рекомендуется: импорт фикстур в модуль зарегистрирует их в pytest, как *определено* в самом модуле.

    Это имеет незначительные последствия, такие как многократное появление в ``pytest --help``, но
    это *не рекомендуется*, поскольку такое поведение может измениться и перестать работать в будущих
    версиях.
