.. _extensions:

Extensions
==========

Customized metrics and MDP 
--------------------------

**The metrics** are reward are computed in the `IntersectionZooEnv` class.

Most metrics should be added first in ``_collect_metrics()``, which is called at each step, and then in ``get_metrics()`` which is called at the end to aggregate the metrics over the episode.

For example the number of hard brakes would be computed by:
- first collecting at each step if a hard brake is occuring during the step in ``_collect_metrics()``, and store in ``self.vehicle_metrics[v_id]["hard_brake"]``
- then at the end of the episode, the number of hard brakes would be computed in ``get_metrics()`` by summing over all vehicles the number of hard brakes, and store in ``aggregated_metrics``, which is the variable containing all the metrics at the end of the episode.

The simulation data can either be found in the ``TrafficState`` object, which contains a lot of information about the current state of the simulation, or can directly queried from the SUMO simulation using the `TraCI API <https://sumo.dlr.de/docs/TraCI.html>`_ via the ``traci`` attribute.
However please note that the TraCI API can be slow, so relying on the ``TrafficState`` should be preferred.

**The reward** is computed in the ``_get_reward()`` method, which is called at each step, the TrafficState and Traci can be used as well.

**The Observation** can be modified by:
1. Add the field in the ``observation_space`` attribute of the enviroment (using ``Gymnasium`` spaces).
2. Adding new field in the observation in the ``_get_vehicle_obs()`` method.


Custom policies
---------------

The easiest way to develop customized policies is by using RLlib wrappers, see their `documentation <https://docs.ray.io/en/latest/rllib/rllib-concepts.html>`_.
RLlib's documentation also describe in good details how the Multi agent enviroment interface, and is thus a good ressource to develop custom policies for IntersectionZooEnviroment from scratch.