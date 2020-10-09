# Introduction to OpenVINO™ Model Server

OpenVINO™ Model Server is a scalable, high-performance solution for serving machine learning models optimized for Intel® architectures. The server provides an inference service via gRPC(Remote Procedure Calls) endpoint or REST API  making it easy to deploy new algorithms while keeping the same server architecture and APIs.

### Features of Model Server :

* Serves multiple models, or multiple versions of the same model simultaneously.
* Supports multiple Deep Learning frameworks such as Caffe*, TensorFlow*, MXNet* and ONNX*.
* Supports multi-worker configuration and parallel inference execution.
* Supports model reshaping at runtime.
* Speeds time-to-market as development of client code is reduced
* Server can be deployed in a Kubernetes cluster allowing the inference service to scale horizontally and ensure high availability. [Kubernetes deployments](https://github.com/openvinotoolkit/model_server/tree/master/deploy)
* The server supports reshaphing models in runtime - [Model Reshaping](./ShapeandBatchSize.md)

### Model Server Components:
OpenVINO™ Model Server includes the following components:

1. Deep Learning Inference Engine
2. OpenCV 
3. Inference Engine Serving 

### Documentation Set Contents
OpenVINO™ Model Server documentation set includes the following documents:

- [Understanding Architecture of OpenVINO™ Model Server](./Architecture.md)
- [Install the OpenVINO™ Model Server on Linux using Docker](./InstallationsLinuxDocker.md)
- [Installation of OpenVINO™ Model Server with Kubernetes](./InstallationsKubernetes.md)
- [Install the  OpenVINO™ Model Server on Bare Metal Hosts and Virtual Machines](./InstallationsModelServerVMAndBareMetal.md)
- [OpenVINO™ Model Server gRPC Reference Guide](./ModelServerGRPCAPI.md)
- [OpenVINO™ Model Server REST API Reference Guide](./ModelServerRESTAPI.md)



