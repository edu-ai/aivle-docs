# Creating Your Test Suite

[GitHub repo](https://github.com/edu-ai/aivle-grader)

---

## Overview

There are 3 components of user's concern:

- **Agent**: replaced by student's submission
- **Evaluator**: records the history of simulation, give scores from the history
- **Test Cases**: runs the underlying environment (Gym compatible is sufficient. I also provide examples on how to adapt
aiVLE Gym environments. The process is very straightforward as aiVLE Gym is also Gym compatible.) with runtime and 
episode count limit, attaches evaluator to the simulation to produce evaluation results.

A typical example of grader program looks like this:

```python
env = gym.make("...")  # any OpenAI Gym compatible environment is acceptable
evaluator = RewardEvaluator()
test_cases = [ReinforcementLearningTestCase(...), ...]
test_suite = TestSuite(suite_id="...", cases=test_cases)
eval_result = test_suite.run(create_agent)
```

## Examples

Please refer to the `main` function of `aivle_gym_xxx.py` in [examples](https://github.com/edu-ai/aivle-grader/tree/main/examples)
folder of aiVLE Grader repository.

## Documentation

### `Agent` abstract class

To be considered a gradable agent, one needs to provide:

- `step(state)`: returns an action from observed state
- `reset()`: resets internal state for a new episode - reset is guaranteed to be called once before every episode

### `Evaluator` abstract class

!!! note
    You may consider `Evaluator` as a storage of evaluation results (most of the time, equivalent to reward) for each episode/
    run. At the end of evaluation (after n_runs episodes concluded), you can get a summary of this evaluation session using 
    `get_result()` method.

There are 4 abstract methods that need to be implemented:

- `reset()`: called once at the beginning of each run
- `step(full_state: dict)`: called once after taking one action in the environment. You should give everything you 
received from env.step in full_state argument. The evaluator will decide which information to use.
- `get_result()`: returns an EvaluationResult object that summarizes all executed episodes

#### Built-in concrete class

You may refer to these concrete implementations before creating your own custom concrete subclass.

- `RewardEvaluator`: computes average reward across episodes 
    - ensure reward field is provided in full_state
- `StepCountEvaluator`: computes average steps across episodes
    - no special requirements

### TestCase abstract class

There are 6 properties that need initialization:

- `case_id`
- `time_limit`
- `n_runs`: number of episodes to run
- `agent_init`: init params passed to `__init__` method of Agent
- `env`: Gym compatible environment
- `evaluator`: `Evaluator` object

There is one abstract method that needs to be implemented:

- `run`: runs the env environment for `n_runs` times with evaluator attached to the execution.

#### Built-in concrete class

You may refer to these concrete implementations before creating your own custom concrete subclass.

- `ReinforcementLearningTestCase`
