
"Моргающие"(Flaky) тесты
--------------------------

Flaky-тест - это тест, который демонстрирует периодические или спорадические
падения, которые, кажется, имеют недетерминированное поведение. Иногда проходит, иногда
падает и непонятно почему. На этой странице обсуждаются функции pytest, которые
могут помочь, и другие общие стратегии для их выявления, исправления или уменьшения.

Почему flaky-тесты - это проблема
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flaky-тесты особенно неприятны, когда используется сервер непрерывной интеграции (CI),
поэтому все тесты должны пройти, прежде чем можно будет сделать слияние(merge).
Если результат теста не заслуживает доверия - сбой теста означает, что изменение кода
нарушило тест и разработчики могут с недоверием относиться к результатам теста, что
может привести к игнорированию реальных сбоев.
Это также приводит к потере времени, поскольку разработчикам приходится повторно
запускать наборы тестов и исследовать ложные сбои.


Возможные первопричины
^^^^^^^^^^^^^^^^^^^^^^^

Состояние системы
~~~~~~~~~~~~~~~~~~

Вообще говоря, flaky-тест указывает на то, что тест основан на некотором состоянии
системы, которое не контролируется должным образом - тестовое окружение недостаточно
изолировано. Тесты более высокого уровня с большей вероятностью будут flaky,
поскольку они полагаются на большее количество состояний.

Flaky-тесты иногда появляются при параллельном запуске набора тестов
(например, при использовании pytest-xdist). Это может указывать на то, что тест
зависит от порядка тестирования.

- Возможно, другой тест не может очистить за собой и оставляет данные, что приводит
  к сбою flaky-теста.
- flaky-тест основан на данных из предыдущего теста, который не очищается после
  себя, и при параллельном запуске этот предыдущий тест не всегда присутствует.
- Тесты, которые изменяют глобальное состояние, обычно не могут выполняться параллельно.


Чрезмерно строгое утверждение
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чрезмерно строгие утверждения могут вызвать проблемы при сравнением чисел с плавающей точкой,
а также проблемы с синхронизацией.
Подробнее здесь `pytest.approx <https://docs.pytest.org/en/stable/reference.html#pytest-approx>`_.

Возможности Pytest
^^^^^^^^^^^^^^^^^^^^

Параметр strict в Xfail
~~~~~~~~~~~~~~~~~~~~~~

:ref:`pytest.mark.xfail ref` с параметром ``strict=False`` можно использовать для маркировки
теста, чтобы его сбой не приводил к поломке всей сборки.
Это можно рассматривать как ручную изоляцию и довольно опасно использовать постоянно.

PYTEST_CURRENT_TEST
~~~~~~~~~~~~~~~~~~~

:envvar:`PYTEST_CURRENT_TEST` может быть полезно для выяснения, "какой тест застрял".
См. :ref:`pytest current test env` для большей информации.


Плагины
~~~~~~~~

Повторный запуск любых упавших тестов может смягчить негативные эффекты flaky-тестов,
дав им дополнительные шансы на прохождение, чтобы общая сборка не провалилась.
Это поддерживают несколько плагинов pytest:

* `flaky <https://github.com/box/flaky>`_
* `pytest-flakefinder <https://github.com/dropbox/pytest-flakefinder>`_ - `blog post <https://blogs.dropbox.com/tech/2016/03/open-sourcing-pytest-tools/>`_
* `pytest-rerunfailures <https://github.com/pytest-dev/pytest-rerunfailures>`_
* `pytest-replay <https://github.com/ESSS/pytest-replay>`_:этот плагин помогает воспроизводить локальные сбои или flaky-тесты, наблюдаемые во время выполнения CI.

Плагины для преднамеренной рандомизации тестов могут помочь выявить тесты с проблемами состояния:

* `pytest-random-order <https://github.com/jbasko/pytest-random-order>`_
* `pytest-randomly <https://github.com/pytest-dev/pytest-randomly>`_


Другие рекомендации
^^^^^^^^^^^^^^^^^^^^^^^^

Разделение тестовых наборов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Обычно можно разделить один набор тестов на два, например модульные и интеграционные,
и использовать только набор модульных тестов в качестве элемента CI.
Это также помогает контролировать время сборки, поскольку тесты высокого уровня
обычно выполняются медленнее. Однако это означает, что становится возможным
объединить(merge) код, нарушающий сборку, поэтому требуется повышенная бдительность для
мониторинга результатов интеграционных тестов.


Скриншот/видео при сбое
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для тестов пользовательского интерфейса(UI) важно знание того, в каком состоянии был
пользовательский интерфейс, когда тест не прошел. ``pytest-splinter`` может
использоваться с такими плагинами, как ``pytest-bdd``, и может `сохранять снимок экрана при падении теста <https://pytest-splinter.readthedocs.io/en/latest/#automatic-screenshots-on-test-failure>`_,
что может помочь изолировать причину.


Удаление или изменение теста
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Возможно тест можно удалить, если функциональность покрывается другими тестами.
В противном случае, может быть существует возможность переписать его на более низком уровне,
что устранит flaky или сделает его источник более очевидным.

Карантин
~~~~~~~~~~

Mark Lapierre обсуждает `Плюсы и минусы карантинных тестов <https://dev.to/mlapierre/pros-and-cons-of-quarantined-tests-2emj>`_ в сообщении от 2018.



Инструменты CI, которые перезапускаются в случае сбоя
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Azure Pipelines (инструмент Azure cloud для CI/CD, ранее Visual Studio Team Services или VSTS)
имеет особенность
`выявлять flaky-тесты <https://docs.microsoft.com/en-us/azure/devops/release-notes/2017/dec-11-vsts#identify-flaky-tests>`_
и повторно запустить неудавшиеся тесты.


Исследования
^^^^^^^^^^^^^

Это ограниченный список, пожалуйста, отправьте issue или pull request, чтобы расширить его!

* Gao, Zebao, Yalan Liang, Myra B. Cohen, Atif M. Memon, and Zhen Wang. "Making system user interactive tests repeatable: When and what should we control?." In *Software Engineering (ICSE), 2015 IEEE/ACM 37th IEEE International Conference on*, vol. 1, pp. 55-65. IEEE, 2015.  `PDF <http://www.cs.umd.edu/~atif/pubs/gao-icse15.pdf>`__
* Palomba, Fabio, and Andy Zaidman. "Does refactoring of test smells induce fixing flaky tests?." In *Software Maintenance and Evolution (ICSME), 2017 IEEE International Conference on*, pp. 1-12. IEEE, 2017. `PDF in Google Drive <https://drive.google.com/file/d/10HdcCQiuQVgW3yYUJD-TSTq1NbYEprl0/view>`__
*  Bell, Jonathan, Owolabi Legunsen, Michael Hilton, Lamyaa Eloussi, Tifany Yung, and Darko Marinov. "DeFlaker: Automatically detecting flaky tests." In *Proceedings of the 2018 International Conference on Software Engineering*. 2018. `PDF <https://www.jonbell.net/icse18-deflaker.pdf>`__


Источники
^^^^^^^^^

* `Eradicating Non-Determinism in Tests <https://martinfowler.com/articles/nonDeterminism.html>`_ by Martin Fowler, 2011
* `No more flaky tests on the Go team <https://www.thoughtworks.com/insights/blog/no-more-flaky-tests-go-team>`_ by Pavan Sudarshan, 2012
* `The Build That Cried Broken: Building Trust in your Continuous Integration Tests <https://www.youtube.com/embed/VotJqV4n8ig>`_ talk (video) by `Angie Jones <http://angiejones.tech/>`_ at SeleniumConf Austin 2017
* `Test and Code Podcast: Flaky Tests and How to Deal with Them <https://testandcode.com/50>`_ by Brian Okken and Anthony Shaw, 2018
* Microsoft:

  * `How we approach testing VSTS to enable continuous delivery <https://blogs.msdn.microsoft.com/bharry/2017/06/28/testing-in-a-cloud-delivery-cadence/>`_ by Brian Harry MS, 2017
  * `Eliminating Flaky Tests <https://docs.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/eliminating-flaky-tests>`_ blog and talk (video) by Munil Shah, 2017

* Google:

  * `Flaky Tests at Google and How We Mitigate Them <https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html>`_ by John Micco, 2016
  * `Where do Google's flaky tests come from? <https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html>`_  by Jeff Listfield, 2017
