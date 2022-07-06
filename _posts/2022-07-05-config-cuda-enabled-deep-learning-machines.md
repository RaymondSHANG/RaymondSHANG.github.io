---
layout: post
title: "Config cuda enabled deep learning machines"
subtitle: "version matches matters"
date: 2022-07-05 10:57:38
header-style: text
catalog: true
author: "Yuan"
tags: [cuda, cuDNN, cudatoolkits, pytorch, tensorflow]
---
{% include linksref.html %}
> 勿以恶小而为之，勿以善小而不为

I spent the whole weekend trying to install a cuda-enabled deep learning machine. Similar to most of other tasks, if you are in the right way, you could finish them in 30 mins. But if you are not familiar with them, after try, error and google search, you need a whole weekend!
Some hinders in my case as of July 5th,2022, listed below:

1. Version matters! You need to find the latest version of pytorch and tersorflow that support the same cuda version. Then, you need make sure your NVIDIA driver support your cuda version. minor: You will also need make sure cuDNN version matches with cuda version.
2. conda install is good for most of python packages, and probably most of pytorch subversions, <b>except 1.12 bond with cuda 11.6</b>. I think the image from conda is cpuonly version, as shown by `print(torch.__version__)`, which returns "+cpu". I checked the raw files from anaconda (website)[https://anaconda.org/pytorch/pytorch/files], the files looks good. Then, I checked cudatoolkit, it should from 'conda-forge', rather than 'anaconda'. Still, there are some problems. By the way, the pytorch.1.12-cuda11.6.tar.bz2 is only 1.2G, which is smaller than pytorch1.8-cuda11.1 (1.5G), and the whl file for the same version(2.?G).
3. To install pytorch here, you shall use <b>pip</b> instead.
4. The pytorch1.12-cuda11.6 is 1.2GB, the cudatoolkits is ~900MB. If our internet connection is not stable, the installation failed. After several tries, I mannually downloaded the required tar.bz2 files(for conda, although still failed, as the conda installation itself failed) and whl(for pip) files.
5. You could also set time:

```bash
#For conda
conda config --set
conda config --set remote_connect_timeoutsecs 10000
remote_read_timeoutsecs 10000
#For pip
pip --default-timeout=10000 install xxx

```

# General steps to config a pytorch cuda dl platform

A good updated tutorial [here by CtrlZ1](https://blog.csdn.net/qq_41076797/article/details/116448817)， and [here by 米饭的白色
](https://blog.csdn.net/mifangdebaise/article/details/124404955)

Currently, I choose cuda11.6+cudnn8.4.1+pytorch for both my windows and ubuntu system.

A common steps for both installations:

1. Install the Nvidia driver version510.
2. Install cuda version 11.6
3. Check nvcc and nvidia-smi return the same cuda versions, add cuda/bin to system PATH, /usr/local/cuda-11.6/lib64 to LD_LIBRARY_PATH.
4. Check cuda using deviceQuery
5. Install cudnn which support cuda11.x
6. Check cudnn and cuda by mnistCUDNN.
7. Install python or anaconda. Please, do not uninstall your python!!! All your work from step1-6 will lost, and your ubuntu broke! Use your own env after installing conda for specific python. <b>DO NOT</b> do anything to python originally installed in your system!!!
8. Install pytorch 1.12 with cuda 11.6API,using pip
9. Test

# Other tips
## 1. Remove nouveau
NVIDIA-SMI has failed because it couldn‘t communicate with the NVIDIA driver.
### check nouveau
   
```bash
lsmod | grep nouveau
```

If return nothing. Then OK.
### Remove nouveau

```bash
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo update-initramfs -u
sudo reboot
```
## 2. Uninstall cuda and nvidia drivers

```bash
sudo apt-get --purge remove "*cublas*" "cuda*" "nsight*" 
sudo apt-get --purge remove "*nvidia*"
sudo rm -rf /usr/local/cuda*
#From cuda 11.4 onwards, an uninstaller script has been provided. Use it for the uninstallation instead:
# To uninstall cuda
sudo /usr/local/cuda-11.4/bin/cuda-uninstaller 
# To uninstall nvidia
sudo /usr/bin/nvidia-uninstall
```


## 3. cuda install and cuda test
After installing cuda, add cuda/bin and lib to path, such as in .bashrc
If you have nvidia driver already installed, either uninstall it, or do not include nvidia driver when you install cuda. Remember, your cuda version should match your nvidia version.

```bash
# user-add: cuda
export PATH=/usr/local/cuda-11.6/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.6/lib64:$LD_LIBRARY_PATH
export CUDA_HOME=/usr/local/cuda
nvcc -V
```

Test your cuda in ubuntu:

```bash
git clone https://github.com/NVIDIA/cuda-samples.git
cd <sample_dir>
make
Samples/1_Utilities/deviceQuery/deviceQuery
Samples/1_Utilities/bandwidthTest/bandwidthTest
```

Under windows, you could find deviceQuery.ext, just run it in cmd.

## 4. cudnn install and cudnn test
For windows, it's direct, just unzip and merge cudnn files to cuda directories.

For ubuntu, version 8.4.1.50 install failed if I follow the install-guide from nvidia. Solution from [dishant.daredevil](https://forums.developer.nvidia.com/t/e-version-8-3-1-22-1-cuda10-2-for-libcudnn8-was-not-found/200801/9)

{{tip}} <b>From dishant.daredevil</b>
After step -<br><br>
sudo apt-key add /var/cudnn-local-repo-*/7fa2af80.pub<br><br>
You will have the directory /var/cudnn-local-repo-ubuntu2004-8.4.0.27 (with your ubuntu version and cudnn downloaded)<br><br>
Inside this directory, you will be having three .deb files.<br><br>
just do for all the deb files- <br><br>
sudo gdebi xxx.deb<br><br>
which will install cudnn. 
{{end}}

{{note}}
Note that gdebi and apt will install independence, comparing to dpkg, though both of them use dpkg as background
{{end}}

After cudnn was installed, you should find cudnn_samples_v8 in /usr/src, copy it to your home directory, so you could make, and test it.

```bash
cp -r /usr/src/cudnn_samples_v8 ~/pytorch_install
cd ~/pytorch_install/cudnn_samples_v8
cd mnistCUDNN
sudo apt-get install libfreeimage3 libfreeimage-dev
make clean
make
./mnistCUDNN
#Should get 'Test passed' to indicate cudnn configed correctly
#You could also make and run other samples in the samples_v8 directory
```

## 5. make your dlenv enviroment

```bash
conda create --name py39 python=3.9
conda activate py39
#conda install -n py39 scipy=0.17.3
#conda remove --name myenv --all
#conda env list
```

## 6. pytorch test
```bash
#after downloading 3/4 whl from https://download.pytorch.org/whl/torch_stable.html, 
#cuXXX/torchXXX-cp39xxxx-linux_x86_64.whl, or -win_amd64.whl
conda activate py39
pip install torchxxx.whl
pip install torchaudioxxx.whl
pip install torchvisionxxx.whl
pip install torchdistxxx.whl
python -c "import torch;print(torch.__version__);print(torch.cuda.is_available())"
#should return True to show pytorch successfully configured.
```
---
