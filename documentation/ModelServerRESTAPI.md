# OpenVINO™ Model Server RESTful API Documentation

## Introduction
In addition with [gRPC APIs](link) OpenVino model server also supports RESTful APIs. This page describes these API endpoints and an end-to-end example on usage.

The request and response is a JSON object.

## Model Status API 
Get information about the status of served models

#### URL 

```
GET http://${REST_URL}:${REST_PORT}/v1/models/${MODEL_NAME}/versions/${MODEL_VERSION}
```
Including /versions/${MODEL_VERSION} is optional. If omitted status for all versions is returned in the response.

#### Response format
If successful, returns a JSON of following format 
```
{
  'model_version_status':[
    {
      'version': '1', 
      'state': 'AVAILABLE', 
      'status': {
        'error_code': 'OK', 
        'error_message': ''
      }
    }
  ]
}
```
#### Usage Example
```
python rest_get_model_status.py --rest_port 8000 --model_version 1
{
    "model_version_status": [
        {
            "version": "1",
            "state": "AVAILABLE",
            "status": {
                "error_code": "OK",
                "error_message": ""
            }
        }
    ]
}
```


## Model Metadata API
Get the metadata of a model in the model server.

#### URL 
```
GET http://${REST_URL}:${REST_PORT}/v1/models/${MODEL_NAME}/versions/${MODEL_VERSION}/metadata
```
Including ${MODEL_VERSION} is optional. If omitted the model metadata for the latest version is returned in the response.

#### Response format
```
{
  "modelSpec": {
    "name": "resnet",
    "version": "1"
  },
  "metadata": {
    "signature_def": {
      "@type": "type.googleapis.com/tensorflow.serving.SignatureDefMap",
      "signatureDef": {
        "serving_default": {
          "inputs": {
            "data": {
              "name": "data_2:0",
              "dtype": "DT_FLOAT",
              "tensorShape": {
                "dim": [
                  {
                    "size": "1"
                  },
                  {
                    "size": "3"
                  },
                  {
                    "size": "224"
                  },
                  {
                    "size": "224"
                  }
                ]
              }
            }
          },
          "outputs": {
            "prob": {
              "name": "prob_2:0",
              "dtype": "DT_FLOAT",
              "tensorShape": {
                "dim": [
                  {
                    "size": "1"
                  },
                  {
                    "size": "1000"
                  }
                ]
              }
            }
          },
          "methodName": "tensorflow/serving/predict"
        }
      }
    }
  }
}
```
#### Usage Example
```
python rest_get_serving_meta.py --rest_port 8000
{
  "modelSpec": {
    "name": "resnet",
    "version": "1"
  },
  "metadata": {
    "signature_def": {
      "@type": "type.googleapis.com/tensorflow.serving.SignatureDefMap",
      "signatureDef": {
        "serving_default": {
          "inputs": {
            "data": {
              "name": "data_2:0",
              "dtype": "DT_FLOAT",
              "tensorShape": {
                "dim": [
                  {
                    "size": "1"
                  },
                  {
                    "size": "3"
                  },
                  {
                    "size": "224"
                  },
                  {
                    "size": "224"
                  }
                ]
              }
            }
          },
          "outputs": {
            "prob": {
              "name": "prob_2:0",
              "dtype": "DT_FLOAT",
              "tensorShape": {
                "dim": [
                  {
                    "size": "1"
                  },
                  {
                    "size": "1000"
                  }
                ]
              }
            }
          },
          "methodName": "tensorflow/serving/predict"
        }
      }
    }
  }
}
```

## Model predict function
Sends requests via TensorFlow Serving RESTful API using images in numpy format. It displays performance statistics and optionally the model accuracy.

#### Command
```
python rest_serving_client.py --help
usage: rest_serving_client.py [-h] --images_numpy_path IMAGES_NUMPY_PATH
                              [--labels_numpy_path LABELS_NUMPY_PATH]
                              [--rest_url REST_URL] [--rest_port REST_PORT]
                              [--input_name INPUT_NAME]
                              [--output_name OUTPUT_NAME]
                              [--transpose_input {False,True}]
                              [--transpose_method {nchw2nhwc,nhwc2nchw}]
                              [--iterations ITERATIONS]
                              [--batchsize BATCHSIZE]
                              [--model_name MODEL_NAME]
                              [--request_format {row_noname,row_name,column_noname,column_name}]
                              [--model_version MODEL_VERSION]

Sends requests via TensorFlow Serving RESTfull API using images in numpy
format. It displays performance statistics and optionally the model accuracy

optional arguments:
  -h, --help            show this help message and exit
  --images_numpy_path IMAGES_NUMPY_PATH
                        numpy in shape [n,w,h,c] or [n,c,h,w]
  --labels_numpy_path LABELS_NUMPY_PATH
                        numpy in shape [n,1] - can be used to check model
                        accuracy
  --rest_url REST_URL   Specify url to REST API service. default:
                        http://localhost
  --rest_port REST_PORT
                        Specify port to REST API service. default: 5555
  --input_name INPUT_NAME
                        Specify input tensor name. default: input
  --output_name OUTPUT_NAME
                        Specify output name. default:
                        resnet_v1_50/predictions/Reshape_1
  --transpose_input {False,True}
                        Set to False to skip NHWC>NCHW or NCHW>NHWC input
                        transposing. default: True
  --transpose_method {nchw2nhwc,nhwc2nchw}
                        How the input transposition should be executed:
                        nhwc2nchw or nhwc2nchw
  --iterations ITERATIONS
                        Number of requests iterations, as default use number
                        of images in numpy memmap. default: 0 (consume all
                        frames)
  --batchsize BATCHSIZE
                        Number of images in a single request. default: 1
  --model_name MODEL_NAME
                        Define model name, must be same as is in service.
                        default: resnet
  --request_format {row_noname,row_name,column_noname,column_name}
                        Request format according to TF Serving API:
                        row_noname,row_name,column_noname,column_name
  --model_version MODEL_VERSION
                        Model version to be used. Default: LATEST
```

#### Response format
```
('Image data range:', 0, ':', 255)
Start processing:
	Model name: resnet
	Iterations: 10
	Images numpy path: imgs.npy
	Images in shape: (10, 3, 224, 224)

output shape: (1, 1000)
Iteration 1; Processing time: 57.42 ms; speed 17.41 fps
imagenet top results in a single batch:
('\t', 0, 'airliner', 404, '; Correct match.')
output shape: (1, 1000)
Iteration 2; Processing time: 57.65 ms; speed 17.35 fps
imagenet top results in a single batch:
('\t', 0, 'Arctic fox, white fox, Alopex lagopus', 279, '; Correct match.')
output shape: (1, 1000)
Iteration 3; Processing time: 59.21 ms; speed 16.89 fps
imagenet top results in a single batch:
('\t', 0, 'bee', 309, '; Correct match.')
output shape: (1, 1000)
Iteration 4; Processing time: 59.64 ms; speed 16.77 fps
imagenet top results in a single batch:
('\t', 0, 'golden retriever', 207, '; Correct match.')
output shape: (1, 1000)
Iteration 5; Processing time: 59.96 ms; speed 16.68 fps
imagenet top results in a single batch:
('\t', 0, 'gorilla, Gorilla gorilla', 366, '; Correct match.')
output shape: (1, 1000)
Iteration 6; Processing time: 59.41 ms; speed 16.83 fps
imagenet top results in a single batch:
('\t', 0, 'magnetic compass', 635, '; Correct match.')
output shape: (1, 1000)
Iteration 7; Processing time: 59.45 ms; speed 16.82 fps
imagenet top results in a single batch:
('\t', 0, 'peacock', 84, '; Correct match.')
output shape: (1, 1000)
Iteration 8; Processing time: 59.91 ms; speed 16.69 fps
imagenet top results in a single batch:
('\t', 0, 'pelican', 144, '; Correct match.')
output shape: (1, 1000)
Iteration 9; Processing time: 63.17 ms; speed 15.83 fps
imagenet top results in a single batch:
('\t', 0, 'snail', 113, '; Correct match.')
output shape: (1, 1000)
Iteration 10; Processing time: 52.59 ms; speed 19.01 fps
imagenet top results in a single batch:
('\t', 0, 'zebra', 340, '; Correct match.')

processing time for all iterations
average time: 58.30 ms; average speed: 17.15 fps
median time: 59.00 ms; median speed: 16.95 fps
max time: 63.00 ms; max speed: 15.00 fps
min time: 52.00 ms; min speed: 19.00 fps
time percentile 90: 59.40 ms; speed percentile 90: 16.84 fps
time percentile 50: 59.00 ms; speed percentile 50: 16.95 fps
time standard deviation: 2.61
time variance: 6.81
Classification accuracy: 100.00
```
## Usage example 
```Bash
python rest_serving_client.py --images_numpy_path imgs.npy --labels_numpy_path lbs.npy --input_name data --output_name prob --rest_port 8000 --transpose_input False
('Image data range:', 0, ':', 255)
Start processing:
	Model name: resnet
	Iterations: 10
	Images numpy path: imgs.npy
	Images in shape: (10, 3, 224, 224)

output shape: (1, 1000)
Iteration 1; Processing time: 57.42 ms; speed 17.41 fps
imagenet top results in a single batch:
('\t', 0, 'airliner', 404, '; Correct match.')
output shape: (1, 1000)
Iteration 2; Processing time: 57.65 ms; speed 17.35 fps
imagenet top results in a single batch:
('\t', 0, 'Arctic fox, white fox, Alopex lagopus', 279, '; Correct match.')
output shape: (1, 1000)
Iteration 3; Processing time: 59.21 ms; speed 16.89 fps
imagenet top results in a single batch:
('\t', 0, 'bee', 309, '; Correct match.')
output shape: (1, 1000)
Iteration 4; Processing time: 59.64 ms; speed 16.77 fps
imagenet top results in a single batch:
('\t', 0, 'golden retriever', 207, '; Correct match.')
output shape: (1, 1000)
Iteration 5; Processing time: 59.96 ms; speed 16.68 fps
imagenet top results in a single batch:
('\t', 0, 'gorilla, Gorilla gorilla', 366, '; Correct match.')
output shape: (1, 1000)
Iteration 6; Processing time: 59.41 ms; speed 16.83 fps
imagenet top results in a single batch:
('\t', 0, 'magnetic compass', 635, '; Correct match.')
output shape: (1, 1000)
Iteration 7; Processing time: 59.45 ms; speed 16.82 fps
imagenet top results in a single batch:
('\t', 0, 'peacock', 84, '; Correct match.')
output shape: (1, 1000)
Iteration 8; Processing time: 59.91 ms; speed 16.69 fps
imagenet top results in a single batch:
('\t', 0, 'pelican', 144, '; Correct match.')
output shape: (1, 1000)
Iteration 9; Processing time: 63.17 ms; speed 15.83 fps
imagenet top results in a single batch:
('\t', 0, 'snail', 113, '; Correct match.')
output shape: (1, 1000)
Iteration 10; Processing time: 52.59 ms; speed 19.01 fps
imagenet top results in a single batch:
('\t', 0, 'zebra', 340, '; Correct match.')

processing time for all iterations
average time: 58.30 ms; average speed: 17.15 fps
median time: 59.00 ms; median speed: 16.95 fps
max time: 63.00 ms; max speed: 15.00 fps
min time: 52.00 ms; min speed: 19.00 fps
time percentile 90: 59.40 ms; speed percentile 90: 16.84 fps
time percentile 50: 59.00 ms; speed percentile 50: 16.95 fps
time standard deviation: 2.61
time variance: 6.81
Classification accuracy: 100.00
```



