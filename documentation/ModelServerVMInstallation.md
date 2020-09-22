# Landing OpenVINO™ Model Server on Bare Metal Hosts and Virtual Machines

> **NOTES**:
> * These steps apply to Ubuntu*, CentOS* and RedHat*
> * An internet connection is required to follow the steps in this guide.

## Introduction
OpenVINO™ model server is a Python* implementation of gRPC and RESTful API interfaces defined by Tensorflow serving. In the backend it uses Inference Engine libraries from OpenVINO™ toolkit, which speeds up the execution on CPU, and enables it on FPGA and Movidius devices.

OpenVINO™ model server can be hosted on a bare metal server, virtual machine or inside a docker container. It is also suitable for landing in Kubernetes environment.

## System Requirements

#### Operating Systems 

* Ubuntu 16.04.x or higher, long-term support (LTS), 64-bit
* CentOS 7.4, 64-bit (for target only)
* Red Hat Enterprise Linux
* ClearLinux
* SUSE Linux

Download the installer from : [OpenVino Toolkit](https://software.intel.com/content/www/us/en/develop/tools/openvino-toolkit/choose-download.html) 

## Installation of dependencies 
1. Open a command prompt terminal window.

2. Install Linux packages of cpio, make, wget, python3, python setup tools and virtualenv using command : 
   ```
   sudo apt-get update && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            libgomp1 \
            python3-dev \
            python3-pip \
            virtualenv \
            usbutils \
            gnupg2
    ```

3. Download GPG public key for Intel OpenVino using the command :
    ```
    curl -o GPG-PUB-KEY-INTEL-OPENVINO-2020 https://apt.repos.intel.com/openvino/2020/GPG-PUB-KEY-INTEL-OPENVINO-2020
    ```
    And add the downloaded key to list of trusted files using the command : 
    ```
    apt-key add GPG-PUB-KEY-INTEL-OPENVINO-2020
    ```

4. Set correct environment variables PYTHONPATH and LD_LIBRARY_PATH pointing to Python modules of OpenVINO™ and compiled libraries using the command : 
   ```
   export DL_INSTALL_DIR=/opt/intel/openvino/deployment_tools && \
   export PYTHONPATH="/opt/intel/openvino/python/python3.6" && \
   export LD_LIBRARY_PATH="$DL_INSTALL_DIR/inference_engine/external/tbb/lib:$DL_INSTALL_DIR/inference_engine/external/mkltiny_lnx/lib:$DL_INSTALL_DIR/inference_engine/lib/intel64:$DL_INSTALL_DIR/ngraph/lib"
   ```


## Model Server Installation

> * To avoid avoid all conflicts between Python dependencies from other applications running on the same host, it is recommended to run OpenVINO™ model server from Python virtual environment.
> * All python dependencies are included in requirements.txt file
1. Clone model server git repository using command : 
   ``` 
   git clone https://github.com/openvinotoolkit/model_server.git
   ```

2. Naviagate to model server directory using command : 
   ``` 
   cd model_server
   ```
3. If you want to install with virtual machine, use following       command to create virtual environment and install required dependencies :  
   ``` 
   virtualenv -p python3 .venv && \
   . .venv/bin/activate && \ 
   pip3 install --upgrade pip==20.2.1 && \
   pip3 --no-cache-dir install -r requirements.txt --use-feature=2020-resolver
   ```
   And install model server using command:

   ```
    make install
   ```
4. To run without a virtual environment
    ```
    pip3 --no-cache-dir install -r requirements.txt && \
    pip3 install .
    ```
    If you want to modify the source code and easily test changes:
    ```
    pip3 --no-cache-dir install -r requirements.txt && \
    pip3 install -e .
    ```

## Starting the Serving Service
The server can be started using ```ie_serving``` command when working inside a virtual environment as follow : 
```
ie_serving --help
usage: ie_serving [-h] {config,model} ...

positional arguments:
  {config,model}  sub-command help
    config        Allows you to share multiple models using a configuration file
    model         Allows you to share one type of model

optional arguments:
  -h, --help      show this help message and exit
```
The server can be started in interactive mode, as a background process or a daemon initiated by ```systemctl/initd``` depending on the Linux distribution and specific hosting requirements.

## Using Neural Compute Sticks

OpenVINO Model Server can employ AI accelerators [Intel® Neural Compute Stick and Intel® Neural Compute Stick 2](https://software.intel.com/content/www/us/en/develop/hardware/neural-compute-stick.html).

To use Movidus Neural Compute Sticks with OpenVINO Model Server you need to have OpenVINO Toolkit with Movidius VPU support installed. In order to do that follow [OpenVINO installation instruction](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html). Don't forget about [additional steps for NCS](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html).

On server startup, you need to specify that you want to load model on Neural Compute Stick for inference execution. You can do that by setting target_device parameter to MYRIAD. If it's not specified, OpenVINO will try to load model on CPU.

Example:
```
ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device MYRIAD
```
You can also run it in [Docker container](provide-a-link)

>Note: 
>* Checkout supported configurations. Look at VPU Plugins to see if your model is supported. If not, take a look at OpenVINO Model Optimizer and convert your model to desired format.
>* A single stick can handle one model at a time. If there are multiple sticks plugged in, OpenVINO Toolkit chooses to which one the model is loaded.

## Using HDDL accelerators
OpenVINO Model Server can employ High-Density Deep Learning (HDDL) accelerators based on Intel Movidius Myriad VPUs. To use HDDL accelerators with OpenVINO Model Server you need to have OpenVINO Toolkit with Intel® Vision Accelerator Design with Intel® Movidius™ VPU support installed. In order to do that follow [OpenVINO installation instruction](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html). Don't forget about [additional steps for Intel® Movidius™ VPU](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html).

On server startup, you need to specify that you want to load model on HDDL card for inference execution. You can do that by setting target_device parameter to HDDL. If it's not specified, OpenVINO will try to load model on CPU.

Example:
```
ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device HDDL
```
Check out our recommendation for [throughput optimization on HDDL](provide-a-link)

You can also run it in [Docker container](provide-a-link)

>Note: Check out supported configurations. Look at VPU Plugins to see if your model is supported. If not, take a look at OpenVINO Model Optimizer and convert your model to desired format.


