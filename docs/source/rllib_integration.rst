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
one can either sample a single task uniformly at random with ``.sample()`` or list all possible single tasks exhaustively with ``.list_tasks()``.


The intersections used with IntersectionZoo can either be real-world intersections provided with IntersectionZoo. The intersection SUMO network file datasets of 10 cities can be found `here <https://drive.google.com/drive/folders/1y3W83MPfnt9mSFGbg8L9TLHTXElXvcHs>`_.
Once downloaded, they should be placed in the dataset folder. Once configured, further traffic scenario variations can be generated using ``PathTaskContext``.

``PathTaskContext`` define real-world intersections with:

- ``single_approach``: how to simulate the intersection. If the strings "A", "B", "C", "D" is used only a one incoming and outgoing approach of the intersection is simulated.
   if set to "True" each incoming and outgoing approach pair will be simulated seperately, and if "False" all of them at the same time woill be used. Note that if not all approaches are used, 
   unprotected left turns will not be simulated and the intersection will be simplified.
- ``penetration_rate``: the proportion of vehicles controlled by the learned policy (any value between 0 to 1).
- ``temperature_humidity``: a string indicating the temperature and humidity for the defined traffic scenario (format: temperature_humidity).
- ``electric_or_regular``: what type of setup to use in term of having electric vehicles vs internal combustion engine vehicles.
        Use keywords REGULAR for internal combustion engine vehicles, ELECTRIC for electric vehicles. 
- ``path``: path to the intersection dataset (this can also point to a single intersection file)
- ``aadt_conversion_factor``: the conversion factor to use to convert daily inflow rates to to hourly inflow rates. By default we recommend using 0.084 for peak hours and 0.055 for off-peak hours.

Further details of these parameters can be found in the class definition. For a tutorial on how to use ``PathTaskContext``, 
please refer to the `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section.

To use syntetically generated intersections, ``NetGenTaskContext`` can be used. However, we simulate each incoming and outgoing approach pair seperately.

``NetGenTaskContext`` define synthetic intersections with:

- ``base_id``: number of lanes in the approach and the number of relevant traffic signal phases put together. First digit is lane number, second is phase number. Only 11, 21, 31, 41, 22, 32, 42 supported. For example
    21 means 2 lanes and 1 phase (through traffic), 42 means 4 lanes and 2 phases (through and left turn traffic).
- ``inflow``: Inflow in vehicles per hour.
- ``green_phase``: Duration of the green phase of the traffic signal in seconds.
- ``red_phase``: Duration of the red phase of the traffic signal in seconds.
- ``lane_length``: Lane length in meters.
- ``speed_limit``: Speed limit in m/s.
- ``offset``: Offset between nearby intersections (modelling incoming traffic) traffic signal timing programs and the main intersection traffic signal timing program.

Further details of these parameters can be found in the class definition. For a tutorial on how to use ``NetGenTaskContext``, 
please refer to the `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section.

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