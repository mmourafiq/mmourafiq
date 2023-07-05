---
layout: blog
title: "Introducing a tensorflow library for deep learning and reinforcement learning"
subtitle: "This post tries to answer a couple of practical questions about the library and the intent behind it's creation."
cover_image: images/blog/tensorflow.jpg
cover_image_caption: ""
tags: [deep-learning, python, tensorflow]
---

This post tries to answer a couple of practical questions about the project and the intent behind it's creation.

## What is polyaxon-lib

polyaxon-lib is a library and built on top of TensorFlow for building deep learning and reinforcement learning experiments. It provides popular DL and RL modules that can be easily customized and assembled for creating real-world machine learning models.

## Why another TensorFlow library

Since the intention of creating library is to have an end to end system and a platform for managing experiments, the choice of creating a new library was very important, we wanted to to have complete control over the apis and future design decisions. We tried before to base the platform on Keras and/or tf.contrib.learn, but it was not easy, we were brought to tweak many parts which ended by creating a new library. In fact, if you have a chance to read the code, you will find that many programming decisions were completely inspired by these libraries, and sometimes only patching/mirroring the code of these libraries.

Having said that, the use of polyaxon-lib does not exclude using other libraries. So every module created by polyaxon-lib could be mixed and extended by other libraries.


## How can I start using polyaxon-lib

polyaxon-lib is very modular and provides a simple interface to creating deep learning algorithms.

There are 2 ways you can follow to use polyaxon-lib:

  * By creating Python objects in a programatic way. polyaxon-lib provides high level Layers and Models based on GraphModules, a high level object that easily share variables.

  * By creating configuration files (Json or YAML). In this case, the library is meant to be used in applications that want to utilize an intelligent system to achieve some task. This configurations would then be customized through an interface, web/cli, and would enable the user to easily tweak the parameters without caring about all the underlying implementation.

### An example of graph defined in a Json file:

```json
{
   "name":"MyModel",
   "output_dir":"/tmp/polyaxon-log/mymodel",
   "eval_every_n_steps":10,
   "train_steps_per_iteration":1000,
   "run_config":{
      "save_checkpoints_steps":100
   },
   "train_input_data_config":{
      "pipeline_config":{
         "module":"TFRecordImagePipeline",
         "batch_size":64,
         "num_epochs":5,
         "shuffle":true,
         "dynamic_pad":false,
         "params":{
            "data_files":"/path-to-my-data/mnist_train.tfrecord",
            "meta_data_file":"/path-to-my-data/meta_data.json"
         }
      }
   },
   "eval_input_data_config":{
      "pipeline_config":{
         "module":"TFRecordImagePipeline",
         "batch_size":32,
         "num_epochs":1,
         "shuffle":true,
         "dynamic_pad":false,
         "params":{
            "data_files":"/path-to-my-data/mnist_eval.tfrecord",
            "meta_data_file":"/path-to-my-data/meta_data.json"
         }
      }
   },
   "estimator_config":{
      "output_dir":"/tmp/polyaxon-log/mymodel"
   },
   "model_config":{
      "summaries":[
         "loss",
         "gradients",
         "activations"
      ],
      "module":"Classifier",
      "loss_config":{
         "module":"softmax_cross_entropy"
      },
      "eval_metrics_config":[
         {
            "module":"streaming_true_positives"
         },
         {
            "module":"streaming_true_negatives"
         },
         {
            "module":"streaming_false_positives"
         },
         {
            "module":"streaming_false_negatives"
         },
         {
            "module":"streaming_recall"
         },
         {
            "module":"streaming_auc"
         },
         {
            "module":"streaming_accuracy"
         },
         {
            "module":"streaming_precision"
         }
      ],
      "optimizer_config":{
         "module":"adam",
         "learning_rate":0.007,
         "decay_type":"exponential_decay",
         "decay_rate":0.2
      },
      "one_hot_encode":true,
      "n_classes":10,
      "graph_config":{
         "name":"lenet",
         "features":[
            "image"
         ],
         "definition":[
            [
               "Conv2d",
               {
                  "regularizer":"l2_regularizer",
                  "num_filter":32,
                  "filter_size":5,
                  "strides":1
               }
            ],
            [
               "MaxPool2d",
               {
                  "kernel_size":2
               }
            ],
            [
               "Conv2d",
               {
                  "num_filter":64,
                  "filter_size":5,
                  "regularizer":"l2_regularizer"
               }
            ],
            [
               "MaxPool2d",
               {
                  "kernel_size":2
               }
            ],
            [
               "FullyConnected",
               {
                  "num_units":1024,
                  "activation":"tanh"
               }
            ],
            [
               "FullyConnected",
               {
                  "num_units":10
               }
            ]
         ]
      }
   }
}
```


### An example of graph defined in a Python file:

```python
import polyaxon_lib as plx
import tensorflow as tf

env = plx.envs.GymEnvironment('CartPole-v0')

def graph_fn(mode, inputs):
    return plx.layers.FullyConnected(mode, num_units=512)(inputs['state'])

def model_fn(features, labels, mode):
    model = plx.models.DDQNModel(
        mode,
        graph_fn=graph_fn,
        loss_config=plx.configs.LossConfig(module='huber_loss'),
        num_states=env.num_states,
        num_actions=env.num_actions,
        optimizer_config=plx.configs.OptimizerConfig(module='sgd', learning_rate=0.01),
        exploration_config=plx.configs.ExplorationConfig(module='decay'),
        target_update_frequency=10,
        summaries='all')
    return model(features, labels)

memory = plx.rl.memories.Memory()
estimator = plx.estimators.Agent(
  model_fn=model_fn, memory=memory, model_dir="/tmp/polyaxon_logs/ddqn_cartpole")

estimator.train(env)
```


## What do you support currently in Polyaxon

The current polyaxon-lib version offers a very rich set of modular [layers](/docs/layers/introduction/), deep learning [models](/docs/models/models/) and reinforcement learning models, [here](/docs/models/rl_q_models/) and [here](/docs/models/rl_pg_models/), [estimators](/docs/estimators/estimator/)/[agents](/docs/agents/agent/) to construct your algorithms with.


For training and evaluating an algorithm, you can use Polyaxon built-in [experiments](/docs/experiments/introduction/) or use custom scripts with training loops and evaluation.

## Can I train my models in a distributed way

Yes, as it was mentioned before, the project was initialy built on top of tf.learn, but the API was constantly changing and breaking, and this is why we decided to roll our own estimator and experiemnt modules, with the idea that in the future we can interchangeably use tensorflow built in estimators/experiments when they become stable.

## How can deploy and monitor my algorithms

First we hope that you might find a useful use-case of Polyaxon for your projects. We are currently working on getting an architecture which will allow to easily manage, monitor, and compare experiments. And finally deploy an API to the cloud. We are also trying different approaches and concepts in an internal version to have a consistent, simple and stable way to create useful application of deep learning to real world problems.
