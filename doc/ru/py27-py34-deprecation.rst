Поддержка Python 2.7 и 3.4
===========================

Поддержка нескольких версий Python требует от сопровождающих проект с открытым исходным кодом больших усилий, поскольку
это дополнительные затраты на поддержание совместимости кода между всеми версиями, в то время как поддержание
функций, которые стали возможны только в новых версиях Python.

В случае с Python 2 и 3 разница между языками делает это еще более заметным,
поскольку многие новые возможности Python 3 не могут быть использованы в кодовой базе, совместимой с Python 2/3.

Python 2.7 EOL был достигнут `в 2020 году <https://legacy.python.org/dev/peps/pep-0373/#id4>`__, и
последний релиз был выпущен в апреле 2020 года.

Python 3.4 EOL достигнут `в 2019 году <https://www.python.org/dev/peps/pep-0429/#release-schedule>`__, последний релиз был выпущен в марте 2019 года.

По этим причинам в июне 2019 года было решено, что версия **pytest 4.6** будет последней, поддерживающей Python 2.7 и 3.4.

Что это значит для обычных пользователей
---------------------------------

Благодаря опции `python_requires`_ в setuptools,
пользователи Python 2.7 и Python 3.4, использующие современную версию pip
будут автоматически устанавливать последнюю версию pytest 4.6.X, даже если 5.0 или более поздние версии
доступны на PyPI.

Пользователи должны убедиться, что они используют последние версии pip и setuptools, чтобы это работало.

Сопровождение версий 4.6.X
-----------------------------

До января 2020 года основная команда pytest переносила многие исправления из основного релиза в ветку
``4.6.x``, при этом в течение года было выпущено несколько релизов 4.6.X.

С этого момента основная команда больше не будет **активно переносить исправления**, но ветка ``4.6.x`` будет существовать для сообщества,
чтобы сообщество могло вносить исправления.

Основная команда будет рада принимать эти исправления и выпускать новые релизы 4.6.X **до середины 2020 года**.
(но рассматривайте эту дату как ориентировочную, после этой даты команда все еще может решить выпустить новые релизы
для критических ошибок).

.. _`python_requires`: https://packaging.python.org/guides/distributing-packages-using-setuptools/#python-requires

Технические аспекты
~~~~~~~~~~~~~~~~~

(Этот раздел является расшифровкой из `#5275 <https://github.com/pytest-dev/pytest/issues/5275>`__).

В этом разделе мы описываем технические аспекты плана поддержки Python 2.7 и 3.4.

Что войдет в релизы 4.6.X
+++++++++++++++++++++++++++++

Новые релизы 4.6.X будут содержать только исправления ошибок.

Когда появятся релизы 4.6.X
+++++++++++++++++++++++++++++++

Новые релизы 4.6.X будут выпущены после того, как будет исправлено несколько ошибок, или если пройдет
несколько недель (например, одна ошибка будет исправлена через месяц после выхода последней версии 4.6.X).

Здесь нет жестких правил, только приблизительные.

Кто будет заниматься исправлением ошибок
+++++++++++++++++++++++++++++++++++++++++++

Мы, сопровождающие ядра, ожидаем, что люди, все еще использующие Python 2.7/3.4 и обнаружившие
ошибки, примут участие и предоставят исправления и/или перенесут исправления из активных веток.

Мы будем рады помочь пользователям, заинтересованным в этом, поэтому, пожалуйста, не стесняйтесь спрашивать.

**Обратный перенос изменений в 4.6**.

Пожалуйста, следуйте этим инструкциям:

#. ``git fetch --all --prune``

#. ``git checkout origin/4.6.x -b backport-XXXX`` # используйте PR-номер здесь

#. Найдите коммит слияния в PR, в сообщении *merged*, например:

    nicoddemus объединил коммит 0f8b462 в pytest-dev:features

#. ``git cherry-pick -m1 REVISION`` # используйте ревизию, которую вы нашли выше (``0f8b462``).

#. Откройте PR, нацеленный на ``4.6.x``:

   * Припишите к сообщению префикс ``[4.6]``, чтобы это был очевидный бэкпорт.
   * Удалите тело PR, обычно оно содержит дублирующее сообщение о фиксации.

**Предоставление новых PR для 4.6**

Свежие запросы на исправление до ``4.6.x`` будут приниматься при условии, что
эквивалентный код в активных ветках не содержит этой ошибки (например, ошибка специфична
только для Python 2).

Исправления ошибок, которые также имеют место в основной версии, должны быть сначала исправлены
там, а затем переносить в соответствии с инструкциями выше.
