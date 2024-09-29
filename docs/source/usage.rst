Usage
=====

.. _installation:

Installation
------------

1. Install `SUMO <https://sumo.dlr.de/docs/Installing/index.html>`_  (we have tested IntersectionZoo with SUMO 1.12), and make sure that the environment variable ``SUMO_HOME`` is set as per the SUMO installation instructions.
2. Create and activate `conda <https://docs.conda.io/en/latest>`_ environment or `venv <https://docs.python.org/3/library/venv.html>`_ with python 3.10. 
3. We use `Weights and Biases <https://wandb.ai/>`` for logging. Create a free account on `Weights and Biases <https://wandb.ai/>`_ and `confgire your terminal <https://docs.wandb.ai/quickstart>`.
4. Clone the IntersectionZoo `repository <https://github.com/mit-wu-lab/IntersectionZoo/>`_ and from the root of the repo, install the dependencies using the following command.
5. By defualt we use ray[rllib]==2.11.0 for training. Some popular RL algorithms like DDPG have been moved to `RLlib contrib <https://github.com/ray-project/ray/tree/master/rllib_contrib>`. If you want to use these algorithms, please follow the instructions listed `here <https://github.com/ray-project/ray/tree/master/rllib_contrib>` for individual algorithms.

.. code-block:: console

   pip install -r requirements.txt

Runnig IntersectionZoo
-----------------------

To undertand how an enviroment simulations work, we provide a simple simulation loop. To run and visualize the simualtion, use the following command from the root of IntersectionZoo repository. Note that this will not start any training but will only run a single simulation with pre-define constant accelerarion for each vehicle.

.. code-block:: console

   python code/env_demo.py

Run the following command to train an example multi-task PPO agent on Salt Lake City intersections. 
Check `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section for more details on training configurations inclduing how to change the intersection dataset.

.. code-block:: console

   python code/ppo_training.py --dir <exp_dir>

Here, ``<exp_dir>`` is where all the training artifacts will be stored and/or the checkpoints will be retrieved (to evaluate or restart the training).
It will be created if it doesn't exist.

To override the default configurations, pass a python dictionary with the arguments to override the config in ppo_training.py.

.. code-block:: console

   python code/ppo_training.py --dir <exp_dir> --kwargs <python dictionary with arguments to override the config in code/ppo_training.py>


Similarly, to evaluate a trained agent on Salt Lake City intersections, run the following command. ``<exp_dir>`` should point to a directory where all training artifacts are stored for the checkpoints to be retrieved.
Check `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section for more details on evaluation configurations inclduing how to change the intersection dataset.

.. code-block:: console

   python code/policy_evaluation.py --dir <exp_dir>


IntersectionZoo uses SUMO microscopic traffic simualator for simualtions. Run the following command to visulize the trained agents on a given intersection dataset with SUMO GUI. 
Check `Tutorials <https://intersectionzoo-docs.readthedocs.io/en/latest/tutorial.html>`_ section for more details on evaluation configurations inclduing how to change the intersection dataset.

.. code-block:: console

   python code/visualize.py --dir <exp_dir>