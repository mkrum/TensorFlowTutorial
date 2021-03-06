#Building a LSTM in TensorFlow
##Example Walk Through

In this tutorial, I am going to be going through how to build a simple LSTM in TensorFlow. In this LSTM, we are going to train an LSTM to predict a sin and cosine curve concurrently. First, we will begin with the necessary imports.

	import numpy as np
	import tensorflow as tf
	from math import sin, cos
	import matplotlib.pyplot as plt

We also need to define some constants that we are going to use later on.

	n_hidden =  128
	n_rows =  5
	n_vals = 2
	n_outputs = 2


n_hidden is the number of hidden layers, n_rows is the number of rows we are going to have in each inputs, n_vals is the number of vals in each row, n_outputs is the number of points that we are going to predict. Next we need to define some tensor variables.


	x = tf.placeholder(tf.float32, [None, n_rows, n_vals])
	y = tf.placeholder(tf.float32, [None, n_outputs])
				
	weights =  tf.Variable(tf.truncated_normal([n_hidden, n_outputs], stddev=0.1))
	biases  =  tf.Variable(tf.constant(0.1, shape=[n_outputs]))
				


x is the variable that will hold our inputs. This inputs will consist of five sets of two values, one for sin and one for cosine. The y value will hold the target. The target will be the next two values for the functions. 


	x_m =  tf.transpose(x, [1, 0, 2])
	x_m =  tf.reshape(x_m, [-1, n_vals])
	x_m =  tf.split(0, n_rows, x_m)


This data needs to be slightly formatted before it is used with some of the tensorflow functions we will be using. Namely, the data can only have rank 2, meaning that it can only have tow different dimensions. This means that since our data is currently in the form (batch_size, 5, 2), we need to condense this to just two dimensions.

	
	lstm_cell       =  tf.nn.rnn_cell.BasicLSTMCell(n_hidden, state_is_tuple=True)
	outputs, states =  tf.nn.rnn(lstm_cell, x_m, dtype=tf.float32)
	y_              =  tf.nn.tanh(tf.matmul(outputs[-1], weights) + biases)

These are the main functions that drive the actual LSTM functionality. We create the LSTM cell using a builtin function called BasicLSTMCell. We then pass this cell along with our inputs into a rnn function.
	
	
	cost = tf.reduce_mean(tf.square(y - y_))
	optimizer =  tf.train.AdamOptimizer(.001).minimize(cost)

Here we are just defining the standard cost and optimizer functions.
	
	with tf.Session() as sess:
	    sess.run(tf.initialize_all_variables())
	    lin      =  np.linspace(0, 100, 2000)
	    sin_vals =  np.array([ sin(l) for l in lin ])
	    cos_vals =  np.array([ cos(l) for l in lin ])
	    vals     =  []
	
	    for i in range(len(sin_vals) - 5):
	        row = []
	        for j in range(5):
	            row.append([cos_vals[i + j], sin_vals[i + j]])
	        vals.append(row)
	
	    vals = np.array(vals)
	    targets =  [ v[-1] for v in vals[1:] ]
	    vals    =  np.array(vals[:-1])
	    targets =  np.reshape(targets, (-1, n_vals))
	    test_vals = vals[-100:]
	    train_vals = vals[:-100]
	    test_targets = targets[-100:]
	    train_targets = targets[:-100]
	
	    for i in range(0, 1900):
	        sess.run(optimizer, feed_dict={x: train_vals[i:i + 10], y: train_targets[i: i + 10]})
	
	    in_x = sess.run(x_m, feed_dict={x: test_vals})
	    projected_y = sess.run(y_, feed_dict={x: test_vals})
	    plt.plot(test_targets, 'ro')
	    plt.plot(projected_y, 'bo')
	    plt.show()

Now we actually run the model. The majority of this code is specific to the dataset, the important part is that we are feeding a numpy array of shape (1900, 5, 2) and (1900, 2) into the model to train and then (100, 5, 2) and (100, 2) to test. If you run the code, you should see that the plot of the projected variables are very similar to the actual values.

##Assignment
In this assignment we are going to be creating a LSTM that can do simple image classification on the MNIST dataset.
<ol
<li> First we need to get this data set. Since the mnist is such a common dataset, tensorflow comes with a built in way to load the data. Add this to the begin of your program: </li>

	from tensorflow.examples.tutorials.mnist import input_data
	mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

<li> Define the n_hidden, n_rows, n_vals, and n_outputs. The images are 28 by 28. The input will be an entire image, row by row. The outputs will be an array of activations for each digit. </li>
<li> Declare the x and y placeholders you will need to use. </li>
<li> Declare the weights and bias variables that you will need. </li>
<li> Modify the x variable to fit into the BasicLSTMCell. </li>
<li> Declare your BasicLSTMCell and rnn layers. </li>
<li> Declare your y\_ to be the activation of ouputs[-1] and the weights and bias. Experiment with different activation functions. </li>
<li> Define a cost function and an optimizer. </li>
<li> Create a tf.Session() and run through your data. In order to get a train batch, use the function mnist.train.next_batch(batch_size). In order to get your train data, use mnist.test.images and mnist.test.labels. These images will initially be flat. Make sure you resize them to (-1, 28, 28) before you put them into the feed_dict. </li>
</ol>
	 
