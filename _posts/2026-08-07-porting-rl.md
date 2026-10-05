---
layout: post
title: "(RL) Moving from PettingZoo to TorchRL"
categories: code
---

[Hello There](https://tenor.com/en-GB/view/hello-there-gif-5677380953331354485). This is somehow not the vacation post I wanted to write, but I managed to [nerd-snipe](https://xkcd.com/356/) myself early enough that it's become the one I ended up writing. This series of posts detail the porting effort for one of my RL projects. In particular, I'm porting it from the general [PettingZoo](https://pettingzoo.farama.org) API to a more specialised [TorchRL's EnvBase](https://docs.pytorch.org/rl/stable/reference/generated/torchrl.envs.EnvBase.html#torchrl.envs.EnvBase) API. I'll try to list my thoughts while I do this, the pros and cons of this task, as well as the code-related problems I encountered along the way.

Before we start, here's some helpful resources that were quite invaluable throughout:

- [TorchRL: Get started with Environments, TED and transforms](https://docs.pytorch.org/rl/stable/tutorials/getting-started-0.html)
- [TorchRL: Competitive Multi-Agent Reinforcement Learning (DDPG) with TorchRL Tutorial](https://docs.pytorch.org/rl/stable/tutorials/multiagent_competitive_ddpg.html)
- [Yoann Poupart's Blog: MARL Cluster Training](https://yp-edu.github.io/projects/marl-cluster-training)

In this post, we'll first look at the motivation to do this and the factors that weigh in (at least, in my head), when doing such a refactor. After that, we'll move into the technicalities of the port itself. We'll end on some gotchas that caught me off guard.

This will be a long(ish) post, so feel free to come back to it over multiple passes, or skip ahead to the part you like.


##### Table of Contents

- [Why would I do this?](#why-would-i-do-this)
   + [The Upsides](#the-upsides)
   + [The Downsides](#the-downsides)
   + [The Decision](#the-decision)
- [The Plan™️](#the-plan)
- [Part 1: Spec Definitions and Architecture](#part-1-spec-definitions-and-architecture)
   * [Making Environment Specs](#making-environment-specs)
   * [Environment Configuration via `gen_params`](#environment-configuration-via-gen_params)
      + [Interlude: What is the Environment State?](#interlude-what-is-the-environment-state)
- [Part 2: Environment Reset](#part-2-environment-reset)
- [Part 3: Step Execution Engine](#part-3-step-execution-engine)
- [Part 4: Verification and Testing](#part-4-verification-and-testing)
- [Bonus: Unexpected Gotchas](#bonus-unexpected-gotchas)
   + [Manual Batch Handling](#manual-batch-handling)
   + [The God Object Problem](#the-god-object-problem)
- [Conclusions](#conclusions)

# Why would I do this?

So, why would I ever want to do this? The original environment is a perfectly functional PettingZoo environment, with integrated RL benches. It has a lot of configurability too; this was achieved via many parameters to change the game settings, many fixed policies and scenarios to train against. It's quite good, in all fairness.

But, despite all this, I had my reasons: it's difficult to extend/change its core logic as it is currently. I was a horrible programmer back in the day (this has not significantly changed since), and I interleaved reward generation, observations, and the actual dynamics of the game into one big mess. Case-in-point: I wanted to change a particular feature, and ended up looking at a huge list of potential changes I'd have to make. This amounted to rewriting large parts of the environment, so I decided to cut my losses and start over.


### The Upsides

This is where the TorchRL rewrite decision crept in, for the following (targeted) advantages:

- *Extensibility*: Adding new observation and reward models should be easier; this is more of a code sanity thing, but quite important for me.
- *Performance* (_to verify_): Since I plan on running my future MARL experiments with the [BenchMARL](https://benchmarl.readthedocs.io/en/latest/) suite anyway, the hope is that I can fit both environment and algorithm onto the GPU to get massive speed gains.
- *Reproducibility*: TorchRL environments can be seeded to ensure JAX-like perfect reproducibility. While the environment is reproducible, I'm doing some horrible things to make it so, and would prefer a much cleaner (almost JAX-like) way to handle things.

### The Downsides

However, as anyone in machine learning will tell you, there is no free lunch. This comes with a couple of huge disadvantages that I had to weigh in:

- *Reimplementation cost*: I will end up rewriting the environment, provide new tests, documentation and have to suitably extend the MARL suite. This is not trivial, not by a long shot.
- *Maintainability*: This hasn't really changed from the PettingZoo days. In fact, it might be worse, since writing everything in tensor operations might make the code even more opaque, so downstream users looking to extend its dynamics will have a hard time.

### The Decision

The downsides are [huge](https://tenor.com/en-GB/view/thats-huge-enormous-thats-big-huge-linus-gif-25856235), almost offsetting the advantages. The primary I went ahead with this is because firstly, I can soak up the reimplementation cost (for now). Furthermore, I expect significant gains in training time for these policies, while hopefully making the code easier to change. The hope is that with sufficiently clean code separating the dynamics, observations, and rewards, extensions will become easier to write, since downstream users (hopefully) only need to modify one place in the code.

# The Plan™️

Since we're doing this, the porting effort needs to be structured if we're going to have a shot at pulling this off. I'm going to divide this into a couple of posts in order for readability reasons. The first will focus on getting the core machinery of the environment working. The second part will focus on its correctness, and on things like rendering and a couple of toy examples.

1. Spec Definitions & Architecture
  -  Standardize Composite specs for multi-agent shapes (`action_spec`, `observation_spec`, `reward_spec`).
  -  Configure the environment state for compatibility with tensor-ops.
2. Environment Reset (`_reset`) 
  -  Provide initial state (verify it is reproducible when supplying a seed!) and initialise the environment.
  -  Vectorisation starts here, and a lot of primitives I'll end up using start to take form here.
3. Step Execution Engine (`_step`)
  -  Write the mechanics of transition model. This is done in tensor-ops to escape slow Python loops and conditionals.
  -  Write observation model as a function `_get_observations (old_state, actions, new_state) -> observations`. Note that this mirrors the definition of a Partially Observable Stochastic Game (POMG). Ditto for rewards.
4. Verification & Testing
  -  Pass TorchRL `check_env_specs(env)`. This is done continually during development to ensure that random rollouts execute without error. This is a preliminary check, and only says that the entire chain works well, has the right shapes etc.
  -  Write tests via PyTest to verify that the environment's behaviour is as intended.


# Part 1: Spec Definitions and Architecture

So, the initial skeleton of the new TorchRL environment is roughly:

```python 
from torchrl.envs import EnvBase

class Env(EnvBase):
    def __init__(self, td_params=None, seed=None, device="cpu"):
        ...

    def _reset(self, td: TensorDict = None, **kwargs) -> TensorDic      t:
        ...

    def _step(self, td: TensorDict) -> TensorDict:
        ...

    def _make_spec(self, td_params) -> None:
        ...

    @staticmethod
    def gen_params(...) -> TensorDictBase:
        ...

```

The `__init__` method, as usual, handles the initialisation of the environment. This is mostly setting the right parameters to properly initialise the environment before use; here, these configuration parameters are passed via `td_params:  TensorDict`. If such a `td_params` dictionary isn't passed, the `gen_params` method is charged with sensible defaults for the environment (more on this in a minute).

## Making Environment Specs

The first important thing is to define the shape of the action and observation spaces, as well as that of the rewards. The first is similar to how PettingZoo defines things: they define what valid actions and observations look like. TorchRL goes a little further and enforces this for the rewards as well.

```python
    def _make_spec():
         ...
         self.action_spec = Composite(
           agents=Categorical(                                  
               n=ACTIONS,                                  
               shape=torch.Size([self.num_agents]),
               dtype=torch.int,
               device=self.device,                           
           ),                                                  
           ...
        ) 
```

Given that most of this logic is similar to the PettingZoo version, this is just a one-to-one mapping (almost makes me wonder if I can write a parser to transform from one to the other). I do end up changing how the observations look, but that's not the important part for this post.

## Environment Configuration via `gen_params`

The other important thing to do is to implement a helper function called `gen_params`. This isn't strictly necessary, but it's a nice trick I picked up from the official tutorials.

This function handles environment configuration via an initial `TensorDict` object (Torch's version of a dictionary, and the main currency of this library). An example of this to create environments that overwrite the defaults is the following.

```python
    @staticmethod                                               
    def gen_params(...) -> TensorDict:
        return TensorDict({                           
                "num_agents": 2,              
                "grid_size": 5,
        })                         

...

# Using gen_params to create a custom env
# by overwriting the defaults.
if __name__ == "__main__":
    params = Env.gen_params()
    params["num_agents"] = 3 
    env = Env(td_params=params)
```

The actual implementation has to account for batch sizes i.e., when initialising multiple environments as we do for MARL experiments, but the above code is roughly what happens. Another nice thing that I do from here on out is to continually validate the environment using the `check_env_specs` function (See [torchrl.envs.utils.check_env_specs](https://docs.pytorch.org/rl/main/reference/generated/torchrl.envs.check_env_specs.html)).

```python
from torchrl.envs.utils import check_env_specs
...

if __name__ == "__main__":
    ...
    check_env_specs(env)
```

This will reset the environment (i.e., call `reset`), and run an episode (i.e., call `step` until episode termination). It also checks that the specs defined earlier in `_make_spec` are being respected during each step. Since it throws an error if they aren't, this is a good sanity check to keep while coding up something this complex.

### Interlude: What is the Environment State?

When implementing the `reset` method, I was forced to make architectural choices. Here, the environment state needs to be reset to some default. The natural question is: how is the environment state represented?

My old way of dealing with this was to use dictionaries and dataclasses; this is extremely Pythonic, but horrendous for performance. For example,
```python
# old method 
self.grid.state : dict = {
    "agent_1": np.array([x_p1, y_p1, num_objects, num_prey]),
    ...
    "obj_1_agent_1: np.array([x_tp1, y_tp1]),
}
```
where `"object_1_agent_1" : Object` is actually a Python dataclass containing the `name` and `value` fields. Looking back on this, I owe a public apology to anyone who had to see this code. 

My time with JAX has cleansed my spirit, however. In particular, I realised that if I want speed, I need to throw away everything superfluous in this representation. For example, instead of using dictionaries, I can use three `torch.Tensor`s to represent the grid state as a `TensorClass` (See [TensorClass](https://docs.pytorch.org/td/main/reference/tc.html)).

```python 
from td import TensorClass

class GridState(TensorClass):
    agents: torch.tensor # Shape: [#agents, 4]
    objects: torch.tensor # Shape: [#agents, #num_objects_per_agent, 3]
```

So, each agent needs two bits of information to indicate his position on the grid (x and y coordinates), and need to track how many objects and prey they're carrying (So, `(x,y,num_objects,num_prey)`). Each object requires one status bit, and two bits for location (So, `(status, x, y)`). This status bit is quite important, since we base a lot of the transition dynamics on this. This status bit is represented by an `Enum` as follows:

```python 
from enum import IntEnum

class ObjStatus(IntEnum):
   Captured: int = -1
   Inactive: int = 0
   ActiveEmpty: int = 1
   ActiveFull: int = 2
```

These status bits are defined as:
- `ObjStatus.Captured`: This object is captured. The location $(x,y)$ should also be $(-1,-1)$ here.
- `ObjStatus.Inactive` (Default): This object is inactive, and is being carried by its owning agent.
- `ObjStatus.ActiveEmpty`: This object is active, placed on cell $(x,y)$, but is empty.
- `ObjStatus.ActiveFull`: This object is active, placed on cell $(x,y)$, and has captured a prey.

Notice that this representation of the state is extremely compact and consists entirely of numbers (integers, in fact). This is very, very important, since that means we can use a lot of tensor operations to do our calculations. The `ObjStatus` enum also helps us write extremely readable code, as we will see in a second, which removes the "magic numbers" worry I had about the old code. 

The downside is that we must enforce the contract here: for example, every time an object is captured, we must set its location to $(-1,-1)$ as well as its status bit. Or, whenever an object is recovered by its owning agent, its location must never be read (this is a garbage value), but its status bit must be $0$. 

# Part 2: Environment Reset

With this compact representation, we can define the `_reset` method quite cleanly, and this gives us our first taste of the tensor-ops to come.

```python

    def _reset(self, td: TensorDict = None, **kwargs) -> TensorDict:
        ...
        agents = torch.randint(
            self.grid_size,
            size=(self.num_agents, 2),
            generator=self.rng,
            device=self.device,
        )
        objects = torch.zeros(
            size=(self.num_agents, self.num_objects, 3), device=self.device
        )
        td["state"] = GridState(
            agents=agents, objects=objects
        ).to(torch.int32)
        
        td["observation"] = ... # Create first observations for all agents
        ... # Populate done, terminated and truncated booleans
        return td
```
The part I'm most proud of is that by choosing the default for `ObjStatus.Inactive` as $0$, an appropriately sized zero-matrix is all we need to perfectly initialise all objects. Isn't that neat? 

There's other logic here, notably to create the initial observations for all agents, setting other flags and the environment seed for reproducibility. With those done, that wraps up the entire reset method!  

# Part 3: Step Execution Engine

That brings us to the heart of the port: simulating the transition dynamics in `_step`. This is also where the `td: TensorDict` that we just created comes into play. The way I understand it is that while JAX opts for a functional (read: pure functions and immutability) style, TorchRL's approach is to pass around a global `TensorDict` object where everyone modifies their part. So, it shouldn't be shocking that the signature of the `_step` function is just `_step(self, td: TensorDict) -> TensorDict:`

Now, the step is divided into multiple steps, each of which performs a part of the transition. This is part of the formal model, is inspired by how the original model was written, and is a clean separation of concerns. For example, the helper functions `_agents_move` and `_object_physics` only update the state entries; more formally, they simulate the transition model $T(s',a,s)$ of the formal POMG model.

```python
    def _step(self, td: TensorDict) -> TensorDict:
        # Store this to calculate observations
        td["old_state"] = td["state"].clone()

        # Run through the transition model.
        self._agents_move(td)
        self._object_physics(td)

        # Calculate rewards and update observations.
        out = TensorDict(device=self.device, batch_size=td.batch_size)
        out.update(
            self._calculate_rewards(new_state, actions, old_state)
        )
        out.update(
            self.obs_model.generate_observations(
                new_state, actions, old_state, prior_obs, self.rng
            )
        )

        return out
```

The real problem is that each of these little steps was defined as Pythonic for-loops over Python objects, and was definitely not very fast. It was correct, however. The challenge, then, is to write each little step using tensor operations to avoid slow conditionals and loops. Fortunately, that's exactly what JAX trained me to do. Let's go into, for example, the `_agents_move` step.

```python
# Global constant
ACTION_DELTAS: torch.Tensor = torch.Tensor(
    [  # No-Op, Up,      Left,   Down,   Right,   PlaceObject
        [0, 0], [0, -1], [1, 0], [0, 1], [-1, 0], [0, 0],
    ]
)

class Env:
    ...
    def _agents_move(self, td: TensorDict) -> None:
        agent_locs = td["state", "agents"]
        actions = td["agents"]
        td["state", "agents"] = (
            agent_locs + ACTION_DELTAS[actions]
        ).clamp(min=0, max=self.grid_size - 1)
```

That's it! The real work is done by the `ACTION_DELTAS` constant: to each action, it associates the effect on the location $(x,y)$ of each agent. So, just indexing into it via each agent's actions fetches the right delta. All we need to do is make sure the agents don't move out-of-bounds; this is done by calling `clamp(min = ..., max = ...)` that makes sure that all location bits (both $x$ and $y$) are within the grid.

Suffice it to say that the other elements of the transition dynamics are coded up similarly: masks for conditionals, array slices instead of loops. There's some more work in there to generate the rewards and observations, but the only important facts for these two are:

- The transition steps we just described do _not_ implement either rewards or observations. They only modify the state.
- These are generated after everything is done, using helper functions with the signature:

This is about as close as I can get to respecting the POMG model, and makes sure that changing observation/reward models is simple to do: just change one function, in one file.

# Part 4: Verification and Testing

With such dense code, as is wont to be produced by vectorised/functional styles, we need a lot of tests to make sure that we have the right model, which follows the original as closely as possible, and doesn't have any new bugs, say, due to bad dimensions, logic, etc.

The basic way to test this was by using [`check_env_specs`](https://docs.pytorch.org/rl/main/reference/generated/torchrl.envs.check_env_specs.html), a helper provided by TorchRL to verify that the action/observation specs are being followed during episodes. I've got the following version here: 

```python 
BATCH_SIZES_TO_TEST: list = [None, 10, (10, 2), (10, 2, 2)]
SEEDS_TO_TEST: list = list(range(5))


@pytest.mark.parametrize("seed", SEEDS_TO_TEST)
@pytest.mark.parametrize("batch_size", BATCH_SIZES_TO_TEST)
def test_check_env(batch_size: int | tuple[int], seed: int):
    """
    Calls check_env_specs on environment with/without batching.
    This is a preliminary check to ensure that the environment runs.
    """
    params = Env.gen_params()
    env = Env(td_params=params, batch_size=batch_size)
    check_env_specs(env, seed=seed)
```

Using `pytest.mark.parametrize`, I can freely verify that the environment specs are functioning as intended, for a wide variety of configurations. This rules out the first (and most common) source of bugs: dimensions. 

Once that's done, we need to test specifics: each test is a scenario built to test one specific step mechanic/observation model/rewards, etc. For example, one of the shorter tests I have is expected to fail: if an agent picks up an object, and the environment doesn't update both agent and object statuses correctly, then throw an error.

<div class="remark" text="Conditional checks" markdown="1">
I pass an `is_strict: bool` flag to the environment, which is False by default. All of these checks are usually enabled (by if-conditionals) only if this flag is explicitly passed as True. This is done so as to not disturb the control flow during actual execution.
</div>

```python 
@pytest.mark.xfail
@pytest.mark.parametrize("batch_size", BATCH_SIZES_TO_TEST)
def test_agent_state_sync1(batch_size: int | tuple[int]):

    params = Env.gen_params()
    env = Env(td_params=params, batch_size=batch_size, is_strict=True)
    td = env.reset()

    env.state.objects[..., 0] = ObjStatus.Captured
    env._update_action_masks(env.state, td)
    assert (
        td["agents", "action_mask"][..., 0, Action.PlaceObject] == False
    ).all(), "Agent without objects is allowed to place an object."
```

Add in a few of these, get test coverage to $100%$, and we can be reasonably sure (not entirely!), that we have verified our environment! 

# Bonus: Unexpected Gotchas

Well, on a high note, we're done! I'll just take the time to mention a couple of things that were/are quite important, especially for TorchRL environments. They were pain points for me while writing this, so I assume I have to document them, at least for myself.

### Manual Batch Handling

Unlike JAX, TorchRL does not automatically handle batching for us. Instead, it falls onto us to incorporate the batching dimension into each of our calculations. This is a problem, since our matrices could have dimensions `[#R, #P, #Objects, 3]` or `[batch_size, #R, #P, #Objects, 3]`, depending on if the user calls it in batched more (i.e., MARL experiments), or not.

We also need to distinguish Parallel environments and Batched environments (Vectorisation). As detailed [here](https://docs.pytorch.org/rl/stable/tutorials/torchrl_envs.html#running-environments-in-parallel), Parallel environments are process-level parallelism i.e., each environment is executed on a separate Python process. On the other hand, Batched environments group multiple environments within the same process ([See EnvBase.batch_size](https://docs.pytorch.org/rl/stable/reference/generated/torchrl.envs.EnvBase.html#envbase), or [Brax](https://docs.pytorch.org/rl/stable/reference/generated/torchrl.envs.BraxEnv.html#torchrl.envs.BraxEnv)). This is also called Vectorisation, and is the more familiar thing for JAX-heavy folk like me.

We want to handle Batching via our coded logic. This way, we can batch a few environments per process, and run multiple processes via TorchRL's parallellism (See again [Brax](https://docs.pytorch.org/rl/stable/reference/generated/torchrl.envs.BraxEnv.html#torchrl.envs.BraxEnv)).

To handle this, we first realise that the tails are identical in the aforementioned dimension shapes. So, we must change our indexing operations to start matching dimensions from the back, instead of explicitly managing the entire thing. A key tool is the ellipsis index operator, which matches all dimensions upto the first specified one.

```python
# action_mask: Shape(#agents, #actions)

# Old, unbatched code
action_mask[:,0] = 1

# New, batch-agnostic code
action_mask[...,0] = 1
```

Lots of changes like these make the code in this post subtly different from what is actually in the codebase, but the underlying spirit is the same. The downside is that this plays further into the Maintainability problem we had before - if it wasn't obvious before, it definitely isn't any better now. The upside is that we can dice our vectorisation and parallelisation any way we want, which is a lot of freedom for the folks downstream.

### The God Object Problem

The last hurdle is about code quality. As you might have noticed, our `Env` has a _lot_ of helper functions. It's taking on transition dynamics, state management, movements, action mask generation, rendering - you name it, it does it. There's a name I learned recently about this pattern: the [God Object](https://en.wikipedia.org/wiki/God_object).

Divinity aside, it's not a good sign at all. We therefore split up the class into multiple files as such, each with a clear purpose.

```bash
.
├── model.py
└── observation.py
└── utils.py
```

Note that I didn't do this in the beginning, since it's a lot harder to wrestle with code correctness (especially for tensor-ops) while also wrestling code organisation. However, we magicked out the observation model (which we wanted to be modular anyway), into its own class. This means that I can (and do) test the observation model separately, independent of the actual logic of the state transitions. On the other hand, it also fixes my gripes about having everything intertwined in the original version.

# Conclusions

But, with this done, the environment is finally ported! In other words, [I can finally sleep!](https://tenor.com/en-GB/view/revenk-gif-202597353950725337) I hope this serves as a useful primer on how to code custom TorchRL environments, especially if you're porting it from an older environment.

