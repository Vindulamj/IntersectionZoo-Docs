.. _intersectionzoo_architecture:

IntersectionZoo 
===============

Architecture
------------

IntersectionZoo builds more than one million data-driven traffic scenarios around 16,334 signalized intersections in 10 major cities across United States. The figure below shows the 
10 cities and the number of intersections in each city.

.. image:: image/cities.png
    :alt: Cities
    :scale: 35%
    :align: center


In building IntersectionZoo, three main logical layers are used to abstract functionality as shown in the following figure.

.. image:: image/architecture.png
    :alt: IntersectionZoo Architecture
    :scale: 42%
    :align: center

\

In the **traffic scenario modeling layer**, we first build data-driven simulation environments of signalized intersections and then use them to build traffic scenarios at those
intersections. Concretely, an intersection is first defined by factors such as lane lengths, lane counts, road grades, turn lane configuration, and speed limit of each approach. 
Then, vehicle type, age, and fuel type distributions are used with appropriate traffic flow rates and human driven vehicle behaviors to define a realistic traffic flow. 
Each intersection is then used to define traffic scenarios by further assigning representative atmospheric temperature and humidity values based on the season. 
Further scenario variations can be achieved by changing the eco-driving adoption level (0%-100%). 

The factors we consider and data sources we use for modeling each factor is given in the following table. 

.. list-table:: Eco-driving factors, data sources, and notes
   :widths: 20 40 40
   :header-rows: 1

   * - Factor
     - Data Source
     - Notes
   * - Intersection topology
     - Open Street Maps
     - Followed the guidelines from `Qu et al. (2023) <https://arxiv.org/abs/2405.13480>`_ to extract intersection topology from Open Street Maps.
   * - Speed limits
     - Open Street Maps
     - 
   * - Road grades
     - US geological surveys
     - 
   * - Vehicle inflow
     - Annual Average Daily Traffic (AADT) data
     - Collected from each city transportation department open data portal
   * - Driving hour (peak/off-peak)
     - Standard conversion rates
     - Taken from `Precision Traffic and Safety <https://www.precisiontrafficsafety.com/solutions/traffic-studies/>`_. 
   * - Vehicle arrival process
     - Realistic vehicle arrival process based on nearby intersections
     - For every intersection, a set of default nearby intersections \
       are added as a way of achieving realistic vehicle arrival \ 
       processes subjected to nearby traffic signal dynamics.
   * - Traffic Signal Timing
     - Optimal traffic signal plan
     - Exhasutively search through `Fixed Time <https://nacto.org/publication/urban-street-design-guide/intersection-design-elements/traffic-signals/fixed-vs-actuated-signalization/>`_ traffic signal plans \
       to find the optimal plan for each intersection.
   * - Vehicle age distribution
     - MOVES database
     - Taken from open database of `MOVES <https://www.epa.gov/moves>`_.
   * - Fuel type distribution
     - MOVES database
     - Taken from open database of `MOVES <https://www.epa.gov/moves>`_.
   * - Vehicle type distribution
     - MOVES database
     - Taken from open database of `MOVES <https://www.epa.gov/moves>`_.
   * - Temperature and humidity
     - US National Centers for Environmental Information
     - All processes data are available here `MOVES <https://docs.google.com/spreadsheets/d/1IxSaxkgkE9tA21u5CtSUVWJPa15QfLHT/edit?usp=sharing&ouid=111770128718724110720&rtpof=true&sd=true>`_ \
       for each city under each season (Fall, Spring, Summer and Winter) \
       and under different weather conditions (sunny, rain, snow).
   * - Human driver models
     - Intelligent Driver Model (IDM) callibrated with real-world data.
     - Calibration method is adopted from `Zhang et al. (2022) <https://arxiv.org/abs/2210.03571>`_ and \
       the data is taken from `CitySim dataset <https://github.com/UCF-SST-Lab/UCF-SST-CitySim1-Dataset>`_ \
       released under Apacahe 2.0 License.
   * - Eco-driving adoption level
     - Any user prefered value between 0%-100% 
     -
   * - Engine type
     - Internal combusion engines and electric engines as user prefered.
     - 

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