# Object Detection with TensorFlow and Confusion Matrix

This script generates a confusion matrix using the detections record file generated by the `/object_detection/inference/infer_detections.py` script.

For a full description of the format of the matrix, and how it is computed, take a look at the [Confusion Matrix in Object Detection with TensorFlow](https://www.shiftedup.com/2018/10/10/confusion-matrix-in-object-detection-api-with-tensorflow) article.

To run this script you need a couple of things:

* The label map used by your model — This is the proto-buff file that you created in order to train your model.
  
* A detection record file — This is the file generated by using the `/object_detection/inference/infer_detections.py`. This script runs a TFRecord file through your model and saves the results in a detection record file. 

Here is an example of running the script:

```
python confusion_matrix.py --detections_record=testing_detections.record --label_map=label_map.pbtxt --output_path=confusion_matrix.csv
```

The script will print the confusion matrix along with precision and recall information to the standard output and save the precision/recall results to the specified output area. 

The following error can be realised:

```
Traceback (most recent call last):
  File "confusion_matrix.py", line 133, in <module>
    tf.app.run(main)
  File "/home/ubuntu/anaconda3/lib/python3.6/site-packages/tensorflow/python/platform/app.py", line 125, in run
    _sys.exit(main(argv))
  File "confusion_matrix.py", line 128, in main
    confusion_matrix = process_detections(FLAGS.detections_record, categories)
  File "confusion_matrix.py", line 45, in process_detections
    decoded_dict = data_parser.parse(example)
  File "/home/ubuntu/tensorflow/models/research/object_detection/metrics/tf_example_parser.py", line 153, in parse
    results_dict[key] = parser.parse(tf_example)
  File "/home/ubuntu/tensorflow/models/research/object_detection/metrics/tf_example_parser.py", line 49, in parse
    self.field_name].HasField("bytes_list") else None
TypeError: sequence item 0: expected str instance, bytes found
```

The problem is with the TensorFlow library and the solution is [explained here](https://github.com/tensorflow/models/issues/3252). You'll have to patch your local copy of TensorFlow to get around it.

> I had same error with Python3.5 and TF1.5. This causes tf_example.features.feature[self.field_name].bytes_list.value returns byte type instead of string type in metrics/tf_example_parser.StringParser.
> 
> So I changed tf_example_parser.StringParser below
Find `metrics/tf_example_parser`, and make the change:
```
class StringParser(data_parser.DataToNumpyParser):
  """Tensorflow Example string parser."""
  def __init__(self, field_name):
    self.field_name = field_name
  def parse(self, tf_example):
    if tf_example.features.feature[self.field_name].HasField("bytes_list"):
        result = tf_example.features.feature[self.field_name].bytes_list.value
        result = "".join([x if type(x)=='str' else x.decode('utf-8') for x in result])
    else:
        result = None
    return result
```

