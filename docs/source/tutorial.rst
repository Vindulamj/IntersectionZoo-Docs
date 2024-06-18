Tutorial
========

Multiple tutorials are provide in ``code/`` directory to train and evaluate agents on the IntersectionZoo environments. A list of given tutorial examples are provided below:

1. ``ppo_training.py``: Training a multi-task PPO agent on the IntersectionZoo environments.
2. ``ddpg_training.py``: Training a multi-task DDPG agent on the IntersectionZoo environments.
3. ``synthetic_ppo_training.py``: Training a multi-task PPO agent on procedurally generated environments.
4. ``synthetic_ddpg_training.py``: Training a multi-task DDPG agent on procedurally generated environments.
5. ``policy_evaluation.py``: Evaluating a trained policy on the IntersectionZoo environments.

These tutorials are designed for users to get familiar on how to use RLlib with the IntersectionZoo environment. Below, we provide a step-by-step guide on how to train 
a PPO agent on the IntersectionZoo environment following the tutorial script ``ppo_training.py``.

Training
--------

The first step is to define the tasks on which the agent will be trained. The tasks are defined in the ``PathTaskContext`` object. In the ``dir`` parameter, the path to the intersection dataset is provided. 
The other parameters are used to configure each intersection with traffic scenario variations. For details on the parameters, please refer to the `RLlib Intergration <https://intersectionzoo-docs.readthedocs.io/en/latest/rllib_integration.html#task-definitions>`_ section.
An important configuration needed is the ``curriculum_fn()`` function which is used to select the task on which the agent will be trained. 
In the example below, the task is randomly selected from the list of tasks defined in the ``PathTaskContext`` object. The ``single_approach`` parameter is set to True to only simulate one approach of the intersection at a time.

.. code-block:: python
    
    tasks = PathTaskContext(
        dir=Path(PATH),                    
        single_approach=True,
        penetration_rate=args.penetration,
        temperature_humidity=args.temperature_humidity,
        electric_or_regular=REGULAR,
    )

    def curriculum_fn(train_results, task_settable_env, env_ctx):
        return tasks.sample_task()

Next, the simulation configuration is defined. The ``IntersectionZooEnvConfig`` object is used to configure the simulation. The ``working_dir`` is where the simulation files are stored during the simualtion.
It is important to provide a task for initailizing the simulations. For this, ``task_context`` is set to a randomly sampled task from the ``PathTaskContext``. 
It will later be overriden by the curriculum function. 

.. code-block:: python

    env_conf = IntersectionZooEnvConfig(
        task_context=tasks.sample_task(),
        working_dir=Path(args.dir)
    )


Next, the RLlib policy is setup, in accordance with RLlib standard initialization. The ``PPOConfig`` object is used to configure the PPO policy. The ``rollouts`` method is used to configure the rollout settings.
The ``batch_mode`` is set to ``complete_episodes`` to ensure that the rollouts are complete episodes. The evalution configurtations will not be used for training but will be required to evaluate the policy once it is trained.
The ``.callbacks(MetricsCallback)`` is necessary to send the the custom metrics that IntersectionZoo collects to RLlib.

.. code-block:: python

    algo = (
        PPOConfig()
        .rollouts(num_rollout_workers=args.num_workers, sample_timeout_s=3600, \
            batch_mode="complete_episodes", rollout_fragment_length=400)
        .resources(num_gpus=args.num_gpus)
        .evaluation(evaluation_num_workers=1, evaluation_duration=1, \
            evaluation_duration_unit='episodes', evaluation_force_reset_envs_before_iteration=True)
        .environment(
            env=IntersectionZooEnv,
            env_config={"intersectionzoo_env_config": env_conf},
            env_task_fn=curriculum_fn,
        )
        .callbacks(MetricsCallback)
        .build()
    )

Finally, run the training for ``ITER`` iterations. The results are logged to `weights and biases <https://wandb.ai/home>`_ and the model checkpoint are saved every ``save_frequency`` iterations.

.. code-block:: python

    for i in range(ITER):
        
        result = algo.train()
        
        print(f"iteration {i} completed.")
        
        sampler_results = result['sampler_results']
        custom_results = result['custom_metrics']

        print({**sampler_results, **custom_results})
        
        if i % args.save_frequency == 0:
            save_dir = f'{args.dir}/runs/{str(i)}/{datetime.now().strftime("%Y%m%d_%H%M")}'
            checkpoint_dir = algo.save(save_dir).checkpoint.path
            print(f"Checkpoint saved at {checkpoint_dir}")



Evalution
---------

For evaluating the trained agent as described above, ``policy_evaluation.py`` can be used. The evaluation script is similar to the training script, with the exception of the evaluation configurations.

First the tasks on which the agent will be evaluated are defined.

.. code-block:: python
    
    tasks = PathTaskContext(
        dir=Path(PATH),
        single_approach=True,
        penetration_rate=args.penetration,
        temperature_humidity=args.temperature_humidity,
        electric_or_regular=REGULAR,
    )

Next, load the model checkpoint. The standard RLlib methods is used to load the model checkpoints.

.. code-block:: python

    algo = Algorithm.from_checkpoint(args.checkpoint)

The evaluation is then performed. For every single task listed in the ``tasks`` object, EVAL_PER_TASK times, the policy will be used to do rollouts. The results will be saved in a csv file. Please note that this 
file could be large with many columns as IntersectionZoo collected many metrics. Also note that the paramaters used by RLlib for evalution is loaded from the ``.evaluate`` call defined in the training script when the model checkpoints are loaded. 

.. code-block:: python

    res_df = pd.DataFrame()

    for i, task in enumerate(tasks.list_tasks(False)):
        for _ in range(EVAL_PER_TASK):
        
            algo.evaluation_workers.foreach_worker(
                    lambda ev: ev.foreach_env(
                        lambda env: env.set_task(task)))
            results = algo.evaluate()

            flattened_results = {**flatten_dict(results)}
            results_df = pd.DataFrame([flattened_results])
            res_df = pd.concat([res_df, results_df], ignore_index=True)
            
        print(f'Completed evaluation for task {i+1}/{len(tasks.list_tasks(False))}')

    res_df.to_csv(f'{args.dir}/eval_result_pen_rate_{args.penetration}.csv')

