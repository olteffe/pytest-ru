
.. _`accept example`:

пример: определение и выбор приемочных тестов
--------------------------------------------------------------

.. sourcecode:: python

    # ./conftest.py
    def pytest_option(parser):
        group = parser.getgroup("myproject")
        group.addoption(
            "-A", dest="acceptance", action="store_true", help="run (slow) acceptance tests"
        )


    def pytest_funcarg__accept(request):
        return AcceptFixture(request)


    class AcceptFixture:
        def __init__(self, request):
            if not request.config.getoption("acceptance"):
                pytest.skip("specify -A to run acceptance tests")
            self.tmpdir = request.config.mktemp(request.function.__name__, numbered=True)

        def run(self, *cmd):
            """ вызывается тестовым кодом для выполнения приемочного теста. """
            self.tmpdir.chdir()
            return subprocess.check_output(cmd).decode()


и фактический пример тестовой функции:

.. sourcecode:: python

    def test_some_acceptance_aspect(accept):
        accept.tmpdir.mkdir("somesub")
        result = accept.run("ls", "-la")
        assert "somesub" in result

Если вы запустите этот тест без указания параметра командной строки,
то этот тест будет пропущен с соответствующим сообщением. В противном
случае вы можете начать добавлять удобные и тестовые методы поддержки
к вашему AcceptFixture и управлять запуском инструментов или
приложений, а также предоставлять способы выполнения вывода assertions.

.. _`decorate a funcarg`:

пример: декорирование funcarg в тестовом модуле
--------------------------------------------------------------

Для больших сборок иногда полезно декорировать funcarg только
для определенного тестового модуля.  Мы можем расширить
`accept example`_, поместив его в наш тестовый модуль:

.. sourcecode:: python

    def pytest_funcarg__accept(request):
        # вызов следующей фабрики (находящийся в нашем conftest.py)
        arg = request.getfuncargvalue("accept")
        # создание специального макета в нашей временной папке
        arg.tmpdir.mkdir("special")
        return arg


    class TestSpecialAcceptance:
        def test_sometest(self, accept):
            assert accept.tmpdir.join("special").check()

Наша фабрика уровня модуля будет вызвана первой, и она может попросить свой объект
запроса вызвать следующую фабрику, а затем декорировать свой результат. Этот механизм
позволяет нам оставаться в неведении относительно того, как/где предоставляется аргумент
функции - в нашем примере из `conftest plugin`_.

sidenote: временный каталог, используемый здесь, является экземплярами класса
`py.path.local`_, который обеспечивает многие из методов os.path удобным способом.

.. _`py.path.local`: ../path.html#local
.. _`conftest plugin`: customize.html#conftestplugin
