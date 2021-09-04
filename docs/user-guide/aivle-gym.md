# Creating Your Environment

[GitHub repo](https://github.com/edu-ai/aivle-gym)

---

## Getting Started

We will call the original Gym environment the *base environment* in this tutorial.

There are three components in an aiVLE Gym environment:

- serializer (`EnvSerializer`): translates certain non-JSON compatible Python objects into compatible ones, and in reverse
- agent env (`AgentEnv`): agent-side Gym-compatible class that communicates with the judge-side (simulation-side)
- judge env (`JudgeEnv`/`JudgeMultiEnv`): simulation environment, you may reuse existing Gym environment with little modification

You will create concrete class of corresponding abstract base class by implementing certain abstract methods. (Also
remember to call the base class constructor in `__init__` method)

### Single-agent task

We'll use Gym's built-in cart pole environment as an example:

#### Serializer
```python
class CartPoleEnvSerializer(EnvSerializer):
    def action_to_json(self, action):
        return action
    def json_to_action(self, action_json):
        return action_json

    '''Because numpy.array objects are not JSON-serializable by default,
    we provide custom methods to marshal/unmarshal observations.
    As shown in this example, if action/observation/info are JSON-serializable
    to begin with, you just return the original value.
    '''
    def observation_to_json(self, obs):
        return obs.tolist()
    def json_to_observation(self, obs_json):
        return numpy.array(obs_json)

    def info_to_json(self, info):
        return info
    def json_to_info(self, info_json):
        return info_json
```

#### Agent Environment
```python
class CartPoleAgentEnv(AgentEnv):
    '''Instead of instantiating the base environment like in this example,
    since action/observation space and reward range are constants,
    you may use these constants directly when creating the agent environment.
    '''
    def __init__(self):
        base_env = gym.make('CartPole-v0')
        super().__init__(CartPoleEnvSerializer(), base_env.action_space, base_env.observation_space,
                         base_env.reward_range, uid=0)  # uid can be any int for single-agent agent env
```

#### Judge Environment

As shown below, if you don't have special requirements, simply calling the corresponding methods in the base
environment is good enough.

```python
class CartPoleJudgeEnv(JudgeEnv):
    def __init__(self):
        self.env = gym.make('CartPole-v0')
        super().__init__(CartPoleEnvSerializer(), self.env.action_space, self.env.observation_space,
                         self.env.reward_range)
    def step(self, action):
        return self.env.step(action)
    def reset(self):
        return self.env.reset()
    def render(self, mode='human'):
        return self.env.render(mode=mode)
    def close(self):
        self.env.close()
    def seed(self, seed=None):
        self.env.seed(seed)
```

#### Agent

Note that `CartPoleAgentEnv()` and `gym.make('CartPole-v0')` are designed to be interchangable in the agent code.

```python
use_aivle = True
if use_aivle:
    env = CartPoleAgentEnv()
else:
    env = gym.make('CartPole-v0')
for i_episode in range(10):
    env.reset()
    for t in range(100):
        env.render()
        action = env.action_space.sample()
        observation, reward, done, info = env.step(action)
        if done:
            break
env.close()
```

#### Execution

These code can be found under [./example](https://github.com/edu-ai/aivle-gym/tree/main/example). To execute this concrete example:

1. `python judge.py` to start the simulation process first
2. `python agent.py` to run the agent code

### Multi-agent task

Multi-agent case is very similar to single-agent case, differences are:

- agent env:
    - `action_space`, `observation_space` and `reward_range` need to be that of this specific agent
    - `uid` needs to be meaningful (i.e. unique among all participating agents, etc.)
- judge env: additional constructor params:
    - `n_agents`: number of agents
    - `uid_to_idx`: map from uid to agent index (0-indexed)

A concrete example can be found under [./example/multi_agent.py](https://github.com/edu-ai/aivle-gym/tree/main/example/multi_agent.py) 
and [./example/multi_judge.py](https://github.com/edu-ai/aivle-gym/tree/main/example/multi_judge.py)