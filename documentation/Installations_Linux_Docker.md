# Installing OpenVINO&trade; Model Server for Linux using Docker Container

## Introduction

 OpenVINO&trade; Model Server is a serving system for machine learning models.  OpenVINO&trade; Model Server  makes it easy to deploy new algorithms and experiments, while keeping the same server architecture and APIs. This guide will help you install  OpenVINO&trade; Model Server for Linux through docker containers.

## System Requirements

### Hardware 
* 6th to 10th generation Intel® Core™ processors and Intel® Xeon® processors
* Intel® Neural Compute Stick 2
* Intel® Vision Accelerator Design with Intel® Movidius™ VPUs

### Operating Systems

* Ubuntu 18.04.x long-term support (LTS), 64-bit

### Overview 

This guide provides step-by-step instructions on how to install OpenVINO&trade; Model Server for Linux using Docker Container including a Quick Start guide. Links are provided for different compatible hardwares. Following guiinstructions are covered in this:

- [Installing OpenVINO&trade; Model Server with existing Docker Container](link)
- [Quick Start Guide for OpenVINO&trade; Model Server](link)
- [Building the OpenVINO&trade; Model Server Image using Source Code](link)
- [Starting Docker Container with a Single Model](link)
- [Starting docker container with a configuration file](link)
- [Starting docker container with NCS](link)
- [Starting docker container with HDDL](link)
- [Starting docker container with GPU](link)



## 1. Installing OpenVINO&trade; Model Server with existing Docker Container

A quick start guide to install model server and run it with face detection model is provided below. It includes scripts to query the gRPC endpoints and save results.

For additional endpoints, refer the REST API (add link)

```bash
 # Pull the latest version of OpenVINO&trade; Model Server - 
docker pull openvino/ubuntu18_model_server:latest

#  Download or model files into a separate directory
curl --create-dirs https://download.01.org/opencv/2020/openvinotoolkit/2020.2/open_model_zoo/models_bin/3/face-detection-retail-0004/FP32/face-detection-retail-0004.xml https://download.01.org/opencv/2020/openvinotoolkit/2020.2/open_model_zoo/models_bin/3/face-detection-retail-0004/FP32/face-detection-retail-0004.bin -o model/face-detection-retail-0004.xml -o model/face-detection-retail-0004.bin

#  Start the container serving gRPC on port 9000 
docker run -d -v $(pwd)/model:/models/face-detection/1 -e LOG_LEVEL=DEBUG -p 9000:9000 openvino/ubuntu18_model_server /ie-serving-py/start_server.sh ie_serving model --model_path /models/face-detection --model_name face-detection --port 9000  --shape auto

#  Download the example client script to test and run the gRPC 
curl https://raw.githubusercontent.com/openvinotoolkit/model_server/master/example_client/client_utils.py -o client_utils.py https://raw.githubusercontent.com/openvinotoolkit/model_server/master/example_client/face_detection.py -o face_detection.py  https://raw.githubusercontent.com/openvinotoolkit/model_server/master/example_client/client_requirements.txt -o client_requirements.txt

# Download an image to be analyzed
curl --create-dirs https://raw.githubusercontent.com/openvinotoolkit/model_server/master/example_client/images/people/people1.jpeg -o images/people1.jpeg

# Install pyhton client dependencies
pip install -r client_requirements.txt

#  Create a directory for results
mkdir results

# Run inference and store results in the newly created folder
python face_detection.py --batch_size 1 --width 600 --height 400 --input_images_dir images --output_dir results
```

## Detailed steps to install OpenVINO&trade; Model Server using Docker container

### Install Docker

Install Docker for Ubuntu 18.04 using the following link

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### Pulling OpenVINO&trade; Model Server image

After Docker installation you can pull the OpenVINO&trade; Model Server image. Open Terminal and run following command:

```
docker pull openvino/ubuntu18_model_server:latest
```

### Running the OpenVINO&trade; Model Server image

Run the OpenVINO&trade; Model Server by running the following commnad: 


docker run -d -v $(pwd)/model:/models/${MODEL_NAME}/${VERSION_NUMBER} -e LOG_LEVEL=DEBUG -p 9000:9000 -p 9001:9001 openvino/ubuntu18_model_server /ie-serving-py/start_server.sh ie_serving model --model_path ${MODEL_PATH} --model_name ${MODEL_NAME} --port 9000 --rest_port 9001

- Publish the container's port to your host's **open ports**
- In above command port 9000 is exposed for gRPC and port 9001 is exposed for REST API calls.
- For preparing and saving models to serve with OpenVINO&trade; Model Server refer [this](link)
- Add a name to your model for the client gRPC/REST API calls.

### Other Arguments

Additional arguments that can be used while running the docker image:

```
-h, --help            show this help message and exit
  --model_name MODEL_NAME
                        name of the model
  --model_path MODEL_PATH
                        absolute path to model,as in tf serving
  --batch_size BATCH_SIZE
                        sets models batchsize, int value or auto. This
                        parameter will be ignored if shape is set
  --shape SHAPE         sets models shape (model must support reshaping). If
                        set, batch_size parameter is ignored.
  --port PORT           gRPC server port
  --rest_port REST_PORT
                        REST server port, the REST server will not be started
                        if rest_port is blank or set to 0
  --model_version_policy MODEL_VERSION_POLICY
                        model version policy
  --grpc_workers GRPC_WORKERS
                        Number of workers in gRPC server. Default: 1
  --rest_workers REST_WORKERS
                        Number of workers in REST server - has no effect if
                        rest port not set. Default: 1
  --nireq NIREQ         Number of parallel inference requests for model. Default: 1
  --target_device TARGET_DEVICE
                        Target device to run the inference, default: CPU
  --plugin_config PLUGIN_CONFIG
                        A dictionary of plugin configuration keys and their values

               
```

More details about starting container with one model and examples can be found [here](link to singlemodel)
## 2. Building the OpenVINO&trade; Model Server Image using Source Code

The OpenVINO&trade; Model Server Image can be built from source. 
- Download the source code with following command:

```
git clone https://github.com/openvinotoolkit/model_server.git
```
- To build the docker image from source code use following command:

```
cd model_server

docker build -f Dockerfile --build-arg http_proxy=<http_proxy> --build-arg https_proxy=<https_proxy> -t ie-serving-py:latest .
```

- Test the built image by running it with your saved models:
```
docker run -d -v `pwd`/model:/models -e LOG_LEVEL=DEBUG -p 9000:9000 ie-serving-py:latest \
/ie-serving-py/start_server.sh ie_serving model --model_path /models --model_name ${MODEL_NAME} --port 9000  --shape auto

```


- OpenVINO&trade; Model Server docker image can be built using various Dockerfiles:
- [Dockerfile](https://github.com/openvinotoolkit/model_server/blob/master/Dockerfile) - based on ubuntu with [apt-get package](https://docs.openvinotoolkit.org/latest/_docs_install_guides_installing_openvino_apt.html) 
- [Dockerfile_clearlinux](../Dockerfile_clearlinux) - [clearlinux](https://clearlinux.org/) based image with [DLDT package](https://github.com/clearlinux-pkgs/dldt) included
- [Dockerfile_binary_openvino](../Dockerfile_binary_openvino) - ubuntu image based on Intel Distribution of OpenVINO&trade; [toolkit package](https://software.intel.com/en-us/openvino-toolkit)

- The last option requires URL to OpenVINO Toolkit package that you can get after registration on [OpenVINO&trade; Toolkit website](https://software.intel.com/en-us/openvino-toolkit/choose-download).
Use this URL as an argument in the `make` command as shown in example below:

```bash
make docker_build_bin dldt_package_url=<url-to-openvino-package-after-registration>/l_openvino_toolkit_p_2020.1.023_online.tgz
```
or
```bash
make docker_build_apt_ubuntu
```
or
```bash
make docker_build_ov_base
```
or
```bash
make docker_build_clearlinux
```

> **Note:** You can use also publicly available docker image from [dockerhub](https://hub.docker.com/r/intelaipg/openvino-model-server/)
based on clearlinux base image.

```bash
docker pull intelaipg/openvino-model-server
```



## 3. Starting Docker Container with a Single Model

- When the models are ready and stored in correct folders structure, you are ready to start the Docker container with the 
OpenVINO&trade; model server. To enable just a single model, you _do not_ need any extra configuration file, so this process can be 
completed with just one command like below:

```bash
docker run --rm -d  -v /models/:/opt/ml:ro -p 9001:9001 -p 8001:8001 ie-serving-py:latest \
/ie-serving-py/start_server.sh ie_serving model --model_path /opt/ml/model1 --model_name my_model --port 9001 --rest_port 8001
```

* option `-v` defines how the models folder should be mounted inside the docker container.

* option `-p` exposes the model serving port outside the docker container.

* `ie-serving-py:latest` represent the image name which can be different depending the tagging and building process.

* `start_server.sh` script activates the python virtual environment inside the docker container.

* `ie_serving` command starts the model server which has the following parameters:

```bash
usage: ie_serving model [-h] --model_name MODEL_NAME --model_path MODEL_PATH
                        [--batch_size BATCH_SIZE] [--shape SHAPE]
                        [--port PORT] [--rest_port REST_PORT]
                        [--model_version_policy MODEL_VERSION_POLICY]
                        [--grpc_workers GRPC_WORKERS]
                        [--rest_workers REST_WORKERS] [--nireq NIREQ]
                        [--target_device TARGET_DEVICE]
                        [--plugin_config PLUGIN_CONFIG]


```


- The model path could be local on docker container like mounted during startup or it could be Google Cloud Storage path 
in a format `gs://<bucket>/<model_path>`. In this case it will be required to 
pass GCS credentials to the docker container,
unless GKE kubernetes cluster, which handled the authorization automatically,
 is used.

- Below is an example presenting how to start docker container with a support for GCS paths to the models. The variable 
`GOOGLE_APPLICATION_CREDENTIALS` contain a path to GCP authentication key. 

```bash
docker run --rm -d  -p 9001:9001 ie-serving-py:latest \
-e GOOGLE_APPLICATION_CREDENTIALS=“${GOOGLE_APPLICATION_CREDENTIALS}”  \
-v ${GOOGLE_APPLICATION_CREDENTIALS}:${GOOGLE_APPLICATION_CREDENTIALS}
/ie-serving-py/start_server.sh ie_serving model --model_path gs://bucket/model_path --model_name my_model --port 9001
```

Learn [more about GCP authentication](https://cloud.google.com/docs/authentication/production).


- It is also possible to provide paths to models located in S3 compatible storage in a format `s3://<bucket>/<model_path>`. 

- In this case it is necessary to provide credentials to bucket by setting environmental variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. You can also set `AWS_REGION` variable, although it's not always required. 

- Incase of custom storage server compatible with S3, set `S3_ENDPOINT` environmental variable in a HOST:PORT format. 

- In an example below you can see how to start docker container serving single model located in S3.

```bash
docker run --rm -d  -p 9001:9001 ie-serving-py:latest \
-e AWS_ACCESS_KEY_ID=“${AWS_ACCESS_KEY_ID}”  \
-e AWS_SECRET_ACCESS_KEY=“${AWS_SECRET_ACCESS_KEY}”  \
-e AWS_REGION=“${AWS_REGION}”  \
-e S3_ENDPOINT=“${S3_ENDPOINT}”  \
/ie-serving-py/start_server.sh ie_serving model --model_path 
s3://bucket/model_path --model_name my_model --port 9001 --batch_size auto --model_version_policy '{"all": {}}'
```


If you need to expose multiple models, you need to create a model server configuration file, which is explained in the following section.

## 4. Starting docker container with a configuration file

Model server configuration file defines multiple models, which can be exposed for clients requests.
It uses `json` format as shown in the example below:

```json
{
   "model_config_list":[
      {
         "config":{
            "name":"model_name1",
            "base_path":"/opt/ml/models/model1",
            "batch_size": "16"
         }
      },
      {
         "config":{
            "name":"model_name2",
            "base_path":"/opt/ml/models/model2",
            "batch_size": "auto",
            "model_version_policy": {"all": {}}
         }
      },
      {
         "config":{
            "name":"model_name3",
            "base_path":"gs://bucket/models/model3",
            "model_version_policy": {"specific": { "versions":[1, 3] }},
            "shape": "auto"
         }
      },
      {
         "config":{
             "name":"model_name4",
             "base_path":"s3://bucket/models/model4",
             "shape": {
                "input1": "(1,3,200,200)",
                "input2": "(1,3,50,50)"
             },
             "plugin_config": {"CPU_THROUGHPUT_STREAMS": "CPU_THROUGHPUT_AUTO"}
         }
      },
      {
         "config":{
             "name":"model_name5",
             "base_path":"s3://bucket/models/model5",
             "shape": "auto",
             "nireq": 32,
             "target_device": "HDDL",
         }
      }
   ]
}

```
It has a mandatory array `model_config_list`, which includes a collection of `config` objects for each served model. 
Each config object includes values for the model `name` and the `base_path` attributes.

When the config file is present, the docker container can be started in a similar manner as a single model. Keep in mind that models with cloud storage path require specific environmental variables set. Configuration file above contains both GCS and S3 paths so starting docker container supporting all those models can be done with:

```bash
docker run --rm -d  -v /models/:/opt/ml:ro -p 9001:9001 -p 8001:8001 ie-serving-py:latest \
-e GOOGLE_APPLICATION_CREDENTIALS=“${GOOGLE_APPLICATION_CREDENTIALS}”  \
-v ${GOOGLE_APPLICATION_CREDENTIALS}:${GOOGLE_APPLICATION_CREDENTIALS}  \
-e AWS_ACCESS_KEY_ID=“${AWS_ACCESS_KEY_ID}”  \
-e AWS_SECRET_ACCESS_KEY=“${AWS_SECRET_ACCESS_KEY}”  \
-e AWS_REGION=“${AWS_REGION}”  \
-e S3_ENDPOINT=“${S3_ENDPOINT}”  \
/ie-serving-py/start_server.sh ie_serving config --config_path /opt/ml/config.json --port 9001 --rest_port 8001
```

Below is the explanation of the `ie_serving config` parameters
```bash
usage: ie_serving config [-h] --config_path CONFIG_PATH [--port PORT]
                         [--rest_port REST_PORT] [--grpc_workers GRPC_WORKERS]
                         [--rest_workers REST_WORKERS]

optional arguments:
  -h, --help            show this help message and exit
  --config_path CONFIG_PATH
                        absolute path to json configuration file
  --port PORT           gRPC server port
  --rest_port REST_PORT
                        REST server port, the REST server will not be started
                        if rest_port is blank or set to 0
  --grpc_workers GRPC_WORKERS
                        Number of workers in gRPC server
  --rest_workers REST_WORKERS
                        Number of workers in REST server - has no effect if
                        rest_port is not set

```

## 5. Starting docker container with NCS

Plugin for [Intel® Movidius™ Neural Compute Stick](https://software.intel.com/en-us/neural-compute-stick), starting from 
version 2019 R1.1 is distributed both in a binary package and [source code](https://github.com/opencv/dldt). 
You can build the docker image of OpenVINO Model Server, including Myriad plugin, using any form of the OpenVINO toolkit distribution:
- `make docker_build_bin dldt_package_url=<url>` 
- `make docker_build_apt_ubuntu`

Neural Compute Stick must be visible and accessible on host machine. You may need to update udev 
rules:
<details>
<summary><i>Updating udev rules</i></summary>
</br>

1. Create file __97-usbboot.rules__ and fill it with:

```
   SUBSYSTEM=="usb", ATTRS{idProduct}=="2150", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1" 
   SUBSYSTEM=="usb", ATTRS{idProduct}=="2485", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
   SUBSYSTEM=="usb", ATTRS{idProduct}=="f63b", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
```   
2. In the same directory execute following: 
 ```
   sudo cp 97-usbboot.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   sudo ldconfig
   rm 97-usbboot.rules
```

</details>
</br>

3. To start server with NCS you can use command similar to:

```
docker run --rm -it --net=host -u root --privileged -v /opt/model:/opt/model -v /dev:/dev -p 9001:9001 \
ie-serving-py:latest /ie-serving-py/start_server.sh ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device MYRIAD
```

`--net=host` and `--privileged` parameters are required for USB connection to work properly. 

`-v /dev:/dev` mounts USB drives.

A single stick can handle one model at a time. If there are multiple sticks plugged in, OpenVINO Toolkit 
chooses to which one the model is loaded. 

## 6. Starting docker container with HDDL

Plugin for High-Density Deep Learning (HDDL) accelerators based on [Intel Movidius Myriad VPUs](https://www.intel.ai/intel-movidius-myriad-vpus/#gs.xrw7cj).
is distributed only in a binary package. You can build the docker image of OpenVINO Model Server, including HDDL plugin
, using OpenVINO toolkit binary distribution:
- `make docker_build_bin dldt_package_url=<url>` 

In order to run container that is using HDDL accelerator, _hddldaemon_ must
 run on host machine. It's  required to set up environment 
 (the OpenVINO package must be pre-installed) and start _hddldaemon_ on the
  host before starting a container. Refer to the steps from [OpenVINO documentation](https://docs.openvinotoolkit.org/latest/_docs_install_guides_installing_openvino_docker_linux.html#build_docker_image_for_intel_vision_accelerator_design_with_intel_movidius_vpus).

To start server with HDDL you can use command similar to:

```
docker run --rm -it --device=/dev/ion:/dev/ion -v /var/tmp:/var/tmp -v /opt/model:/opt/model -p 9001:9001 \
ie-serving-py:latest /ie-serving-py/start_server.sh ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device HDDL
```

`--device=/dev/ion:/dev/ion` mounts the accelerator.

`-v /var/tmp:/var/tmp` enables communication with _hddldaemon_ running on the
 host machine

Check out our recommendations for [throughput optimization on HDDL](performance_tuning.md#hddl-accelerators)

## 7. Starting docker container with GPU

The GPU plugin uses the Intel® Compute Library for Deep Neural Networks (clDNN) to infer deep neural networks.
It employs for inference execution Intel® Processor Graphics including Intel® HD Graphics and Intel® Iris® Graphics

Before using GPU as OVMS target device, you need to install the required drivers. Next, start the docker container
with additional parameter `--device /dev/dri` to pass the device context and set OVMS parameter `--target_device GPU`.
The command example is listed below:

```
docker run --rm -it --device=/dev/dri -v /opt/model:/opt/model -p 9001:9001 \
ie-serving-py:latest /ie-serving-py/start_server.sh ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device GPU
```

