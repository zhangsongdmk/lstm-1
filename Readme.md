
### How to define a prediction interval for LSTM network for time series.

This is the desciption of software used in the [How to define a predictioni interval based on training set for an LSTM network for time-series.
](https://rtaubes.pythonanywhere.com/lstm-1/)

One of the possible types of appications for Long-short term neural networks is forecasting.
There are a lot of articles and books describing how to implement and use LSTM for prediction of count
of passengeres in airlines, trends in a stock market.
Having forecasted value, it is important to answer question:<br/>
What is a possible range for predicted value? In other words, if predicted value is X, actually it
means that actual value expected to be in the range [X-x0, X+x1], where<br/>
__X__ - predicted value,<br/>
__x0__ and __x1__ - lower and upper bounds of prediction<br/>

The LSTM model is based on Tensorflow.
The source of data is the Kaggle competition [Bitcoin Historical Data](https://www.kaggle.com/mczielinski/bitcoin-historical-data).

The data generator in the 'src' folder can generate not only batches from CSV files, but also
some simple sequences like Sin for debugging.
The main algorithm of using the data source is:

1. create a data generator
<pre>
  import data_generator
  gen = data_generator.BatchGenerator(...)
</pre>
2. do training of a model use the code
<pre>
  for idx, x_batches, y_batches in gen:
    do something
</pre>
3. if it is required, it is possible to reset a data generator to begin using the method reset().
For example:
<pre>
  gen.reset()
  for idx, x_batches, y_batches is gen:
    do something
</pre>
  This method is used by the evaluate() method of Model:

    - temporary set a batch size to 1
    - reset generator using the reset()
    - make predictions using data of generator
    - returns actual and predicted values
  The results of estimate() can be used to estimate a quality of a trained algorithm.

4. When the previous cycle has been finished, it is possible to make prediction of a future value using the last batch with length 1
5. Having a new input value, it can be added to a generator, and a generator will be ready to create a new batch of data for a model.
6. Another possibility is to add a predicted data to a generator, and predict new value. This method can be useful if more than one
value should be predicted.

I want to notice that for currently prediction is used only for estimation of a prediction interval. Forecasting of future values
is out of the current topic and will be implemented later.

The LSTM model saves checkpoints each 10 steps(this value can be changed), and at the end of each traing.
This allows to make an interactive training:

- Define in settings how many epochs is used for training
- create model instanse as model = Model(SETS)
- call the create() method which removes all previous checkpoints for this model
- call the train() method of the model.
- estimate quality of the model
- if model should be trained later, call the train() method
- repeat the two last steps as much as you need

Another possibility is to call the method from_latests_point() when a new model has been created.
In this case, rather then starting training from scratch, model will start from the latest point.

The third method allows to start training or evaluation from some saved point. Use the method from_custom_point() where the argument
is the path to a checkpoint. The path should not use extension. For example, if a checkpoint includes 3 files 'ckpt-1.data', 'cpkt-1.index', 'cpkt-1.data-...',
use 'ckpt-1' as the path to the checkpoint.

There are two set of settings in the notebook. One is used for debugging an algorithm and settings on a laptop
without using a GPU.<br/>
Another set is used for a computer with GPU. This algoritm was trained on a Google computer with GPU.


__TODO__:

- implement the predict() method of Model
- move the Model class outside of the notebook
- implement an algorithm for retraining using when a received value is out of calculated confidence interval.

