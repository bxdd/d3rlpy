Tips
====

Reproducibility
---------------

Reproducibility is one of the most important things when doing research
activity.
Here is a simple example in d3rlpy.

.. code-block:: python

  import d3rlpy
  import gym

  # set random seeds in random module, numpy module and PyTorch module.
  d3rlpy.seed(313)

  # set environment seed
  env = gym.make('Hopper-v2')
  env.seed(313)

Create your own dataset
-----------------------

It's easy to create your own dataset with d3rlpy.

.. code-block:: python

  import d3rlpy

  # vector observation
  # 1000 steps of observations with shape of (100,)
  observations = np.random.random((1000, 100))

  # image observation
  # 1000 steps of observations with shape of (3, 84, 84)
  observations = np.random.randint(256, size=(1000, 3, 84, 84), dtype=np.uint8)

  # 1000 steps of actions with shape of (4,)
  actions = np.random.random((1000, 4))
  # 1000 steps of rewards
  rewards = np.random.random(1000)
  # 1000 steps of terminal flags
  terminals = np.random.randint(2, size=1000)

  dataset = d3rlpy.dataset.MDPDataset(observations, actions, rewards, terminals)

  # train with your dataset
  cql = d3rlpy.algos.CQL()
  cql.fit(dataset)

Please check :ref:`mdp_dataset` for more details.

Learning from image observation
-------------------------------

d3rlpy supports both vector observations and image observations.
There are several things you need to care about if you want to train RL agents from
image observations.

.. code-block:: python

  from d3rlpy.dataset import MDPDataset

  # observation MUST be uint8 array, and the channel-first images
  observations = np.random.randint(256, size=(100000, 1, 84, 84), dtype=np.uint8)
  actions = np.random.randomint(4, size=100000)
  rewards = np.random.random(100000)
  terminals = np.random.randint(2, size=100000)

  dataset = MDPDataset(observations, actions, rewards, terminals)


  from d3rlpy.algos import DQN

  dqn = DQN(scaler='pixel', # you MUST set pixel scaler
            n_frames=4) # you CAN set the number of frames to stack

Improve performance beyond the original paper
---------------------------------------------

d3rlpy provides many options that you can use to improve performance potentially
beyond the original paper.
All the options are powerful, but the best combinations and hyperparameters are
always dependent on the tasks.

.. code-block:: python

  from d3rlpy.models.encoders import DefaultEncoderFactory
  from d3rlpy.models.q_functions import QRQFunctionFactory
  from d3rlpy.algos import DQN, SAC

  # use batch normalization
  # this seems to improve performance with discrete action-spaces
  encoder = DefaultEncoderFactory(use_batch_norm=True)

  dqn = DQN(encoder_factory=encoder,
            n_critics=5,  # Q function ensemble size
            n_steps=5, # N-step TD backup
            q_func_factory='qr') # use distributional Q function

  # use dropout
  # this will dramatically improve performance
  encoder = DefaultEncoderFactory(dropout_rate=0.2)

  sac = SAC(actor_encoder_factory=encoder)
