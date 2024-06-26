Usage
=====

.. _installation:

Installation
------------

1. Install `SUMO <https://sumo.dlr.de/docs/Installing/index.html>`_  (we have tested IntersectionZoo with SUMO 1.12), and make sure that the environment variable ``SUMO_HOME`` is set as per the SUMO installation instructions.
2. Create and activate `conda <https://docs.conda.io/en/latest>`_ environment or `venv <https://docs.python.org/3/library/venv.html>`_ with python 3.10. 
3. Clone the IntersectionZoo `repository <https://github.com/mit-wu-lab/IntersectionZoo/>`_ and from the root of the repo, install the dependencies using the following command.

.. code-block:: console

   pip install -r requirements.txt

Runnig IntersectionZoo
-----------------------

From the root of IntersectionZoo repository, run the following command to train an example multi-task PPO agent on Salt Lake City intersections. 
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