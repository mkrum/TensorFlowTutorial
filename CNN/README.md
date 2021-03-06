##MNIST Expert Example Walk Through 
### I will be going through the tutorial available [here] (https://www.tensorflow.org/versions/r0.9/tutorials/mnist/pros/index.html)
You may want to also consider reading through the [beginner version] (https://www.tensorflow.org/versions/r0.9/tutorials/mnist/beginners/index.html)
This tutorial assumes the reader understands at some level the functionality of a convolutional neural network. If this is not the case, I recommend the following introductory reasources:
[here] (http://deeplearning.net/tutorial/lenet.html)
and [here] (http://cs231n.github.io/convolutional-networks/)

The full code can be found [here] (https://github.com/mkrum/TFSamples/blob/master/mnistexample.py)


###Loading the Data
Correctly loading the data into the network will be one of the most time consuming parts of building a network. This example uses a pre-loaded version that drastically simplifies things. The dataset is called MNIST and it is a series of 28 by 28 greyscale images of hand written digits. The goal of the network will be to classify these handwritten digits to their actual values.

<pre><code>
from tensorflow.examples.tutorials.mnist import input_data 
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
</code></pre>

When you build your own network, you will need to convert the images into numpy arrays of floats scaled between 0 and 1. The next step is relatively straightforward, declaring the placeholders. 

<pre><code>
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
</code></pre>

x is the image data. It is of shape [None, 784], because it is not specifying the number of images, but it is specifying the they will be of size 784 (total number of pixels in a 28 by 28 greyscale image). y_ is the labels. It is of shape [None, 10] because again, it is not specifying the number of images, but it is specifying the size of the label, which is ten. The label is of size ten because it is being store as a one-hot representation of each digit. This means that 0 is 1000000000, 1 is 0100000000, 2 is 0010000000, and so on. Next, we declare some useful functions for later in the program.

<pre><code>
def weight_variable(shape):
      initial = tf.truncated_normal(shape, stddev=0.1)
      return tf.Variable(initial)
 
def bias_variable(shape):
      initial = tf.constant(0.1, shape=shape)
      return tf.Variable(initial)
</code></pre>

These functions are just shortcuts for the declaration of weights and bias variables that will be used in the network. Both functions return variables of the requested shape. Weights are randomized using the tf.truncated_normal function, which generates random values. Biases are all initialized at a constant value of 0.1. Next we declare the functions that while define the behavior of this network.

<pre><code>
def conv2d(x, W):
      return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')
  
def max_pool_2x2(x):
      return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                           	  strides=[1, 2, 2, 1], padding='SAME')
                           	  
</code></pre>
	
Since TensorFlow is a deep learning package, it has many built in functions that are incredibly helpful. Here we see two examples of this. Building a 2-d convolutional layer is accomplished in a single line. This function, conv2d, takes in two parameters, x and W. x is the input to the convolutional layer and W is the weights. The strides are the distance the kernel moves in each direction. SAME means that padding is added along to the edges to make the shapes match as needed. VALID means no padding. The max-pooling layer only takes in the input value, and then returns the modified value. Remember that this value is now of size (width/2.0, height/2.0).  The ksize parameter modifies the size of the values incorporated in the pool. The strides in this case will usually be the same size of the kernel, so each maximum value depends on a wholly independent 2 by 2 square. 

<pre><code>
x_image = tf.reshape(x, [-1,28,28,1])
</code></pre>

First step in the network is to make sure that the image is in the right shape. These Images are fed into the network as a flat array. Reshape puts them back into the shape of the image. The first dimension, –1, is a placeholder, because again we are not specifying the total number of images that we are putting into the network. The next two dimensions are the width and height of the image, 28 by 28. The last dimension is the number of channels in the image. Since these images are grey-scale, this value is 1. For a color image, this value will be 3 for the red, green, and blue values. The next step will be defining the first convolutional layer.

<pre><code>
W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
</code></pre>

First we can use the functions we defined earlier to create the weights and bias variables for this layer. The first two dimensions, 5 by 5, is the dimension of the kernel for this convolution.  The next dimension is the number of channels in the image, which again is one. The final dimension is the number of feature maps you would like to produce, in this case 32. The bias needs to only be one dimension, the size of the feature map. The output of the layer is h_conv1, uses the activation function RELU, or rectifier linear unit.  Then we pool the output. 

<pre><code>
h_pool1 = max_pool_2x2(h_conv1)

	This pattern will be repeated again. 

W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])
h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
 
h_pool2 = max_pool_2x2(h_conv2)
</code></pre>

The only structural difference is the dimensions on the weights and bias. For the weights, the kernel size remains the same, 5 by 5. Now, the second dimension needs to be the size of the feature maps declared in the last layer, 32. The last dimension again is the number of feature maps you want to generate from this layer, in this case 64. Now, once we have the output of this second layer, we need to send this into a fully connected layer. Currently the output is still two dimensional, so we need to flatten it to fit into the network.

<pre><code>
h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
</code></pre>

The first dimension is -1, a placeholder for the number of images. The second dimension is the total number of values in the last feature map. We’ve pooled twice, so the map is of size width/4 by height/4, in this case 7 by 7. So the total number of values is the area of an individual map, 7 multiplied by 7, multiplied by the number of maps, 64. 

<pre><code>
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
</code></pre>

The weight variable takes the dimensions of the input, and the number of neurons you want in this layer. The bias variable has a single dimension, the number of neurons. Before the final softmax layer, we add in a drop out layer.

<pre><code>
keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
</code></pre>

This layer is relatively straightforward. The keep_prob is left as a placeholder because we will want to be able to control this easily later. 

<pre><code>
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])
y_conv = tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
</code></pre>

The final outputs, y_conv, will be ten different values, one for each digit, between zero and one. Now we need to tell the network how to handle these outputs. First, we need to define how we are going to measure the accuracy of the network.

<pre><code>
correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
</code></pre>

The correct_prediction statement is basically a Boolean defined as whether the maximum values in the predicated value and the actual value are at the same index for every value in y_conv and y_. To compute the accuracy, the Booleans are all converted to 1.0’s and 0.0’s. Then it calculates the average of these values, effectively calculating the success rate. 

<pre><code>
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y_conv), reduction_indices=[1]))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
</code></pre>

The error is calculated using categorical cross entropy. The train_step defines how the network trains itself to minimize the measure of error. 1e-4 is the step size of the optimizer. Now all we need to do is actually run the model.
<pre><code>
with tf.Session() as sess:
     sess.run(tf.initialize_all_variables())
     for i in range(20000):
         batch = mnist.train.next_batch(50)
         if i%100 == 0:
             train_accuracy = accuracy.eval(feed_dict={
                 x: batch[0], y_: batch[1], keep_prob: 1.0})
             print("step %d, training accuracy %g"%(i, train_accuracy))
         train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})
                           
     print("test accuracy %g"%accuracy.eval(feed_dict={
             x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))
</code></pre>

This will print the accuracy every 100 steps and the test accuracy after 20,000 steps. The mnist.train.next_batch(50) is the built in way for getting batches from this dataset. You will have to replace this with your own batching function that will return a variable amount of train data. train_accuracy is just the accuracy on the training batch. This generated by running the .eval method on the accuracy metric we had just defined. We only print this out every hundred layers. In order to actually train the model, we have to run train_step statement. We do this for all 20,000 of the epochs. Finally, we print the accuracy on the test set. Again, minst.test.images and mnist.test.labels are built in functions that you will have to replace with your own. Make sure to se the keep_prob to 1.0 any time you are testing the model, to ensure that there is no unintentional drop out.

##Project

<ol>
<li>Create a data handing program that will accomplish the following things
<ol>
<li>Load the images and labels from a file</li>

<li>Split the data into randomized training and testing sets</li>

<li>Convert the images into a numpy array</li>
</ol></li>

<li>Easily obtain batches of specific size for both the train and test</li>

<li>Create a network with three convolutional layers and one fully connected layer that can handle images from your dataset. Feel free to use the example above as a base. Pick some arbitrary, but reasonable values for your variables</li>

<li>Record your results for the initial network. Begin to start to experiment with various different network structures and variable values. Record your results from all of the experiments and visualize them.</li>

<li>Write a short report about what you found to be the optimal structure along with the effects you found various changes had on your network. Try to explain the reasons behind any differences you observed. Include graphs to give a visual representation of the changes.</li>
</ol>

