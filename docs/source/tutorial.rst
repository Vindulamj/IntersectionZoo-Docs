Tutorial
========

Step by step guide through the training of an RLlib PPO agent and evaluation. Please refer to ``ppo_training.py`` and ```` for all the details

Training
--------

.. code-block:: python

    env_conf = IntersectionZooEnvConfig(
        task_context=tasks.sample_task(),
        working_dir=Path(args.dir),
        moves_emissions_models=[args.temperature_humidity],
        fleet_reward_ratio=1,
    )

Here the environment is setup. The task gien here tough does not matter because it is later overriden by the curriculum function. It is however required for the env to be initialized.
The ``working_dir`` is where the simulation files are stored during the simualtion. Other params can be left to default.


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


Here a TaskContext object is first defined, which itself defines a wide number of unique tasks. The ``PATH`` is the path to a dataset.
The ``single_approach`` parameter is set to True to only simulate one approach of the intersection at a time. It is the recommended setting as it accelerates rollouts which usually collect enough samples given the large number of agents.
The curriculum function is later passed to RLlib to select on which task each rollout will be performed. Here we simply randomly select a task from the ones defined above.

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

Here the RLlib policy is setup, in a fairly standard way. Please note:
- The ``sample_timeout_s`` needs to be large because the simulation can be slow.
- The ``batch_mode`` is set to ``complete_episodes`` to ensure that the rollouts are complete episodes, and avoids issues when evaluating.
- The evalution config will be used later when evaluating the policy.
- The ``.callbacks(MetricsCallback)`` is necessary to collect the env's custom metrics.

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

Runs the training for ITER iterations. The results are printed and the checkpoint are saved every ``save_frequency`` iterations.

Evalution
---------

.. code-block:: python
    tasks = PathTaskContext(
        dir=Path(PATH),
        single_approach=True,
        penetration_rate=args.penetration,
        temperature_humidity=args.temperature_humidity,
        electric_or_regular=REGULAR,
    )

First the tasks on which the agent will be evaluated are defined. The same as in the training.

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

The evaluation is performed here, for every single task listed in the ``tasks`` object, EVAL_PER_TASK times. The results are saved in a csv file.
Please note that the paramater used by RLlib for the ``.evaluate`` call are the ones defined in the training script.