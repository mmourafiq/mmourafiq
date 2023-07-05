---
layout: blog
title: "Sequence prediction using recurrent neural networks(LSTM) with TensorFlow"
subtitle: "LSTM regression using TensorFlow."
cover_image: images/blog/tensorflow.jpg
cover_image_caption: ""
tags: [deep-learning, python, tensorflow]
---
This post tries to demonstrates how to approximate a sequence of vectors using a recurrent neural networks, in particular I will be using the LSTM architecture, The complete code used for this post could be found [here](https://github.com/mouradmourafiq/tensorflow-lstm-regression). Most of the examples I found in the internet apply the LSTM architecture to natural language processing problems, and I couldn't find an example where this architecture could be used to predict continuous values.

----

## Update: code compatible with tensorflow 1.1.0

The same model can be achieved by using the LSTM layer from [polyaxon](https://github.com/polyaxon/polyaxon/), here's a an experiment configuration to achieve the same results from this post:

```python
import polyaxon as plx


def create_experiment(output_dir, X, y, train_steps=1000, num_units=7, output_units=1, num_layers=1):
    """Creates an experiment using LSTM architecture for timeseries regression problem."""

    config = {
        'name': 'time_series',
        'output_dir': output_dir,
        'eval_every_n_steps': 100,
        'train_steps_per_iteration': 100,
        'train_steps': train_steps,
        'run_config': {'save_checkpoints_steps': 100},
        'train_input_data_config': {
            'input_type': plx.configs.InputDataConfig.NUMPY,
            'pipeline_config': {'name': 'train', 'batch_size': 64, 'num_epochs': None,
                                'shuffle': False},
            'x': {'x': X['train']},
            'y': y['train']
        },
        'eval_input_data_config': {
            'input_type': plx.configs.InputDataConfig.NUMPY,
            'pipeline_config': {'name': 'eval', 'batch_size': 32, 'num_epochs': None,
                                'shuffle': False},
            'x': {'x': X['val']},
            'y': y['val']
        },
        'estimator_config': {'output_dir': output_dir},
        'model_config': {
            'module': 'Regressor',
            'loss_config': {'module': 'mean_squared_error'},
            'eval_metrics_config': [{'module': 'streaming_root_mean_squared_error'},
                                    {'module': 'streaming_mean_absolute_error'}],
            'optimizer_config': {'module': 'adagrad', 'learning_rate': 0.1},
            'graph_config': {
                'name': 'regressor',
                'features': ['x'],
                'definition': [
                    (plx.layers.LSTM, {'num_units': num_units, 'num_layers': num_layers}),
                    (plx.layers.FullyConnected, {'num_units': output_units}),
                ]
            }
        }
    }
    experiment_config = plx.configs.ExperimentConfig.read_configs(config)
    return plx.experiments.create_experiment(experiment_config)
```

End of the update.

----

## Original Post:

So the task here is to predict a sequence of real numbers based on previous observations. The traditional neural networks architectures can't do this, this is why recurrent neural networks were made to address this issue, as they allow to store previous information to predict future event.

In this example we will try to predict a couple of functions:

 * sin

![sin-function](/images/posts/sin.png)

 * sin and cos on the same time

![sin-cos-function](/images/posts/sin-cos.png)

 * x*sin(x)

![x*sin-function](/images/posts/xsin.png)

First of all let’s build our model, `lstm_model`, the model is a list of stacked lstm cells of different time steps followed by a dense layers.

```python
def lstm_model(time_steps, rnn_layers, dense_layers=None):
    """
    Creates a deep model based on:
        * stacked lstm cells
        * an optional dense layers
    :param time_steps: the number of time steps the model will be looking at.
    :param rnn_layers: list of int or dict
                         * list of int: the steps used to instantiate the `BasicLSTMCell` cell
                         * list of dict: [{steps: int, keep_prob: int}, ...]
    :param dense_layers: list of nodes for each layer
    :return: the model definition
    """

    def lstm_cells(layers):
        if isinstance(layers[0], dict):
            return [tf.nn.rnn_cell.DropoutWrapper(tf.nn.rnn_cell.BasicLSTMCell(layer['steps'],
                                                                               state_is_tuple=True),
                                                  layer['keep_prob'])
                    if layer.get('keep_prob') else tf.nn.rnn_cell.BasicLSTMCell(layer['steps'],
                                                                                state_is_tuple=True)
                    for layer in layers]
        return [tf.nn.rnn_cell.BasicLSTMCell(steps, state_is_tuple=True) for steps in layers]

    def dnn_layers(input_layers, layers):
        if layers and isinstance(layers, dict):
            return learn.ops.dnn(input_layers,
                                 layers['layers'],
                                 activation=layers.get('activation'),
                                 dropout=layers.get('dropout'))
        elif layers:
            return learn.ops.dnn(input_layers, layers)
        else:
            return input_layers

    def _lstm_model(X, y):
        stacked_lstm = tf.nn.rnn_cell.MultiRNNCell(lstm_cells(rnn_layers), state_is_tuple=True)
        x_ = learn.ops.split_squeeze(1, time_steps, X)
        output, layers = tf.nn.rnn(stacked_lstm, x_, dtype=dtypes.float32)
        output = dnn_layers(output[-1], dense_layers)
        return learn.models.linear_regression(output, y)

    return _lstm_model
```

So our model expects a data with dimension corresponding to `(batch size, time_steps of the first lstm cell, num_features in our data)`.


Next we need to prepare the data in a way that could be accepted by our model.

```python

def rnn_data(data, time_steps, labels=False):
    """
    creates new data frame based on previous observation
      * example:
        l = [1, 2, 3, 4, 5]
        time_steps = 2
        -> labels == False [[1, 2], [2, 3], [3, 4]]
        -> labels == True [2, 3, 4, 5]
    """
    rnn_df = []
    for i in range(len(data) - time_steps):
        if labels:
            try:
                rnn_df.append(data.iloc[i + time_steps].as_matrix())
            except AttributeError:
                rnn_df.append(data.iloc[i + time_steps])
        else:
            data_ = data.iloc[i: i + time_steps].as_matrix()
            rnn_df.append(data_ if len(data_.shape) > 1 else [[i] for i in data_])
    return np.array(rnn_df)


def split_data(data, val_size=0.1, test_size=0.1):
    """
    splits data to training, validation and testing parts
    """
    ntest = int(round(len(data) * (1 - test_size)))
    nval = int(round(len(data.iloc[:ntest]) * (1 - val_size)))

    df_train, df_val, df_test = data.iloc[:nval], data.iloc[nval:ntest], data.iloc[ntest:]

    return df_train, df_val, df_test


def prepare_data(data, time_steps, labels=False, val_size=0.1, test_size=0.1):
    """
    Given the number of `time_steps` and some data,
    prepares training, validation and test data for an lstm cell.
    """
    df_train, df_val, df_test = split_data(data, val_size, test_size)
    return (rnn_data(df_train, time_steps, labels=labels),
            rnn_data(df_val, time_steps, labels=labels),
            rnn_data(df_test, time_steps, labels=labels))


def generate_data(fct, x, time_steps, seperate=False):
    """generates data with based on a function fct"""
    data = fct(x)
    if not isinstance(data, pd.DataFrame):
        data = pd.DataFrame(data)
    train_x, val_x, test_x = prepare_data(data['a'] if seperate else data, time_steps)
    train_y, val_y, test_y = prepare_data(data['b'] if seperate else data, time_steps, labels=True)
    return dict(train=train_x, val=val_x, test=test_x), dict(train=train_y, val=val_y, test=test_y)

```

this will create a data that will allow our model to look `time_steps` number of times back in the past in order to make a prediction. So if for example our first cell is a 10 `time_steps` cell, then for each prediction we want to make, we need to feed the cell 10 historical data points. The `y` values should correspond to the tenth value of the data we want to predict.

Let's first define the hyperparameters

```python
LOG_DIR = './ops_logs'
TIMESTEPS = 5
RNN_LAYERS = [{'steps': TIMESTEPS}, {'steps': TIMESTEPS, 'keep_prob': 0.5}]
DENSE_LAYERS = [2]
TRAINING_STEPS = 130000
BATCH_SIZE = 100
PRINT_STEPS = TRAINING_STEPS / 100
```

Now we can create a regressor based on our our model

```python
regressor = learn.TensorFlowEstimator(model_fn=lstm_model(TIMESTEPS, RNN_LAYERS, DENSE_LAYERS),
                                      n_classes=0,
                                      verbose=1,  
                                      steps=TRAINING_STEPS,
                                      optimizer='Adagrad',
                                      learning_rate=0.03,
                                      batch_size=BATCH_SIZE)

```

### Predicting the `sin` function

```python
X, y = generate_data(np.sin, np.linspace(0, 100, 10000), TIMESTEPS, seperate=False)
# create a lstm instance and validation monitor
validation_monitor = learn.monitors.ValidationMonitor(X['val'], y['val'],
                                                      every_n_steps=PRINT_STEPS,
                                                      early_stopping_rounds=1000)
regressor.fit(X['train'], y['train'], validation_monitor, logdir=LOG_DIR)

# > last training steps
# Step #9700, epoch #119, avg. train loss: 0.00082, avg. val loss: 0.00084
# Step #9800, epoch #120, avg. train loss: 0.00083, avg. val loss: 0.00082
# Step #9900, epoch #122, avg. train loss: 0.00082, avg. val loss: 0.00082
# Step #10000, epoch #123, avg. train loss: 0.00081, avg. val loss: 0.00081
```

predicting the test data

```python
mse = mean_squared_error(regressor.predict(X['test']), y['test'])
print("Error: {}".format(mse))
# 0.000776
```

 * real sin function

![sin-function](/images/posts/predicted-sin.png)

### Predicting the `sin and cos` functions together

```python
def sin_cos(x):
    return pd.DataFrame(dict(a=np.sin(x), b=np.cos(x)), index=x)

X, y = generate_data(sin_cos, np.linspace(0, 100, 10000), TIMESTEPS, seperate=False)
# create a lstm instance and validation monitor
validation_monitor = learn.monitors.ValidationMonitor(X['val'], y['val'],
                                                      every_n_steps=PRINT_STEPS,
                                                      early_stopping_rounds=1000)
regressor.fit(X['train'], y['train'], validation_monitor, logdir=LOG_DIR)

# > last training steps
# Step #9500, epoch #117, avg. train loss: 0.00120, avg. val loss: 0.00118
# Step #9600, epoch #118, avg. train loss: 0.00121, avg. val loss: 0.00118
# Step #9700, epoch #119, avg. train loss: 0.00118, avg. val loss: 0.00118
# Step #9800, epoch #120, avg. train loss: 0.00118, avg. val loss: 0.00116
# Step #9900, epoch #122, avg. train loss: 0.00118, avg. val loss: 0.00115
# Step #10000, epoch #123, avg. train loss: 0.00117, avg. val loss: 0.00115
```

predicting the test data

```python
mse = mean_squared_error(regressor.predict(X['test']), y['test'])
print("Error: {}".format(mse))
# 0.001144
```

 * predicted sin-cos function

![sin-function](/images/posts/predicted-sin-cos.png)


Predicting the `x*sin` function

```python
def x_sin(x):
    return x * np.sin(x)

X, y = generate_data(x_sin, np.linspace(0, 100, 10000), TIMESTEPS, seperate=False)
# create a lstm instance and validation monitor
validation_monitor = learn.monitors.ValidationMonitor(X['val'], y['val'],
                                                      every_n_steps=PRINT_STEPS,
                                                      early_stopping_rounds=1000)
regressor.fit(X['train'], y['train'], validation_monitor, logdir=LOG_DIR)

# > last training steps
# Step #32500, epoch #401, avg. train loss: 0.48248, avg. val loss: 15.98678
# Step #33800, epoch #417, avg. train loss: 0.47391, avg. val loss: 15.92590
# Step #35100, epoch #433, avg. train loss: 0.45570, avg. val loss: 15.77346
# Step #36400, epoch #449, avg. train loss: 0.45853, avg. val loss: 15.61680
# Step #37700, epoch #465, avg. train loss: 0.44212, avg. val loss: 15.48604
# Step #39000, epoch #481, avg. train loss: 0.43224, avg. val loss: 15.43947
```

predicting the test data

```python
mse = mean_squared_error(regressor.predict(X['test']), y['test'])
print("Error: {}".format(mse))
# 61.024454351
```

 * real x*sin function

![sin-function](/images/posts/xsin.png)

 * predicted x*sin function

![sin-function](/images/posts/predicted-xsin.png)  

### model loss

![sin-loss](/images/posts/sin-loss.png)

![sin-loss-mean](/images/posts/sin-loss-mean.png)


**N.B** I am not completely sure if this is the right way to train lstm on regression problems, I am still experimenting with the [RNN sequence-to-sequence model](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/ops/seq2seq.py#L151), I will update this post or write a new one to use the sequence-to-sequence model.
