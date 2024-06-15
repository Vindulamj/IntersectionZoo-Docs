.. _main-concepts:

Main concepts
=============

.. _rllib:

Cooperative Eco-driving
-----------------------

In a mixed traffic environment, the behavior of human-driven vehicles (HDVs) can be influenced by the behavior of controlled vehicles (CVs). Here, 
the controlled vehicles can be autonomous vehicles or even vehicles with adavanced driver assistance systems (ADAS).
In the following figure, we show an example of cooperative eco-driving at signalized intersections where CVs are controlled to 
minimize the total exhaust emissions of the fleet (both CVs and HDVs) while minimizing the impact on the travel time of each vehicle. 
While CVs are controllable by some defined eco-driving strategy, HDVs are driven by humans and can not be controlled. However, CVs implicitly control HDVs 
through car-following dynamics and by forming locally cooperative teams for better system control 
(by controlling the oppotunities of HDVs to overake CVs). In IntersectionZoo, we focus on the problem of cooperative eco-driving at signalized intersections by 
controlling the longitudinal accelerations of CVs. Lane changing of both CVs and HDVs are controlled by rule-based defualt models of SUMO traffic simulator. 

.. image:: image/eco-drive.png
    :alt: Eco-driving Example
    :scale: 30%
    :align: center

Concreately, the default objective of cooperative eco-driving at individual signalized intersections is to minimize the total exhaust emissions of a 
fleet of vehicles consisting of both Controlled Vehicles (CVs) and Human Driven Vehicles (HDVs) 
while having a minimal impact on individual travel time of vehicles. At a given time :math:`t`, the number of CVs is :math:`k_{CV}^t`, 
and HDVs is :math:`k_{HDV}^t` such that :math:`k_{CV}^t + k_{HDV}^t = n^t` where :math:`n^t` is the total number of vehicles in the fleet. 
Then, we control the longitudinal accelerations of all CVs decentrally using a learned policy to optimize.

.. math::

   \min J = \sum_{i=1}^{n} \sum_{t=0}^{T_i} E\left(a_i(t), v_i(t)\right) + \lambda T_i.


Here, :math:`T_i` denotes the travel time of vehicle :math:`i` and time :math:`t` is a discretized time with increments of :math:`\delta` 
(we use 0.5 seconds).The vehicular exhaust emission function is denoted by :math:`E(\cdot)`, which takes speed :math:`v_i(t)` and acceleration :math:`a_i(t)` 
of vehicle :math:`i` at time :math:`t` and outputs a vehicular emission amount (usually the amount of carbon dioxide). :math:`\lambda` 
is the trade-off hyperparameter. We seek to optimize :math:`J` subject to hard constraints of ensuring vehicle safety, 
connectivity via vehicle-to-vehicle and vehicle-to-traffic signal communication, and soft constraints of vehicle kinematics, 
control realism, traffic safety at the fleet level (e.g., minimum time to collision across all vehicles), and passenger comfort.

Cooperative Eco-driving at City Scale
-------------------------------------

Optimizing eco-driving across a full-traffic network is ideal but is impractical in large cities like Los Angeles, with nearly 5000 signalized intersections, 
and remains an open optimization challenge. A common approach is to decompose the network into individual intersections for separate optimizations while 
regulating intersection throughput to prevent traffic spill-back due to possible increased throughput. Therefore, it is often assumed that vehicle 
flow is not at saturation, allowing for reasonable throughput improvements. By default, we adopt the same modeling assumption in intersectionZoo. However, prefered 
users can choose to regulate throughput according to their methods or estimate potential for improvement in throughput at each intersection. 

Then, given a city-scale traffic network, we can model each intersection separately. However, each signalized intersection yeilds multiple 
traffic scenarios due to the factors such as weather, time of day, and traffic demand and eco-driving adoption level. 
Therefore, we need to optimize the eco-driving policy for across all these scenarios. 


Consider a city comprised of traffic scenarios :math:`\Phi` where :math:`|\Phi|` is large (e.g., millions of traffic scenarios). 
Let :math:`\pi^e :=  (\pi^e_{\phi})_{\phi \in \Phi}` denote the set of eco-driving control laws. 
Similarly, let :math:`\pi^b :=  (\pi^b_{\phi})_{\phi \in \Phi}` denote the baseline (status quo driving). 
Let :math:`f(\pi, \phi)` denote a function that captures the CO\ :sub:`2` emission for the scenario :math:`\phi` under the control law :math:`\pi`. 
We then define the *regional eco-driving effectiveness* as,

.. math::

   E_{\Phi}(\pi^e, \pi^b) := 1 - \frac{\mathbb{E}_{\phi \in \Phi} [f(\pi^e_{\phi}, \phi)]}{\mathbb{E}_{\phi \in \Phi} [f(\pi^b_{\phi}, \phi)]}

We thus seek to solve the *regional eco-driving problem* for the control laws :math:`\pi^e` such that,

Find

.. math::

   \pi^e = \arg\max_{\pi} E_{\Phi}(\pi, \pi^b)

subject to,

.. math::

   n_{\phi}^{e} \geq n_{\phi}^{b} \quad \forall \; \phi \in \Phi

where :math:`n_{\phi}^{e}` and :math:`n_{\phi}^{b}` denote the intersection throughput for scenario :math:`\phi` under the control laws of :math:`\pi^e_{\phi}` and :math:`\pi^b_{\phi}`, respectively.


RLlib
-----

The environment defined in this package ``IntersectionZoo`` is defined in the ``IntersectionZooEnviroment`` class. 
To make it easy to start developping agents for it, it uses the RLlib interfaces (inheritates from ``MultiAgentEnv``). 
You can learn more about RLlib `here <https://docs.ray.io/en/latest/rllib/index.html>`_ and Multi-agents env `here <https://docs.ray.io/en/latest/rllib/package_ref/env/multi_agent_env.html>`_.

Multi-task
----------

Another important functionnailty that the benchmarking suite offers is that the simulation can be based on multiple intersection shapes.

The multi-task is managed by RLlib (documented `there <https://docs.ray.io/en/latest/rllib/rllib-advanced-api.html#curriculum-learning>`_).
The environment thus also inherits from ``TaskSettableEnv``. During training the ``curriculum_fn`` allows users to choose new environment at each rollout.

The tasks are defined by ``TaskContext`` objects. **They can either represent a single task or multiple of them**. If multiple, 
one can either sample a single one uniformly at random with ``.sample()`` or list all possible single tasks exhaustively with ``.list_tasks``.

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