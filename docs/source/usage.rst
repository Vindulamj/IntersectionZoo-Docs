Usage
=====

.. _installation:

Installation
------------

1. Install SUMO (ideally 1.12, but 1.13 seems to work as well), and make sure that the env var SUMO_HOME is set.
2. Create a conda env or venv with python 3.10
3. From that environemt install dependencies using following command:

.. code-block:: console

   pip install -r requirements.txt

Run Instructions
----------------

From the root of the repo and using the correct venv/ conda env, run the following command to train the agents. Check ``code/train.py`` for more options.

.. code-block:: console

   python code/train.py --dir <exp_dir>

.. code-block:: console

   python code/train.py --dir <exp_dir> --kwargs <python dict with arguments to override the config in main.py>

Where ``<exp_dir>`` is where all the training artifacts will be stored and/or the checkpoints will be retrieved (to evaluate or restart the training). 
It will be created if it doesn't exist.

Example:

.. code-block:: console

   python code/train.py --dir wd/test --kwargs "{'wandb_proj':'scenario-env'}" 

From the root of the repo and the correct venv/ conda env, run the following command to evaluate the trained agents on a given intersection dataset. 
Check ``code/evaluate.py`` for more options. Note that ``<dir>`` should point to the same directory used for training.

.. code-block:: console

   python code/evaluate.py --dir <exp_dir>

Visualization
-------------

From the root of the repo and the correct venv/ conda env, run the following command to visulize the trained agents on a given intersection dataset. 
Check ``code/visualize.py`` for more options. Note that ``<dir>`` should point to the same directory used for training.

.. code-block:: console

   python code/visualize.py --dir <exp_dir>

It is possible to visualize any SUMO simulation by setting the ``visualize_sumo`` config to True in the env config.
