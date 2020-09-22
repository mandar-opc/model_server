# RESTful API Documentation

In addition with [gRPC APIs](link) OpenVino model server also supports RESTful APIs. This page describes these API endpoints and an end-to-end example on usage.

The request and response is a JSON object.

## Model Status API 
Get information about the status of served models

#### URL 

```
GET http://${REST_URL}:${REST_PORT}/v1/models/${MODEL_NAME}${MODEL_VERSION}
```
Including ${MODEL_VERSION} is optional. If omitted status for all versions is returned in the response.

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


## Model Metadata API
Get the metadata of a model in the model server.

#### URL 
```
GET http://${REST_URL}:${REST_PORT}/v1/models/${MODEL_NAME}${MODEL_VERSION}/metadata
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


## Model predict function 

