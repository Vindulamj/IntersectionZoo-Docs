.. _rllib_integration:

Rllib Integration
=================

IntersectionZoo is by default integrated with `RLlib <https://docs.ray.io/en/latest/rllib/index.html>`_, a scalable reinforcement learning library.

IntersectionZooEnv (`🔗 <https://github.com/mit-wu-lab/IntersectionZoo/blob/main/code/env/environment.py>`_)
------------------

The default context-MDP used in IntersectionZoo is defined in the ``IntersectionZooEnv``. 
This class inheritates from ``MultiAgentEnv`` and ``TaskSettableEnv`` of RLLib. ``MultiAgentEnv`` is used to handle multiple agents in the environment and ``TaskSettableEnv`` is used to handle multiple tasks (traffic scenarios).
You can learn more about ``MultiAgentEnv`` `here <https://docs.ray.io/en/latest/rllib/package_ref/env/multi_agent_env.html#rllib-env-multi-agent-env-multiagentenv>`_ and 
``TaskSettableEnv`` `here <https://docs.ray.io/en/latest/rllib/rllib-advanced-api.html#curriculum-learning>`_.

Multi-task / Curriculum / Transfer Learning
-------------------------------------------

Common approaches to learn generalizable policies is to use multi-task learning, curriculum learning or transfer learning. By inheriting from ``TaskSettableEnv``, IntersctionZoo provides flexible support for these approaches.
During training the ``curriculum_fn`` as defined `here <https://docs.ray.io/en/latest/rllib/rllib-advanced-api.html#curriculum-learning>`_ allows users to choose new environment at each rollout. For a tutorial on how to use it, 
please refer to the `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section.

Task Definitions (`🔗 <https://github.com/mit-wu-lab/IntersectionZoo/blob/main/code/env/task_context.py>`_)
^^^^^^^^^^^^^^^^

IntersectionZoo defines tasks using ``TaskContext`` objects. They can either represent a single task or multiple tasks. If multiple tasks are defined, 
one can either sample a single one uniformly at random with ``.sample()`` or list all possible single tasks exhaustively with ``.list_tasks()``.

The intersection can either be real-world intersection from <TODO add cities> (TODO cite paper). They can be found in the dataset folder.
They can also be syntetic intersections, built to simplify the training process or evalute agents on very specific situations.

All ``TaskContext`` define:

- ``single_approach``, whether vehicles flow only through 1 of the intersection's approach or all of them.
- ``penetration_rate``, the proportion of vehicles controlled by the RL policy.
- ``temperature_humidity``, the temperature and humidity used by the fuel and emission model.
- ``electric_or_regular``, the vehicle's technology, also for the fuel and emission model.

``PathTaskContext`` define real-world intersections with:

- ``path``, the path to a collection of intersections folder or a single one
- ``aadt_conversion_factor`` (optional), the conversion factor to use to convert daily averages to hourly inflow rates

``NetGenTaskContext`` define synthetic intersections with:

- ``base_id``: Basic shape of the intersection, includes number of lanes and TL phases.
- ``inflow``: Inflow in vehicles per hour, used as is (no more factor for single lane scenario).
- ``green_phase``: Duration of the main green phase
- ``red_phase``: Duration of the main red phase (not including amber)
- ``lane_length``: Lane length in meters
- ``speed_limit``: Speed limit in m/s
- ``offset``: Offset between ghost cells (modelling incoming traffic) TL programs and the main intersection TL program.

**Training and evaluation**

The same policies can technically be used on any kind of intersection, thus it is possible for example to:
- Train on synthetic intersections and evaluate on a real-world one
- Train on a certain city and evaluate on another one
- train on a set of synthetic intersections and evaluate on a distinct one (for example going from short to long cycle time)


Environment config
------------------

The main config settings are:

- Simulation process
 - ``sim_step_duration``: time duration of a simulation step, in seconds
 - ``warmup_steps``: duration (in simulation steps) of the warmup period at the beginning of the simulation during which vehicles are uncontrolled
 - ``task_context``: TaskContext used to initialize the environement. Can be changed later.
 - ``simulation_duration``: How long (in seconds) the simulation should be before finishing. 
- Others
    - ``visualize_sumo``: whether to use the SUMO gui
    - ``control_lane_change``: whether the agents contols also when the vehicles change lane. It is disabled in all teh examples.

Metrics
-------

To evaluate the performance of the agents, multiple metrics are logged by the environment.
At the end of each simulation, the metrics are sent to RLlib using an RLlib callback, allowing them to be collected and aggregated by RLlib.

At the beginning pf the episode a warmup period can be added. During that period metrics vehicles are not controlled and metrics not logged,
vehicles present during warmup are not counted at all in the metrics, even for their actions after the warmup ended.

Weights and Biases can be also be used to log the metrics out of RLlib.

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