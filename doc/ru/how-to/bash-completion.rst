
.. _bash_completion:

Настройка авто-дополнения в bash
==================================

При использовании bash в качестве оболочки, ``pytest`` может использовать ``argcomplete``
(https://argcomplete.readthedocs.io/) для авто-дополнения.
Для этого ``argcomplete`` должен быть установлен **и** включен.

Установите ``argcomplete``, используя:

.. code-block:: bash

    sudo pip install 'argcomplete>=0.5.7'

Для глобальной активации всех приложений ``python`` с поддержкой ``argcomplete`` запустите:

.. code-block:: bash

    sudo activate-global-python-argcomplete

Для постоянной(но не глобальной) активации, используйте:

.. code-block:: bash

    register-python-argcomplete pytest >> ~/.bashrc

Для разовой активации ``argcomplete`` только для ``pytest``, используйте:

.. code-block:: bash

    eval "$(register-python-argcomplete pytest)"
