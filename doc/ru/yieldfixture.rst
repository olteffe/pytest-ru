:orphan:

.. _yieldfixture:

Функции "yield_fixture"
---------------------------------------------------------------





.. important::
    Начиная с pytest-3.0, фикстуры, использующие обычный декоратор ``fixture``, могут использовать оператор ``yield``
    для предоставления значений фикстуры и выполнения кода разрыва, точно так же, как ``yield_fixture``
    в предыдущих версиях.

    Маркировка функций как ``yield_fixture`` все еще поддерживается, но является устаревшей и не должна
    использовать в новом коде.
