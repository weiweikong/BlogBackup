title: Setup AWS EC2 with Tensorflow and Caffe for Deep Learning
date: 2016-09-26 19:55:44
tags:
---

## 1. AWS EC2 的建立

- AMI 选择

  `Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-48db9d28`

- GPU Instance 选择

  目前只有`g2.2xlarge`是最廉价的方案，里面的硬盘空间最大为60g

- 因此需要添加 EBS 硬盘来扩充空间

  - Root - `/dev/sda1` 60GB
  - ebs - `/dev/sdb` 200GB

## 2. Access EC2 through `ssh`

- 使用`ssh`连接系统

  - weiwei_0903.pem 是下载到本地一个目录的key
  - 然后执行下面语句

  ```
  ssh -i "weiwei_0903.pem" ubuntu@ec2-52-53-235-35.us-west-1.compute.amazonaws.com
  ```

  - 让 know_host 记住这个IP地址即可

## 3. 加载 EBS 到刚才建立的 GPU Instance

- 查看EBS是不是存在

  ```
  ubuntu@ip-*-*-*-*:~$ lsblk
  NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
  xvda    202:0    0     8G  0 disk 
  `-xvda1 202:1    0     8G  0 part /
  xvdb    202:16   0   100G  0 disk /home/ubuntu/workspace
  ```

  - 其中 xvdb 是我单独添加的 EBS 硬盘。在最初，`MOUNTPOINT`下的`/home/ubuntu/workspace`应该是没有的，可以通过下面的步骤完成。

- 查询EBS是否已经有 File System

  ```
  [ec2-user ~]$ sudo file -s /dev/xvdb
  /dev/xvdb: data
  ```

  - 返回值是`data`意味着这个device目前没有文件系统，需要进一步格式化

    ```
    [ec2-user ~]$ sudo mkfs -t ext4 /dev/xvdb
    ```

  - 再次查看

    ```
    ec2-user ~]$ sudo file -s /dev/xvdb
    /dev/xvdb: Linux rev 1.0 ext4 filesystem data, UUID=1701d228-e1bd-4094-a14c-8c64d6819362 (needs journal recovery) (extents) (large files) (huge files)
    ```

- 挂载格式化好的device到当前目录

  ```
  ubuntu@ip-*-*-*-*:~$ sudo mount /dev/xvdb workspace
  ubuntu@ip-*-*-*-*:~$ lsblk
  NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
  xvda    202:0    0     8G  0 disk 
  `-xvda1 202:1    0     8G  0 part /
  xvdb    202:16   0   100G  0 disk /home/ubuntu/workspace
  ```

  - 此时就可以看到`xvdb`这个硬盘的挂载点。

- 添加权限

  ```
  $ sudo chmod go+rw workspace
  ```

- 关于其他Storage

  ```
  /dev/sda = /dev/xvda in the instance 8Gb "EBS persistent storage"
  /dev/sdb = /dev/xvdb in the instance 400Gb "Non persistent storage"
  ```

- 查看 Storage

  ```
  df -h
  ```

  ​

## 4. 安装相关软件

- 基本依赖库

  ```shell
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get install -y build-essential git python-pip libfreetype6-dev libxft-dev libncurses-dev libopenblas-dev gfortran python-matplotlib libblas-dev liblapack-dev libatlas-base-dev python-dev python-pydot linux-headers-generic linux-image-extra-virtual unzip python-numpy swig python-pandas python-sklearn unzip wget pkg-config zip g++ zlib1g-dev
  sudo pip install -U pip
  ```

- ~~Install Python2.7, Anconda, CUDA 7.5.178, CUDNN 7.0, Tensorflow 0.10.0~~

  ```shell
  git clone https://gist.github.com/weiweikong/374e93d9ccb88ea45341268a06897259 aws-tensorflow-python2.7-setup
  ```

  - ~~注意给bash文件权限~~

- Set a folder to `/mnt`

  ```shell
  # stop on error
  set -e
  ############################################
  # install into /mnt/bin
  sudo mkdir -p /mnt/bin
  sudo chown ubuntu:ubuntu /mnt/bin
  ```

- Install Anaconda

  ```shell
  wget https://repo.continuum.io/archive/Anaconda2-4.1.1-Linux-x86_64.sh
  bash Anaconda2-4.1.1-Linux-x86_64.sh -b -p /mnt/bin/anaconda2
  rm Anaconda2-4.1.1-Linux-x86_64.sh

  echo 'export PATH="/mnt/bin/anaconda2/bin:$PATH"' >> ~/.bashrc
  ```

  ​

- Install Required Packages

  ```sh
  # install the required packages
  sudo apt-get update && sudo apt-get -y upgrade
  sudo apt-get -y install linux-headers-$(uname -r) linux-image-extra-`uname -r`
  ```

- Install CUDA 7.5

  ```shell
  # install cuda
  wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.5-18_amd64.deb
  sudo dpkg -i cuda-repo-ubuntu1404_7.5-18_amd64.deb
  rm cuda-repo-ubuntu1404_7.5-18_amd64.deb
  sudo apt-get update
  sudo apt-get install -y cuda
  ```

- Manually download CUDNN 7.5 and upload

  ```shell
  scp -i your_pem_file.pem cudnn-7.5-linux-x64-v5.0-ga.tgz ubuntu@ec2-54-67-18-98.us-west-1.compute.amazonaws.com:~/.
  ```

- Install cuDNN 7.5.1

  ```shell
  # get cudnn
  tar xvzf cudnn-7.5-linux-x64-v5.1.tgz
  cd cuda
  sudo cp lib64/* /usr/local/cuda/lib64/
  sudo cp include/* /usr/local/cuda/include/
  sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

  echo 'export CUDA_HOME=/usr/local/cuda
  export CUDA_ROOT=/usr/local/cuda
  export PATH=$PATH:$CUDA_ROOT/bin:$HOME/bin
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_ROOT/lib64
  ' >> ~/.bashrc
  ```

- Install Tensorflow with only cuDNN 7.5.1 and Python 2.7

  ```shell
  export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.10.0-cp27-none-linux_x86_64.whl
  /mnt/bin/anaconda2/bin/pip install $TF_BINARY_URL
  ```

- Install Caffe 依赖库

  ```shell
  sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
  sudo apt-get install --no-install-recommends libboost-all-dev
  sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
  ```

- 配置 Caffe 文件并编译

  ```shell
  cp Makefile.config.example Makefile.config
  # Adjust Makefile.config (for example, if using Anaconda Python, or if cuDNN is desired)

  mkdir build
  cd build
  cmake ..

  make all
  make test
  make runtest
  ```

- Test Caffe Install

  ```shell
  sh data/mnist/get_mnist.sh
  sh examples/mnist/create_mnist.sh
  sh examples/mnist/train_lenet.sh
  ```

  ​

- Monitor Code

  ```shell
  # install monitoring programs
  sudo wget https://git.io/gpustat.py -O /usr/local/bin/gpustat
  sudo chmod +x /usr/local/bin/gpustat
  sudo nvidia-smi daemon
  sudo apt-get -y install htop
  ```

- Ref: http://www.pyimagesearch.com/2016/07/04/how-to-install-cuda-toolkit-and-cudnn-for-deep-learning/

### 4.1 Trouble Shooting

- Caffe 遇到 `locale::facet::_S_create_c_locale name not valid`

  - add `export LC_ALL="en_US.UTF-8"` to `bashrc`
  - Ref: https://gist.github.com/wangruohui/679b05fcd1466bb0937f

- Tensorflwo 遇到 `AttributeError: 'GFile' object has no attribute 'size'`

  ```shell
  sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
  ```

- `libdc1394 error: Failed to initialize libdc1394`

  ```shell
  sudo ln /dev/null /dev/raw1394
  ```

- Cafee 遇到 ` Aborted at 1458527401 (unix time) try "date -d @1458527401" if you are using GNU date`

  - Check Nvidia GPUs List

    ```shell
    $ nvidia-smi
    Mon Sep 26 18:01:53 2016       
    +------------------------------------------------------+                       
    | NVIDIA-SMI 352.63     Driver Version: 352.63         |                       
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla K40c          Off  | 0000:02:00.0     Off |                    0 |
    | 31%   69C    P0    74W / 235W |    150MiB / 11519MiB |     66%      Default |
    +-------------------------------+----------------------+----------------------+
    |   1  GeForce GT 610      Off  | 0000:81:00.0     N/A |                  N/A |
    | 40%   43C    P8    N/A /  N/A |    277MiB /  1023MiB |     N/A      Default |
    +-------------------------------+----------------------+----------------------+
    |   2  Tesla K40c          Off  | 0000:82:00.0     Off |                    0 |
    | 30%   65C    P0    68W / 235W |    113MiB / 11519MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    ```

  - Set Specific GPU Visible

    ```shell
    CUDA_VISIBLE_DEVICES=1	Only device 1 will be seen
    CUDA_VISIBLE_DEVICES=0,1	Devices 0 and 1 will be visible
    CUDA_VISIBLE_DEVICES=”0,1”	Same as above, quotation marks are optional
    CUDA_VISIBLE_DEVICES=0,2,3	Devices 0, 2, 3 will be visible; device 1 is masked
    ```

  - 目前选择两个GPU工作不会报错

  - Ref:https://github.com/BVLC/caffe/issues/1993

### 4.2 Access to EC2 using FileZilla

- Download FileZilla and setup.
- Add `.pem` key file
  - Edit -> Settings -> Connection -> SFTP
  - Select the `.pem` file and add it to the list.
- Add EC2 connection
  - File -> Site Manager
  - Host: EC2 Public IP (Could be check under EC2 Console)
  - Protocol: SFTP
  - Login Type: Normal
  - User: ubuntu
  - Password: /
- Press `Connect`