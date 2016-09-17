## dprl
Deep reinforcement learning package for torch7. 

Algorithms:
* Deep Q-learning [[1]](#references)
* Double DQN [[2]](#references)
* Bootstrapped DQN (broken) [[3]](#references)
* Asynchronous advantage actor-critic [[4]](#references)

## Installation

```
git clone https://github.com/PoHsunSu/dprl.git
cd dprl
luarocks make dprl-scm-1.rockspec
```

## Example

#### Play Catch using double deep Q-learning
[Script](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua)
#### Play Catch using asynchronous advantage actor-critic

## Library
The library provides implementation of deep reinforcement learning algorithms.

### <a name="dql"></a> Deep Q-learning (dprl.dql)
This class contains learning and testing procedures for **d**eep **Q**-**l**earning [[1]](#references).

#### <a name="dprl.dql"></a>dprl.dql(dqn, env, config[, statePreprop[, actPreprop]])

This is the constructor of `dql`. Its arguments are:

* `dqn`: a deep Q-network ([dprl.dqn](#dqn)) or double DQN ([dprl.ddqn](#ddqn))

* `env`: an environment with interfaces defined in [rlenvs](https://github.com/Kaixhin/rlenvs#api)

* `config`: a table containing configurations of `dql`
	* `step`: number of steps before an episode terminates
	* `updatePeriod`: number of steps between successive updates of target Q-network

* `statePreprop`: a function which receives observation from `env` as argument and returns state for `dqn`. See [test-dql-catch.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example

* `actPreprop`: a function which receives output of `dqn` and returns action for `env`. See [test-dql-catch.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example


#### <a name="dql:learn"></a>dql:learn(episode[, report])
This method implements learning procedure of `dql`. Its arguments are:
* `episode`: number of episodes which `dql` learns for
* `report`: a function called at each step for reporting the status of learning. Its inputs are transition, current step number, and current episode number. A transition contains the following keys:
	* `s`: current state
	* `a`: current action
	* `r`: reward given action `a` at state `s`
	* `ns`: next state given action `a` at state `s`
	* `t`: boolean value telling whether `ns` is terminal state or not

You can use `report` to compute total reward of an episode or print the estimated Q value by `dqn`. See [test-dql-catch.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example.


#### <a name="dql:test"></a>dql:test(episode[, report])
This method implements test procedure of `dql`. Its arguments are:
* `episode`: number of episodes which `dql` tests for
* `report`: see `report` in [dql:learn](#dql:learn)

### <a name="dqn"></a> Deep Q-network (dprl.dqn)
"dqn" means **d**eep **Q**-**n**etwork [[1]](#references). It is the back-end of `dql`. It implements interfaces to train the underlying neural network model. It also implements experiment replay. 

#### <a name="dprl.dqn"></a>dprl.dqn(qnet, config, optim, optimConfig)
This is the constructor of `dqn`. Its arguments are:
* `qnet`: a neural network model built with the [nn](https://github.com/torch/nn) package. Its input is always **mini-batch of states** whose dimension defined by `statePreprop` (see [dprl.dql](#dprl.dql)). Its output is estimated Q values of all possible actions.

* `config`: a table containing the following configurations of `dqn`
	* `replaySize`: size of replay memory	
	* `batchSize`: size of mini-batch of training cases sampled on each replay
	* `discount`: discount factor of reward
	* `epsilon`: the ε of ε-greedy exploration
* `optim`: optimization in the [optim](https://github.com/torch/optim) package for training `qnet`. 
* `optimConfig`: configuration of `optim`

### <a name="ddqn"></a> Double Deep Q-network (dprl.ddqn)
"ddqn" means **d**ouble **d**eep **Q**-**n**etwork [[2]](#references). It inherets from `dprl.dqn`. We get double deep Q-learning by giving  [`ddqn`](#ddqn), instead of [`dqn`](#dqn), to [`dql`](#dql) . 

The only difference of `dprl.ddqn` to `dprl.dqn` is how it compute target Q-value. `dprl.ddqn` is recommended because it alleviates the over-estimation problem of `dprl.dqn` [[2]](#references). 
#### <a name="dprl.dqnn"></a>dprl.dqnn(qnet, config, optim, optimConfig)
This is the constructor of `dprl.dqnn`. Its arguments are identical to [dprl.dqn](#dprl.dqn).

### <a name="bdql"></a>Bootstrapped deep Q-learning (dprl.bdql) (Not working now)
`dprl.bdql` implements learning procedure in Bootstrapped DQN. Except initialization, its usage is identical to `dprl.dql`.

1. Initialize a bootstrapped deep Q-learning agent. 

	```
	local bdql = dprl.bdql(bdqn, env, config, statePreprop, actPreprop)
	```
	Except the first arguments `bdqn`, which is an instance of [`dprl.bdqn`](#bdqn), definitions of the other arguments are the same in [`dprl.dql`](#dql).

### <a name="bdqn"></a>Bootstrapped deep Q-network (dprl.bdqn)
`dprl.bdqn` inherets [`dprl.dqn`](#dqn). It is customized for Bootstrapped Deep Q-network.

1. Initialize `dprl.bdqn`
	```
	local bdqn = dprl.bdqn(bqnet, config, optim, optimConfig)
	```
	
	arguments:
	* `bqnet`: a bootstrapped neural network with module [`Bootstrap`](#Bootstrap). 
	* `config`: a table containing the following configurations for `bdqn`
		* `replaySize`, `batchSize`, `discount` ,and `epsilon`: see `config` in [`dprl.dqn`](#dqn).	
		* `headNum`: the number of heads in bootstrapped neural network `bqnet`.
	* `optim`: see `optim` in [`dprl.dqn`](#dqn).
	* `optimConfig`: see `optimConfig` in [`dprl.dqn`](#dqn).

### <a name="asyncl"></a>Asynchronous learning (dprl.asyncl)

`dprl.asyncl` is the framework for asynchronous learning [[4]](#references). It manages the multi-threaded procedure in asynchronous learning. The asynchronous advantage actor critic (a3c) algorithm is realized by providing adventage actor critic agent ([dprl.aac](#aac)) to Asynchronous learning (`dprl.asyncl`). See [test-a3c-atari.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-a3c-atari.lua) for example.

#### <a name="dprl.asyncl"></a> dprl.asyncl(asynclAgent, env, config[, statePreprop[, actPreprop[, rewardPreprop]]])

This is the constructor of `dprl.asyncl`. Its arguments are:

* `asynclAgent`: a learning agent such as avantage actor-critic ([dprl.aac](#aac)).

* `env`: an eviroment with interfaces defined in [rlenvs](https://github.com/Kaixhin/rlenvs#api)

* `config`: a table containing configurations of `asyncl`
	* `nthread`: number of actor-leaner threads
	* `maxSteps`: maximum number of steps in an episode on testing
	* `loadPackage`: an optional function called before construting actor-learner thread for loading package
	* `loadEnv`: an optional function to load enviroment (`env`) in actor-learner thread if the enviroment is not serializable. Enviroments written in Lua are serializable but those written in C like Atari emulator are not serializable. See in [test-a3c-atari.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example.

* `statePreprop`: a function which receives observation from `env` as argument and returns state for `asynclAgent`. See [test-a3c-catch.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example.

* `actPreprop`: a function which receives output of `aac` or other learning agent and returns action for `env`. See [test-a3c-catch.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example.

* `rewardPreprop`: a function which receives reward from `env` as argument and returns processed reward for `asynclAgent`. See reward clippling in [test-a3c-atari.lua](https://github.com/PoHsunSu/dprl/blob/master/example/test-dql-catch.lua) for example.

#### <a name="asyncl:learn"></a> dprl.asyncl:learn(Tmax[, report])
This method is the learning procedure of `dprl.asyncl`. Its arguments are:
* `Tmax`: the limit of total (global) learning steps of all actor-learner threads
* `report`: a function called at each step for reporting the status of learning. Its arguments are transition, count of learning steps in a thread, and count of global learning steps. A transition contains the following keys:
	* `s`: current state
	* `a`: current action
	* `r`: reward given action `a` at state `s`
	* `ns`: next state given action `a` at state `s`
	* `t`: boolean value telling whether `ns` is terminal state or not

### <a name="aac"></a>Advantage actor-critic (dprl.aac)
This class implements routines called by the asynchronous framework `dprl.asyncl` for realizing the Asynchronous Advantage Actor-Critic (a3c) algorithm [[4]](#references). It trains the actor and critic neural network model that should be provided on construction.
#### <a name="dprl.aac"></a> dprl.aac(anet, cnet, config, optim, optimConfig)
This is the constructor of `dprl.aac`. Its arguments are:
* `anet`: the actor network. Its input and output must conform to the requirement of environment `env` in `dprl.asyncl`. Note that you can use `statePreprop` and `actPreprop` in `dprl.asyncl`.
* `cnet`: the critic network. Its input is the same as `anet` and its output must be a single value tensor.
* `config`: a table containing configurations of `aac`
	* `tmax`: number of steps between performing asynchronous update of global parameters
	* `discount`: discount factor for computing total reward
	* `criticGradScale`: a scale multiplying the gradient parameters of critic network `cnet` to allow different learning rate between actor network and critic network.

To share parameters between actor network and critic network, make sure gradient parameters (i.e. `'gradWeight'` and `'gradBias'`) is shared as well. For example, 
```
local cnet = anet:clone('weight','bias','gradWeight','gradBias')

```
## <a name="mod"></a>Modules

#### <a name="Bootstrap"></a>Bootstrap (`nn.Bootstrap`)
This module is for constructing bootstrapped network [[3]](#references). Let the shared network be `shareNet` and the head network be `headNet`. A bootstrapped network `bqnet` for `dprl.bdqn` can be constructed as follows:
```
require 'Bootstrap'

-- Definition of 'shareNet' and head 'headNet'


-- Decorate headNet with nn.Bootstrap
local boostrappedHeadNet = nn.Bootstrap(headNet, headNum, param_init)

-- Connect shareNet and boostrappedHeadNet
local bqnet = nn.Sequential():add(shareNet):add(boostrappedHeadNet)
```
`headNum`: the number of heads of the bootstrapped network

`param_init`: a scalar value controlling the range or variance of parameter initialization in headNet.
It is passed to method `headNet:reset(param_init)` after constructing clones of headNet.

<!---##<a name="API"></a>API-->


## References
[1] Volodymyr Mnih et al., “Human-Level Control through Deep Reinforcement Learning,” Nature 518, no. 7540 (February 26, 2015): 529–33, doi:10.1038/nature14236.

[2] Hado van Hasselt, Arthur Guez, and David Silver, “Deep Reinforcement Learning with Double Q-Learning,” arXiv:1509.06461, September 22, 2015, http://arxiv.org/abs/1509.06461.

[3] Ian Osband et al., “Deep Exploration via Bootstrapped DQN,” arXiv:1602.04621, February 15, 2016, http://arxiv.org/abs/1602.04621.

[4] Volodymyr Mnih et al., “Asynchronous Methods for Deep Reinforcement Learning,” arXiv:1602.01783, February 4, 2016, http://arxiv.org/abs/1602.01783.


<!---
## TODO
#### dqn, dql

- [x] Add test scripts of using optim
- [x] Implement remaining mechenics of DQN
- [x] Finish readme
- [ ] Cuda support
- [ ] Double Q learning
- [ ] Deep exploration
- [ ] Prioritized experience replay
-->
