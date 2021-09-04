---
layout: post
title:  "RL in 0 AD: Custom State and Action Spaces"
date:   2021-02-28 12:55:00
categories: machine-learning games
---

In this post, we will be exploring defining our own state and action spaces for our RL agent. The initial introduction post, we trained an RL agent to micro ranged cavalry units against a small army of infantry. Not only is this a simplified scenario but we also presented the "world" (and possible actions) to the agent in a simple, easy-to-learn form.

Specifically, we encoded the current state of the game as a single number: the distance between the cavalry army and the infantry. The agent was also given two possible actions: attack or retreat (where retreat is a "walk" action in the opposite direction as the enemies). Although these simple representations, or state and action spaces, were fitting for an introductory post, they are not ideal for many use cases.

# What is a State/Action Space?
Before we start, it is important to be clear what a state or action space is. At a high level, a state space defines a numerical representation of the current game state and an action space is a numerical representation of all possible actions the AI could make for any given state. (The [OpenAI documentation](https://gym.openai.com/docs/#spaces) is a good resource for more info!) For example, in the earlier example where cavalry were learning to kite, the state space was simply the distance to the enemy units and the action space allowed the AI to choose between "attack" and "retreat."

 Using this approach makes it easy to train the ML model as it essentially needs to learn to attack if the input is greater than a given value. Since the action space is restricted to choosing between "attack" or "retreat" actions, the agent's behavior is also limited; there is no hope that it would learn to flank or divide its units to attack on different fronts. It can only attack the nearest or run directly away from the enemy units.

Alternatively, the action space could be more expressive and allow for finer unit control. One simple extension would be to allow the AI to select the direction to move the units. Although still pretty restrictive, this would allow the units to avoid the edges of the map when retreating and would be required if we wanted to be able to divide forces to flank the opposition. However, this will require a richer state space as the direction to run when retreating cannot be determined simply from the distance to the enemies! One approach would be to transition to displacement to the enemy units; another would be to use a minimap representation of the game state.


For now, let's keep it simple and use displacement!

# Defining a new state space
In general, defining a new state space requires two steps:
- Defining the state space itself
- Writing a function to map a given game state to a point in the state space

For example, the earlier state space represented the game as simply the distance between the player and enemy units. In other words, the state of the game was represented by a single number. The state space represents the space of all possible states. In this case, this would be a single number where the minimum value is 0 and maximum value is the diameter of the map. (Technically, you may want to normalize the value to be within 0 and 1. We will do this in a later example.)

Supposing the maximum diameter is 700, the state space would be defined in OpenAI Gym would be a single scalar value with a value between 0 and 700. Using OpenAI Gym, this would be defined as follows:

```python
from gym.spaces import Box

state_space = Box(0, 700, shape=(1,))  # min, max, shape of the data
```
We can then use this in a custom gym environment as shown below:
```python
"""
This is simply an example and not meant to actually be used!
We will take a better approach in the next section.
"""
import numpy as np
import gym
from gym.spaces import Box

def center(units):  # Helper method that we will use later!
    positions = np.array([ unit.position() for unit in units ])
    return np.mean(positions, axis=0)

class SimpleEnv(gym.Env):
    def __init__(self):
        self.observation_space = Box(0, 700, shape=(1,))
        # TODO: add action space

    def step(self, action_index):
        # TODO: convert the numerical action to a format understood by the game engine
        state = # TODO: Apply the given action and retrieve the current game state
        return self.observation(state)

    def observation(self, state):
        """Convert the game state to a numerical representation"""
        player_units = state.units(owner=1)
        enemy_units = state.units(owner=2)
        return np.linalg.norm(center(enemy_units) - center(player_units))
```

To make customizing the state/action spaces a little easier, `zero_ad_rl` uses `StateBuilder` and `ActionBuilder` classes. These classes enable you to separate the state/action representation from the rest of the logic of the environment (which may include configurations to load and/or levels in the case of curriculum learning). Using a `StateBuilder`, we could implement the above logic as follows (without all the extra placeholders for code that needs to be added in the future!):

```python
from zero_ad_rl.env.base import StateBuilder
import numpy as np
from gym.spaces import Box

def center(units):  # Helper method that we will use later!
    positions = np.array([ unit.position() for unit in units ])
    return np.mean(positions, axis=0)

class EnemyDistance(StateBuilder):
    def __init__(self):
        space = Box(0., 700., shape=(1,), dtype=np.float32)
        super().__init__(space)

    def from_json(self, state):
        """Convert the game state to a scalar btwn 0 and 700"""
        player_units = state.units(owner=1)
        enemy_units = state.units(owner=2)
        return np.linalg.norm(center(enemy_units) - center(player_units))
```
This state space is quite simplistic. Let's make it a little richer and switch to displacement instead! That is, rather than simply representing the game state simply as the distance to the enemy units, let us represent the game state using the x, z displacement. While we are at it, let's squash the distances to be between 0 and 1.

```python
from zero_ad_rl.env.base import StateBuilder
import numpy as np
from gym.spaces import Box

def center(units):
    positions = np.array([ unit.position() for unit in units ])
    return np.mean(positions, axis=0)

class EnemyDisplacement(StateBuilder):
    def __init__(self):
        space = Box(0., 1., shape=(2,), dtype=np.float32)
        super().__init__(space)

    def from_json(self, state):
        player_units = state.units(owner=1)
        enemy_units = state.units(owner=2)
        max_distance = 80
        displacement = center(enemy_units) - center(player_units)
        # Normalize (and make sure we handle any states where there are no units for one team)
        normalized_displacement = displacement/max_distance if not np.isnan(displacement/max_distance) else 1
        return np.array([ min(d, 1.) for d in normalized_displacement ])
```

# Defining a new action space
Next, we will define a new action space! Like state spaces, `zero_ad_rl` exposes some helpers for defining action spaces using the builder pattern. Let's first create an action builder for the "attack" and "retreat" actions mentioned earlier:

```python
import zero_ad
from zero_ad_rl.env.base import ActionBuilder
from gym.spaces import Discrete
import numpy as np

def center(units):  # Helper method that we will use later!
    positions = np.array([ unit.position() for unit in units ])
    return np.mean(positions, axis=0)

class AttackRetreat(ActionBuilder):
    def __init__(self):
        space = Discrete(2)  # The action space consists of two (discrete) actions: attack or retreat
        super().__init__(space)  

    def to_json(self, action_index, state):
        """
            Convert the numerical representation to a command
            for the game engine
        """
        return self.retreat(state) if action_index == 0 else self.attack(state)

    def retreat(self, state):
        """
            Given a state, create the action that would move
            the player units away from the enemies
        """
        units = state.units(owner=1)
        center_pt = center(units)
        offset = enemy_offset(state)
        rel_position = 20 * (offset / np.linalg.norm(offset, ord=2))
        position = list(center_pt - rel_position)
        return zero_ad.actions.walk(units, *position)

    def attack(self, state):
        """
            Given a state, create the action for attacking the enemy
            units. In this case, it selects the nearest enemy unit.
        """
        units = state.units(owner=1)
        center_pt = center(units)

        enemy_units = state.units(owner=2)
        enemy_positions = np.array([unit.position() for unit in enemy_units])
        dists = np.linalg.norm(enemy_positions - center_pt, ord=2, axis=1)
        closest_index = np.argmin(dists)
        closest_enemy = enemy_units[closest_index]

        return zero_ad.actions.attack(units, closest_enemy)
```

The main method used by an `ActionBuilder` is `to_json`. This method accepts a point in the action space (in this case a 0 or 1) along with the current state and resolves it to a command for 0 AD. The `zero_ad.actions` module provides some useful helpers for this! This particular action builder simply maps them to "attack" or "retreat" which are implemented using their own dedicated methods.

Now that we have a implemented simple action space, let's give the agent a little more control by allowing them to select the direction to move. In this next example, we will allow the agent to move in the main cardinal directions or attack the nearest unit. Let's jump into the code!

We can start from the `AttackRetreat` example (keeping the `attack` method) but we will be changing the action space from 2 to 5:
```python
import zero_ad
from zero_ad_rl.env.base import ActionBuilder
from gym.spaces import Discrete
import numpy as np

def center(units):  # Helper method that we will use later!
    positions = np.array([ unit.position() for unit in units ])
    return np.mean(positions, axis=0)

class AttackMove(ActionBuilder):
    def __init__(self):
        space = Discrete(5)
        super().__init__(space)  

    def to_json(self, action_index, state):
        pass # we will take care of this later!

    def attack(self, state):
        units = state.units(owner=1)
        center_pt = center(units)

        enemy_units = state.units(owner=2)
        enemy_positions = np.array([unit.position() for unit in enemy_units])
        dists = np.linalg.norm(enemy_positions - center_pt, ord=2, axis=1)
        closest_index = np.argmin(dists)
        closest_enemy = enemy_units[closest_index]

        return zero_ad.actions.attack(units, closest_enemy)
```

This action builder has an action space consisting of 5 discrete choices: attack, move north, move east, move west, or move south. However, the RL model will represent the action as a number in the action space (ie, numbers 0-4) which we need to convert to a command that 0 AD understands (just like we did with the original `AttackRetreat` example). To this end, we can decide that 4 will represent "attack" and the other numbers will correspond to the movement actions. Let's add this to our action builder!

First, we will need to import `math`:
```python
import math
```

Then we can finish implementing `to_json`:
```python
    def to_json(self, action_index, state):
        if action_index == 4:
            return self.attack(state)
        else:
            circle_ratio = action_index/4
            self.move(state, 2 * math.pi * circle_ratio)

    def move(self, state, angle):
        units = state.units(owner=1)
        center_pt = center(units)
        distance = 15
        offset = distance * np.array([math.cos(angle), math.sin(angle)])
        position = list(center_pt + offset)

        return zero_ad.actions.walk(units, *position)
```

## Composing the Gym environment
Now that we have defined our custom state and action spaces, we can create our custom OpenAI gym environment that uses them! There is a generic environment in `zero_ad_rl` that we can now use to create the OpenAI Gym environment using our state and action builders. This generic environment also requires the address to the game instance running the RL interface and the configuration for the scenario to load in 0 AD.

```python
from zero_ad_rl.env.base import ZeroADEnv
from zero_ad_rl.env import scenarios

# Add your state, action builder definitions here...

address = 'http://127.0.0.1:6000'
scenario_config = scenarios.load_config('CavalryVsInfantry')
env = ZeroADEnv(address, scenario_config, AttackMove(), EnemyDisplacement())
```

Now, we can use `env` with any OpenAI Gym-compatible RL algorithms or frameworks! A few examples are [RLlib](https://docs.ray.io/en/master/rllib-env.html), [Coach](https://intellabs.github.io/coach/), or [ReAgent](https://reagent.ai/).

## Training an agent using RLlib
Finally, we can train an RL agent using this environment using RLlib. First, we will convert our script to a simple script that wraps `zero_ad_rl.train` (which in turn wraps the RLlib `train` CLI tool). It is quite simple and just registering the environment before delegating the rest of the meaningful logic to `zero_ad_rl.train`. Simply add the following code to a `train.py` file:

```python
from ray.tune.registry import register_env
from zero_ad_rl.train import create_parser, run
from zero_ad_rl.env import scenarios

# Add your state, action builder definitions here...

address = 'http://127.0.0.1:6000'
scenario_config = scenarios.load_config('CavalryVsInfantry')
register_env('CavVsInfDirections', lambda c: ZeroADEnv(address, scenario_config, AttackMove(), EnemyDisplacement()))

parser = create_parser()
args = parser.parse_args()
run(args, parser)
```

Now we can run this script using any of the options available to the [RLlib train command](https://docs.ray.io/en/latest/rllib-training.html#getting-started)! For example, we can train a PPO agent by first starting 0 AD w/ the RL interface enabled
```
pyrogenesis --rl-interface=127.0.0.1:6000 --mod=rl-scenarios --mod=public
```

then training the agent with
```
python train.py --env CavVsInfDirections --run PPO --checkpoint-freq 25 --experiment-name MyFirstEnvironment
```

At this point, you should see a bunch of training logs! If so, you have officially created your own custom state/action space to train an RL agent! Stay tuned for more blog posts on collecting imitation learning data and training agents using multiple environments!
