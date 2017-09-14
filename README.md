# TFI

## Use any TensorFlow model in a single line of code

TFI provides a simple Python interface to any TensorFlow model. It does this by automatically generating a Python class on the fly.

Here's an example of using TFI with a SavedModel based on [Inception v1](https://github.com/tensorflow/models/blob/master/slim/nets/inception_v1.py). This particular SavedModel has a single `predict` method and a [SignatureDef](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/protobuf/meta_graph.proto) that looks something like: `predict(images float <1,224,224,3>) -> (categories string <1001>, scores float <1,1001>)`

### TFI in Action
Using TFI to create a Python class for a model is a single line of code.
```python
>>> import tfi
>>>
>>> InceptionV1 = tfi.saved_model.as_class("./inception-v1.saved_model")
```

### Passing in data

TFI can automatically adapt any data you provide to the shape expected by the graph. Let's take a random photo of a dog I found on the internet...

![dog](https://www.royalcanin.com/~/media/Royal-Canin/Product-Categories/dog-medium-landing-hero.ashx)

```python
>>> model = InceptionV1()
>>> image = tfi.data.file("./dog-medium-landing-hero.jpg")
>>> result = model.predict(images=[image])
>>> categories, scores = result.categories, result.scores[0]
```

If we print the top 5 probabilities, we see:
```python
>>> # Top-5 probabilities
>>> [(scores[i], categories[i].decode('utf-8')) for i in scores.argsort()[:-5:-1]]
[(0.80796158, 'bloodhound, sleuthhound'),
 (0.10305813, 'English foxhound'),
 (0.064740285, 'redbone'),
 (0.009166114, 'beagle')]
```

Not bad!

### Image data
The `tf.data.file` function uses [mimetypes](https://docs.python.org/3.6/library/mimetypes.html) to discover the right data decoder to use. If an input to a graph is an image mimetype, TFI will automatically decode and resize the image to the proper size. In the example above, the jpeg image of a dog is automatically decoded and resized to 224x224.

### Batches
If you look closely at the example code above, you'll see that the images argument is actually an array. The class generated by TFI is smart enough to convert an array of images to an appropriately sized batch of Tensors.

### Graphs with variables
Each instance of the class has separate variables from other instances. If a graph's variables are mutated during a session in a useful way, you can continue to use those mutations by calling methods again on that same instance.

If you'd like to have multiple instances that do not interfere with one another, you can create a second instance and call methods on each of them separately.

### TFI and SavedModels

TFI uses the information in a SavedModel's SignatureDefs to generate methods on the resulting class. The keyword argument names for each method are also pulled from info in the SignatureDef.

The SavedModel used in the example was created using the [`tf.estimator.Estimator#export_savedmodel`](https://www.tensorflow.org/api_docs/python/tf/estimator/Estimator#export_savedmodel) function.

### Future work

Adapting `tfi.data` functions to handle queues and datasets wouldn't require much effort. If this is something you'd like me to do, please [file an issue](https://github.com/ajbouh/tfi/issues/new) with your specific use case!

Extending `tfi.data` to support more formats is also quite straightforward. [File an issue](https://github.com/ajbouh/tfi/issues/new) with a specific format you'd like to see. For bonus points, include the expected tensor dtype and shape. For double bonus points, include a way for me to test it in a real model. :)

#### Acknowledgements
If you're curious, the photo used above was from [a random Google image search](https://goo.gl/images/UNNf2W).
