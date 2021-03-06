========
Overview
========

.. start-badges

.. list-table::
    :stub-columns: 1

    * - docs
      - |docs|
    * - tests
      - | |travis|
        | |codecov|
    * - package
      - |version| |wheel| |supported-versions| |supported-implementations|


.. |docs| image:: https://img.shields.io/readthedocs/pip.svg?style=flat-square
    :alt: Read the Docs
    :target: https://readthedocs.org/projects/pyfsm

.. |travis| image:: https://img.shields.io/travis/Woile/pyfsm.svg?style=flat-square
    :alt: Travis-CI Build Status
    :target: https://travis-ci.org/Woile/pyfsm

.. |codecov| image:: https://img.shields.io/codecov/c/github/Woile/pyfsm.svg?style=flat-square
    :alt: Coverage Status
    :target: https://codecov.io/github/Woile/pyfsm

.. |version| image:: https://img.shields.io/pypi/v/fsmpy.svg?style=flat-square
    :alt: PyPI Package latest release
    :target: https://pypi.python.org/pypi/fsmpy

.. |wheel| image:: https://img.shields.io/pypi/wheel/fsmpy.svg?style=flat-square
    :alt: PyPI Wheel
    :target: https://pypi.python.org/pypi/fsmpy

.. |supported-versions| image:: https://img.shields.io/pypi/pyversions/fsmpy.svg?style=flat-square
    :alt: Supported versions
    :target: https://pypi.python.org/pypi/fsmpy

.. |supported-implementations| image:: https://img.shields.io/pypi/implementation/fsmpy.svg?style=flat-square
    :alt: Supported implementations
    :target: https://pypi.python.org/pypi/fsmpy


.. end-badges

Minimal state machine

* Free software: BSD license


.. code-block:: python

    import fsm

    class MyTasks(fsm.FiniteStateMachineMixin):
        """An example to test the state machine.

        Contains transitions to everywhere, nowhere and specific states.
        """

        state_machine = {
            'created': '__all__',
            'pending': ('running',),
            'running': ('success', 'failed'),
            'success': None,
            'failed': ('retry',),
            'retry': ('pending', 'retry'),
        }

        def __init__(self, state):
            """Initialize setting a state."""
            self.state = state

        def on_before_pending(self):
            print("I'm going to a pending state")

::

    In [4]: m = MyTasks(state='created')

    In [5]: m.change_state('pending')
    I'm going to a pending state
    Out[5]: 'pending'


::

    In [6]: m.change_state('failed')  # Let's try to transition to an invalid state
    ---------------------------------------------------------------------------
    InvalidTransition                         Traceback (most recent call last)
    <ipython-input-6-71d2461eee74> in <module>()
    ----> 1 m.change_state('failed')

    ~/pyfsm/src/fsm/fsm.py in change_state(self, next_state, **kwargs)
        90             msg = "The transition from {0} to {1} is not valid".format(previous_state,
        91                                                                        next_state)
    ---> 92             raise InvalidTransition(msg)
        93
        94         name = 'pre_{0}'.format(next_state)

    InvalidTransition: The transition from pending to failed is not valid

.. contents::
    :depth: 2


Installation
============

::

    pip install fsmpy


Usage
======

1. Define in a class the :code:`state_machine`
2. Initialize :code:`state`, either with a value, using :code:`__init__` or as a django field
3. Add hooks:

+------------------------+-------------------------------------------------------------------------------------------------------------+
| Method                 | Description                                                                                                 |
+------------------------+-------------------------------------------------------------------------------------------------------------+
| on_before_change_state | Before transitioning to the state                                                                           |
+------------------------+-------------------------------------------------------------------------------------------------------------+
| on_change_state        | After transitioning to the state, if no failure, runs for every state                                       |
+------------------------+-------------------------------------------------------------------------------------------------------------+
| pre_<state_name>       | Runs before a particular state, where :code:`state_name` is the specified name in the :code:`state_machine` |
+------------------------+-------------------------------------------------------------------------------------------------------------+
| post_<state_name>      | Runs after a particular state, where :code:`state_name` is the specified name in the :code:`state_machine`  |
+------------------------+-------------------------------------------------------------------------------------------------------------+

This hooks will receive any extra argument given to :code:`change_state`


E.g:

Running :code:`m.change_state('pending', name='john')` will trigger :code:`pre_pending(name='john')`

Django integration
==================

.. code-block:: python

    import fsm
    from django.db import models


    class MyModel(models.Model, fsm.FiniteStateMachineMixin):
        """An example to test the state machine.

        Contains transitions to everywhere, nowhere and specific states.
        """

        CHOICES = (
            ('created', 'CREATED'),
            ('pending', 'PENDING'),
            ('running', 'RUNNING'),
            ('success', 'SUCCESS'),
            ('failed', 'FAILED'),
            ('retry', 'RETRY'),
        )

        state_machine = {
            'created': '__all__',
            'pending': ('running',),
            'running': ('success', 'failed'),
            'success': None,
            'failed': ('retry',),
            'retry': ('pending', 'retry'),
        }

        state = models.CharField(max_length=30, choices=CHOICES, default='created')

        def on_change_state(self, previous_state, next_state, **kwargs):
            self.save()

Django Rest Framework
---------------------

If you are using :code:`serializers`, they usually perform the :code:`save`, so saving inside :code:`on_change_state` is not necessary.

One simple solution is to do this:

.. code-block:: python

    class MySerializer(serializers.ModelSerializer):

        def update(self, instance, validated_data):
            new_state = validated_data.get('state', instance.state)
            try:
                instance.change_state(new_state)
            except fsm.InvalidTransition:
                raise serializers.ValidationError("Invalid transition")
            instance = super().update(instance, validated_data)
            return instance


Documentation
=============

https://pyfsm.readthedocs.org/

Development
===========

To run the tests run::

    tox

Note, to combine the coverage data from all the tox environments run:

.. list-table::
    :widths: 10 90
    :stub-columns: 1

    - - Windows
      - ::

            set PYTEST_ADDOPTS=--cov-append
            tox

    - - Other
      - ::

            PYTEST_ADDOPTS=--cov-append tox
