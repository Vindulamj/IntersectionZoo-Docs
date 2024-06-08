.. _rllib_integration:

Rllib Integration
=================

The main class, ``IntersectionZooEnvironment``, is a subclass of ``rllib.env.MultiAgentEnv`` (multi-agent environment) and ``rllib.env.TaskSettableEnv`` (multi-task environment). When using the environment with an RLlib policy, the following minimal configuration should be passed:

.. code-block:: python

    .environment(
        env=IntersectionZooEnvironment,
        env_config={"intersectionzoo_env_config": env_conf},
        env_task_fn=curriculum_fn,
    )

- The ``env_config`` is mandatory. Please refer to the ``config.py`` file for the minimum list of attributes to pass.
- The ``env_task_fn`` is necessary to change the task during training. You can provide a custom function to handle task changes.

The metrics are also fed to RLlib, for this you need to pass the following configuration:

.. code-block:: python

    .callbacks(MetricsCallback)

The metrics can then be visualized in TensorBoard, or fed back to other systems like Weights and Biases.

The policy can easily be evaluated, as showed in ``policy_evalution.py``. Please note that the eval configuration used is the one passed during training.