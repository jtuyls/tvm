Vitis-AI Integration
====================

`Vitis-AI <https://github.com/Xilinx/Vitis-AI>`__ is Xilinx's
development stack for hardware-accelerated AI inference on Xilinx
platforms, including both edge devices and Alveo cards. It consists of
optimized IP, tools, libraries, models, and example designs. It is
designed with high efficiency and ease of use in mind, unleashing the
full potential of AI acceleration on Xilinx FPGA and ACAP.

The current Vitis-AI Byoc flow inside TVM enables acceleration of Neural
Network model inference on edge and cloud. The identifiers for the
supported edge and cloud Deep Learning Processor Units (DPU's) are
DPUCZDX8G respectively DPUCADX8G. DPUCZDX8G and DPUCADX8G are hardware
accelerators for convolutional neural networks (CNN's) on top of the
Xilinx `Zynq Ultrascale+
MPSoc <https://www.xilinx.com/products/silicon-devices/soc/zynq-ultrascale-mpsoc.html>`__
respectively
`Alveo <https://www.xilinx.com/products/boards-and-kits/alveo.html>`__
(U200/U250) platforms. For more information about the DPU identifiers
see the section on `DPU naming information <#dpu-naming-information>`__.

On this page you will find information on how to
`build <#build-instructions>`__ TVM with Vitis-AI and on how to `get
started <#getting-started>`__ with an example.

DPU naming information
----------------------

+---------------------------------+-----------------+-------------------------------------------------------------------------+------------------------------------------------------------+---------------------------------------------------+--------------------------------------------------------------------------+
| DPU                             | Application     | HW Platform                                                             | Quantization Method                                        | Quantization Bitwidth                             | Design Target                                                            |
+=================================+=================+=========================================================================+============================================================+===================================================+==========================================================================+
| Deep Learning Processing Unit   | C: CNN R: RNN   | AD: Alveo DDR AH: Alveo HBM VD: Versal DDR with AIE & PL ZD: Zynq DDR   | X: DECENT I: Integer threshold F: Float threshold R: RNN   | 4: 4-bit 8: 8-bit 16: 16-bit M: Mixed Precision   | G: General purpose H: High throughput L: Low latency C: Cost optimized   |
+---------------------------------+-----------------+-------------------------------------------------------------------------+------------------------------------------------------------+---------------------------------------------------+--------------------------------------------------------------------------+

Build instructions
------------------

This section lists the instructions for building TVM with Vitis-AI for
both `cloud <#cloud-dpucadx8g>`__ and `edge <#edge-dpuczdx8g>`__.

Cloud (DPUCADX8G)
~~~~~~~~~~~~~~~~~

For Vitis-AI acceleration in the cloud TVM has to be built on top of the
Xilinx Alveo platform.

System requirements
^^^^^^^^^^^^^^^^^^^

The following table lists system requirements for running docker
containers as well as Alveo cards.

+-----------------------------------------------------+----------------------------------------------------------+
| **Component**                                       | **Requirement**                                          |
+=====================================================+==========================================================+
| Motherboard                                         | PCI Express 3.0-compliant with one dual-width x16 slot   |
+-----------------------------------------------------+----------------------------------------------------------+
| System Power Supply                                 | 225W                                                     |
+-----------------------------------------------------+----------------------------------------------------------+
| Operating System                                    | Ubuntu 16.04, 18.04                                      |
+-----------------------------------------------------+----------------------------------------------------------+
|                                                     | CentOS 7.4, 7.5                                          |
+-----------------------------------------------------+----------------------------------------------------------+
|                                                     | RHEL 7.4, 7.5                                            |
+-----------------------------------------------------+----------------------------------------------------------+
| CPU                                                 | Intel i3/i5/i7/i9/Xeon 64-bit CPU                        |
+-----------------------------------------------------+----------------------------------------------------------+
| GPU (Optional to accelerate quantization)           | NVIDIA GPU with a compute capability > 3.0               |
+-----------------------------------------------------+----------------------------------------------------------+
| CUDA Driver (Optional to accelerate quantization)   | nvidia-410                                               |
+-----------------------------------------------------+----------------------------------------------------------+
| FPGA                                                | Xilinx Alveo U200 or U250                                |
+-----------------------------------------------------+----------------------------------------------------------+
| Docker Version                                      | 19.03.1                                                  |
+-----------------------------------------------------+----------------------------------------------------------+

Hardware setup and docker build
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Clone the Vitis AI repository:
   ::


   git clone --recurse-submodules https://github.com/Xilinx/Vitis-AI
   
2. Install Docker, and add the user to the docker group. Link the user
   to docker installation instructions from the following docker's
   website:

   -  https://docs.docker.com/install/linux/docker-ce/ubuntu/
   -  https://docs.docker.com/install/linux/docker-ce/centos/
   -  https://docs.docker.com/install/linux/linux-postinstall/

3. Any GPU instructions will have to be separated from Vitis AI.
4. Set up Vitis AI to target Alveo cards. To target Alveo cards with
   Vitis AI for machine learning workloads, you must install the
   following software components:

   -  Xilinx Runtime (XRT)
   -  Alveo Deployment Shells (DSAs)
   -  Xilinx Resource Manager (XRM) (xbutler)
   -  Xilinx Overlaybins (Accelerators to Dynamically Load - binary
      programming files)

   While it is possible to install all of these software components
   individually, a script has been provided to automatically install
   them at once. To do so:

   -  Run the following commands:
   ::
   
   
      cd Vitis-AI/alveo/packages
      sudo su
      ./install.sh
      
   -  Power cycle the system.
   
5. Clone tvm repo and pyxir repo
   ::
   
   
      git clone --recursive https://github.com/apache/incubator-tvm.git
      git clone --recursive https://github.com/Xilinx/pyxir.git
   
6. Build and start the tvm runtime Vitis-AI Docker Container.
   ::


      bash incubator-tvm/docker/build.sh ci_vai bash
      bash incubator-tvm/docker/bash.sh tvm.ci_vai
	  
      #Setup inside container
      source /opt/xilinx/xrt/setup.sh
      . $VAI_ROOT/conda/etc/profile.d/conda.sh
      conda activate vitis-ai-tensorflow
      
7. Install PyXIR
   ::



     cd pyxir
     python3 setup.py install --use_vai_rt_dpucadx8g --user

   
8. Build TVM inside the container with Vitis-AI
   ::


      cd incubator-tvm
      mkdir build
      cp cmake/config.cmake build
      cd build  
      echo set\(USE_LLVM ON\) >> config.cmake
      echo set\(USE_VITIS_AI ON\) >> config.cmake
      cmake ..
      make -j$(nproc)
   
9.  Install TVM
    ::
      cd incubator-tvm/python
      pip3 install -e . --user
      
Edge (DPUCZDX8G)
^^^^^^^^^^^^^^^^


For edge deployment we make use of two systems referred to as host and
edge. The `host <#host-requirements>`__ system is responsible for
quantization and compilation of the neural network model in a first
offline step. Afterwards, the model will de deployed on the
`edge <#edge-requirements>`__ system.

Host requirements
^^^^^^^^^^^^^^^^^

The following table lists system requirements for running the TVM -
Vitis-AI docker container.

+-----------------------------------------------------+----------------------------------------------+
| **Component**                                       | **Requirement**                              |
+=====================================================+==============================================+
| Operating System                                    | Ubuntu 16.04, 18.04                          |
+-----------------------------------------------------+----------------------------------------------+
|                                                     | CentOS 7.4, 7.5                              |
+-----------------------------------------------------+----------------------------------------------+
|                                                     | RHEL 7.4, 7.5                                |
+-----------------------------------------------------+----------------------------------------------+
| CPU                                                 | Intel i3/i5/i7/i9/Xeon 64-bit CPU            |
+-----------------------------------------------------+----------------------------------------------+
| GPU (Optional to accelerate quantization)           | NVIDIA GPU with a compute capability > 3.0   |
+-----------------------------------------------------+----------------------------------------------+
| CUDA Driver (Optional to accelerate quantization)   | nvidia-410                                   |
+-----------------------------------------------------+----------------------------------------------+
| FPGA                                                | Not necessary on host                        |
+-----------------------------------------------------+----------------------------------------------+
| Docker Version                                      | 19.03.1                                      |
+-----------------------------------------------------+----------------------------------------------+

Host setup and docker build
^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Clone tvm repo
::
   git clone --recursive https://github.com/apache/incubator-tvm.git
2. Build and start the tvm runtime Vitis-AI Docker Container.
::
   cd incubator-tvm 
   bash incubator-tvm/docker/build.sh ci_vai bash
   bash incubator-tvm/docker/bash.sh tvm.ci_vai
  
   #Setup inside container
   . $VAI_ROOT/conda/etc/profile.d/conda.sh
   conda activate vitis-ai-tensorflow
   
3. Install PyXIR
::


   git clone --recursive https://github.com/Xilinx/pyxir.git
   cd pyxir
   sudo python3 setup.py install
   
   
4. Build TVM inside the container with Vitis-AI.
::
   cd incubator-tvm 
   mkdir build
   cp cmake/config.cmake build
   cd build
   echo set\(USE_LLVM ON\) >> config.cmake
   echo set\(USE_VITIS_AI ON\) >> config.cmake
   cmake ..
   make -j$(nproc)
   
5. Install TVM
::
    cd incubator-tvm/python
    pip3 install -e . --user

Edge requirements
^^^^^^^^^^^^^^^^^

The DPUCZDX8G can be deployed on the `Zynq Ultrascale+
MPSoc <https://www.xilinx.com/products/silicon-devices/soc/zynq-ultrascale-mpsoc.html>`__
platform. The following development boards can be used out-of-the-box:

+--------------------+----------------------+-----------------------------------------------------------------------+
| **Target board**   | **TVM identifier**   | **Info**                                                              |
+====================+======================+=======================================================================+
| Ultra96            | DPUCZDX8G-ultra96    | https://www.xilinx.com/products/boards-and-kits/1-vad4rl.html         |
+--------------------+----------------------+-----------------------------------------------------------------------+
| ZCU104             | DPUCZDX8G-zcu104     | https://www.xilinx.com/products/boards-and-kits/zcu104.html           |
+--------------------+----------------------+-----------------------------------------------------------------------+
| ZCU102             | DPUCZDX8G-zcu102     | https://www.xilinx.com/products/boards-and-kits/ek-u1-zcu102-g.html   |
+--------------------+----------------------+-----------------------------------------------------------------------+

Edge hardware setup
^^^^^^^^^^^^^^^^^^^

1. Download the Pynq v2.5 image for your target (use Z1 or Z2 for
   Ultra96 target depending on board version) Link to image:
   https://github.com/Xilinx/PYNQ/releases/tag/v2.5
2. Follow Pynq instructions for setting up the board: `pynq
   setup <https://pynq.readthedocs.io/en/latest/getting_started.html>`__
3. After connecting to the board, make sure to run as root. Execute
   ``su``
4. Set up DPU on Pynq by following the steps here: `DPU Pynq
   setup <https://github.com/Xilinx/DPU-PYNQ>`__
5. Run the following command to download the DPU bitstream:

   ::


     python3 -c 'from pynq_dpu import DpuOverlay ; overlay = DpuOverlay("dpu.bit")'
  
6. Check whether the DPU kernel is alive:
   ::


     dexplorer -w

Edge TVM setup
^^^^^^^^^^^^^^

Building TVM depends on the Xilinx
`PyXIR <https://github.com/Xilinx/pyxir>`__ package. PyXIR acts as an
interface between TVM and Vitis-AI tools.

1. First install the PyXIR h5py and pydot dependencies:
::


   apt-get install libhdf5-dev
   pip3 install pydot h5py
2. Install PyXIR
::


   git clone --recursive https://github.com/Xilinx/pyxir.git
   cd pyxir
   sudo python3 setup.py install --use_vai_rt_dpuczdx8g
   
3. Build TVM with Vitis-AI
::


   git clone --recursive https://github.com/apache/incubator-tvm
   cd incubator-tvm
   mkdir build
   cp cmake/config.cmake build
   cd build
   echo set\(USE_VITIS_AI ON\) >> config.cmake
   cmake ..     
   make
   
4. Install TVM
::
      cd incubator-tvm/python
      pip3 install -e . --user

5. Check whether the setup was successful in the Python shell:
::


   python3 -c 'import pyxir; import tvm'


Getting started
---------------

This section shows how to use TVM with Vitis-AI. For this it's important
to understand that neural network models are quantized for Vitis-AI
execution in fixed point arithmetic. The approach we take here is to
quantize on-the-fly using the first N inputs as explained in the next
section.

On-the-fly quantization
~~~~~~~~~~~~~~~~~~~~~~~

Usually, to be able to accelerate inference of Neural Network models
with Vitis-AI DPU accelerators, those models need to quantized upfront.
In TVM - Vitis-AI flow, we make use of on-the-fly quantization to remove
this additional preprocessing step. In this flow, one doesn't need to
quantize his/her model upfront but can make use of the typical inference
execution calls (module.run) to quantize the model on-the-fly using the
first N inputs that are provided (see more information below). This will
set up and calibrate the Vitis-AI DPU and from that point onwards
inference will be accelerated for all next inputs. Note that the edge
flow deviates slightly from the explained flow in that inference won't
be accelerated after the first N inputs but the model will have been
quantized and compiled and can be moved to the edge device for
deployment. Please check out the `edge <#Edge%20usage>`__ usage
instructions below for more information.

Config/Settings
~~~~~~~~~~~~~~~

A couple of environment variables can be used to customize the Vitis-AI
Byoc flow.

+----------------------------+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Environment Variable**   | **Default if unset**                   | **Explanation**                                                                                                                                                                                                                                                                                                                            |
+============================+========================================+============================================================================================================================================================================================================================================================================================================================================+
| PX\_QUANT\_SIZE            | 128                                    | The number of inputs that will be used for quantization (necessary for Vitis-AI acceleration)                                                                                                                                                                                                                                              |
+----------------------------+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PX\_BUILD\_DIR             | Use the on-the-fly quantization flow   | Loads the quantization and compilation information from the provided build directory and immediately starts Vitis-AI hardware acceleration. This configuration can be used if the model has been executed before using on-the-fly quantization during which the quantization and comilation information was cached in a build directory.   |
+----------------------------+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Cloud usage
~~~~~~~~~~~

This section shows how to accelerate a convolutional neural network
model in TVM with Vitis-AI on the cloud.

To be able to target the Vitis-AI cloud DPUCADX8G target we first have
to import the target in PyXIR. This PyXIR package is the interface being
used by TVM to integrate with the Vitis-AI stack. Additionaly, import
the typical TVM and Relay modules and the Vitis-AI contrib module inside
TVM.

::

    import pyxir
    import pyxir.contrib.target.DPUCADX8G

    import tvm
    import tvm.relay as relay
    from tvm.contrib.target import vitis_ai
    from tvm.relay.build_module import bind_params_by_name
    from tvm.relay.op.contrib.vitis_ai import annotation

After importing a convolutional neural network model using the usual
Relay API's, annotate the Relay expression for the given Vitis-AI DPU
target and partition the graph.

::

    mod["main"] = bind_params_by_name(mod["main"], params)
    mod = annotation(mod, params, target)
    mod = relay.transform.MergeCompilerRegions()(mod)
    mod = relay.transform.PartitionGraph()(mod)

Now, we can build the TVM runtime library for executing the model. The
TVM target is 'llvm' as the operations that can't be handled by the DPU
are executed on the CPU. The Vitis-AI target is DPUCADX8G as we are
targeting the cloud DPU and this target is passed as a config to the TVM
build call.

::

    tvm_target = 'llvm'
    target='DPUCADX8G'

    with tvm.transform.PassContext(opt_level=3, config= {'target_': target}):   
        graph, lib, params = relay.build(mod, tvm_target, params=params)

As one more step before we can accelerate a model with Vitis-AI in TVM
we have to quantize and compile the model for execution on the DPU. We
make use of on-the-fly quantization for this. Using this method one
doesn’t need to quantize their model upfront and can make use of the
typical inference execution calls (module.run) to calibrate the model
on-the-fly using the first N inputs that are provided. After the first N
iterations, computations will be accelerated on the DPU. So now we will
feed N inputs to the TVM runtime module. Note that these first N inputs
will take a substantial amount of time.

::

    module = tvm.contrib.graph_runtime.create(graph, lib, tvm.cpu())
    module.set_input(**params)

    # First N (default = 128) inputs are used for quantization calibration and will
    # be executed on the CPU
    # This config can be changed by setting the 'PX_QUANT_SIZE' (e.g. export PX_QUANT_SIZE=64)
    for i in range(128):
        module.set_input(input_name, inputs[i]) 
        module.run()

Afterwards, inference will be accelerated on the DPU.

::

    module.set_input(name, data)
    module.run()

To save and load the built module, one can use the typical TVM API's:

::

    # save the graph, lib and params into separate files
    from tvm.contrib import util

    temp = util.tempdir()
    path_lib = temp.relpath("deploy_lib.so")
    lib.export_library(path_lib)
    with open(temp.relpath("deploy_graph.json"), "w") as fo:
        fo.write(graph)
    with open(temp.relpath("deploy_param.params"), "wb") as fo:
        fo.write(relay.save_param_dict(params))

Load the module from compiled files and run inference

::

    # load the module into memory
    loaded_json = open(temp.relpath("deploy_graph.json")).read()
    loaded_lib = tvm.runtime.load_module(path_lib)
    loaded_params = bytearray(open(temp.relpath("deploy_param.params"), "rb").read())

    module = tvm.contrib.graph_runtime.create(loaded_json, loaded_lib, ctx)
    module.load_params(loaded_params)
    module.set_input(name, data)
    module.run()

Edge usage
~~~~~~~~~~

This section shows how to accelerate a convolutional neural network
model in TVM with Vitis-AI at the edge. The first couple of steps will
have to be run on the host machine and take care of quantization and
compilation for deployment at the edge.

Host steps
^^^^^^^^^^

To be able to target the Vitis-AI cloud DPUCZDX8G target we first have
to import the target in PyXIR. This PyXIR package is the interface being
used by TVM to integrate with the Vitis-AI stack. Additionaly, import
the typical TVM and Relay modules and the Vitis-AI contrib module inside
TVM.

::

    import pyxir
    import pyxir.contrib.target.DPUCZDX8G

    import tvm
    import tvm.relay as relay
    from tvm.contrib.target import vitis_ai
    from tvm.relay.build_module import bind_params_by_name
    from tvm.relay.op.contrib.vitis_ai import annotation

After importing a convolutional neural network model using the usual
Relay API's, annotate the Relay expression for the given Vitis-AI DPU
target and partition the graph.

::

    mod["main"] = bind_params_by_name(mod["main"], params)
    mod = annotation(mod, params, target)
    mod = relay.transform.MergeCompilerRegions()(mod)
    mod = relay.transform.PartitionGraph()(mod)

Now, we can build the TVM runtime library for executing the model. The
TVM target is 'llvm' as the operations that can't be handled by the DPU
are executed on the CPU. At this point that means the CPU on the host.
The Vitis-AI target is DPUCZDX8G-zcu104 as we are targeting the edge DPU
on the ZCU104 board and this target is passed as a config to the TVM
build call. Note that different identifiers can be passed for different
targets, see `edge targets info <#edge-requirements>`__.

::

    tvm_target = 'llvm'
    target='DPUCZDX8G-zcu104'

    with tvm.transform.PassContext(opt_level=3, config= {'target_': target}):   
        graph, lib, params = relay.build(mod, tvm_target, params=params)

Additionaly, already build the deployment module for the ARM CPU target
and serialize:

::

    # Export lib for aarch64 target

    tvm_target = tvm.target.arm_cpu('ultra96')
    lib_kwargs = {
        'fcompile': contrib.cc.create_shared,
        'cc': "/usr/aarch64-linux-gnu/bin/ld"
    }

    with tvm.transform.PassContext(opt_level=3,
                                   config={'target_': target,
                                           'vai_build_dir_': target + '_build'}):
        graph_arm, lib_arm, params_arm = relay.build(
            mod, tvm_target, params=params)

    lib_dpuv2.export_library('tvm_dpu_arm.so', **lib_kwargs)
    with open("tvm_dpu_arm.json","w") as f:
        f.write(graph_dpuv2)
    with open("tvm_dpu_arm.params", "wb") as f:
        f.write(relay.save_param_dict(params_dpuv2))

As one more step before we can deploy a model with Vitis-AI in TVM at
the edge we have to quantize and compile the model for execution on the
DPU. We make use of on-the-fly quantization on the host machine for
this. This involves using the TVM inference calls (module.run) to
quantize the model on the host using N inputs. After providing N inputs
we can then move the TVM and Vitis-AI build files to the edge device for
deployment.

::

    module = tvm.contrib.graph_runtime.create(graph, lib, tvm.cpu())
    module.set_input(**params)

    # First N (default = 128) inputs are used for quantization calibration and will
    # be executed on the CPU
    # This config can be changed by setting the 'PX_QUANT_SIZE' (e.g. export PX_QUANT_SIZE=64)
    for i in range(128):
        module.set_input(input_name, inputs[i]) 
        module.run()

Now, move the TVM build files (tvm\_dpu\_arm.json, tvm\_dpu\_arm.so,
tvm\_dpu\_arm.params) and the DPU build directory (e.g.
DPUCZDX8G-zcu104\_build) to the edge device. For information on setting
up the edge device check out the `edge setup <#edge-dpuczdx8g>`__
section.

Edge steps
^^^^^^^^^^

The following steps will have to be executed on the edge device after
setup and moving the build files from the host.

Move the target build directory to the same folder where the example
running script is located and explicitly set the path to the build
directory using the PX\_BUILD\_DIR environment variable.

::

    export PX_BUILD_DIR={PATH-TO-DPUCZDX8G-BUILD_DIR}

Then load the TVM runtime module into memory and feed inputs for
inference.

::

    # load the module into memory
    loaded_json = open(temp.relpath("tvm_dpu_arm.json")).read()
    loaded_lib = tvm.runtime.load_module("tvm_dpu_arm.so")
    loaded_params = bytearray(open(temp.relpath("tvm_dpu_arm.params"), "rb").read())

    module = tvm.contrib.graph_runtime.create(loaded_json, loaded_lib, ctx)
    module.load_params(loaded_params)
    module.set_input(name, data)
    module.run()


