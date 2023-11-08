# Build Graphene from 0 to 1

Note: the guideline is based on the graphene ( https://github.com/gramineproject/graphene/tree/2fdb529f81e839ef1d9638362c2c02a4e34af79f ) All the instruction is according to the documentation. ~/Documentation

The basic information of linux is above:

kernel version:5.4.72 (already install the patch), sgx-driver version:2.6.0

python :3.6.9, meson :0.61.5, ninja:1.8.2 

## Step 1: Clone the repository from github 

```shell
git clone https://github.com/gramineproject/graphene/tree/2fdb529f81e839ef1d9638362c2c02a4e34af79f
# select the commit, we use commit 2fdb529f81e839ef1d9638362c2c02a4e34af79f
cd graphene 
git checkout 2fdb529f81e839ef1d9638362c2c02a4e34af79f 
git checkout -b xxx
#then you might use git log to assure you're under this commit to develop
```

## Step 2: Check the version of the SGX, whether it's satisfied on your Linux or not.



```shell
#find SGX-Driver first
#On NUC00, the location of Linux-SGX-Driver is under /root, so you might use sudo to search
 (sudo) find / -name 'sgx.h'
 basically, the sgx-driver contains these headers like :'sgx.h', 'sgx_user.h'...
 We could use that to locate the root folder of linux-sgx-driver
 #If satisfied, continue
```



## Step 3: Check the prequisition 

### 1.Run the following command on Ubuntu to install dependencies

```shell

    sudo apt-get install -y \
        build-essential \
        autoconf \
        bison \
        gawk \
        meson \
        python3-click \
        python3-jinja2
```

### 2.For GDB support and to run all tests locally you also need to install::

```shell
    sudo apt-get install -y python3-pyelftools python3-pytest libunwind8

```

### 3.Dependencies for SGX

The build of Graphene with SGX support requires the corresponding SGX software
infrastructure to be installed on the system. In particular, the FSGSBASE
functionality must be enabled in the Linux kernel, the Intel SGX driver must be
running, and Intel SGX SDK/PSW/DCAP must be installed. In the future, when all
required SGX infrastructure is upstreamed in Linux and popular Linux
distributions, the prerequisite steps will be significantly simplified.

#### 3.1 Required packages

""""""""""""""""""""
Run the following commands on Ubuntu to install SGX-related dependencies::

```shell
   sudo apt-get install -y \
        libcurl4-openssl-dev \
        libprotobuf-c-dev \
        protobuf-c-compiler \
        python3-pip \
        python3-protobuf
    python3 -m pip install toml>=0.10
```

#### 3.2 Install the Linux kernel patched with FSGSBASE (If the kernel version is under 5.9)

Because the kernel version of NUC00  is 5.4.72, which is under the requirements. Thus, we need to the patch to fit it.

```shell
# you might use this shell to check your kernel version
uname -r
```

![1699338331289](E:\CS\work\ImpulseTech\11_6_Week3\assets\1699338331289.png)



```shell
 #you could also verify at first, if it works, it shall display like this.
 LD_SHOW_AUXV=1 /bin/true | grep AT_HWCAP2
```

![1699337000485](E:\CS\work\ImpulseTech\11_6_Week3\assets\1699337000485.png) **It works!**

**"""""""""""""""""""""""""""""""""""""""""""""""""**

FSGSBASE is a feature in recent processors which allows direct access to the FS

and GS segment base addresses. For more information about FSGSBASE and its

benefits, see `this discussion <https://lwn.net/Articles/821719>`__.

FSGSBASE patchset was merged in 5.9. For older kernels it is available as

`separate patches <https://github.com/oscarlab/graphene-sgx-driver/tree/master/fsgsbase_patches>`__.

The following instructions to patch and compile a Linux kernel with FSGSBASE

support below are written around Ubuntu 18.04 LTS (Bionic Beaver) with a Linux

5.4 LTS stable kernel but can be adapted for other distros as necessary. These

instructions ensure that the resulting kernel has FSGSBASE support and up to

date security mitigations.

\#. Clone the repository with patches::

​       git clone https://github.com/oscarlab/graphene-sgx-driver

\#. Setup a build environment for kernel development following `the instructions

   in the Ubuntu wiki <https://wiki.ubuntu.com/KernelTeam/GitKernelBuild>`__.

   Clone Linux version 5.4 via::

```shell
       git clone --single-branch --branch linux-5.4.y \

​           https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git

​       cd linux
```

\#. Apply the provided FSGSBASE patches to the kernel source tree::

```shell
       git am <graphene-sgx-driver>/fsgsbase_patches/*.patch
```
   The conversation regarding this patchset can be found in the kernel mailing

   list archives `here

   <https://lore.kernel.org/lkml/20200528201402.1708239-1-sashal@kernel.org>`__.

\#. Build and install the kernel following `the instructions in the Ubuntu wiki

   <https://wiki.ubuntu.com/KernelTeam/GitKernelBuild>`__.

\#. After rebooting, verify the patched kernel is the one that has been booted

   and is running::

   ```shell
uname -r
   ```


\#. Also verify that the patched kernel supports FSGSBASE (the below command

   must return that bit 2 is set)::

```shell
 LD_SHOW_AUXV=1 /bin/true | grep AT_HWCAP2
```

![1699337000485](E:\CS\work\ImpulseTech\11_6_Week3\assets\1699337000485.png) **It works!**

After the patched Linux kernel is installed, you may proceed with installations

of other SGX software infrastructure: the Intel SGX Linux driver, the Intel SGX

SDK/PSW, and Graphene itself (see next steps). Note that older versions of

these software packages may not work with recent Linux kernels like 5.4. We

recommend to use commit ``b7ccf6f`` of the Intel SGX Linux Driver for Intel SGX

DCAP and commit ``0e71c22`` of the Intel SGX SDK/PSW.

## Step 4: Build 

We assume the linux has already installed SGX-Driver

 ```shell
  #The pattern is like:
#to build graphene-sgx:
   make SGX=1 ISGX_DRIVER_PATH=<path-to-sgx-driver-sources>
   #So the sample instruction is like: 
  sudo  make SGX=1 ISGX_DRIVER_PATH=/root/individual/hello2mao/linux-sgx-driver
#ISGX_DRIVER_PATH="" to use the default in-kernel driver's C header.

 ```

Then use meson 

```shell
    meson build -Dsgx=enabled -Ddirect=disabled
    ninja -C build
    sudo ninja -C build install
```

![img](https://api2.mubu.com/v3/document_image/632a502b-cf26-4caa-a423-52acf37403d0-19922945.jpg)

Then  generate the key:

```shell
   openssl genrsa -3 -out enclave-key.pem 3072
```

Now, you may use the graphene! They are installed /usr/local/bin (default) :

![1699337836368](E:\CS\work\ImpulseTech\11_6_Week3\assets\1699337836368.png)

## Step 5: Test Hello World

The Sample Code is under the folder: /graphene/LibOS/shim/test/regression

```shell
      cd LibOS/shim/test/regression
      make SGX=1
      make SGX=1 sgx-tokens
      graphene-sgx helloworld
```

Then run the sample application!

![img](https://api2.mubu.com/v3/document_image/a5c38892-bf42-4d98-9bbf-0104107c981c-19922945.jpg)

## Step 6: Change the mode

```shell
(sudo)make SGX=1 clean #sudo, only if you're not permitted
    make SGX=1 DEBUG=1
#   sudo rm -rf build  #only if you've already used meson build command.
    meson build -Dsgx=enabled  -Ddirect=disabled
    ninja -C build
    sudo ninja -C build install
```

Then 

```shell
    cd LibOS/shim/test/regression
    make SGX=1
    make SGX=1 sgx-tokens

#To run Graphene with GDB, use the Graphene loader (``graphene-sgx``) and specify

    GDB=1 graphene-sgx [application] [arguments]
    GDB=1 graphene-sgx helloworld
```



The debug mode test file:

![img](https://api2.mubu.com/v3/document_image/dae60f0b-ba42-4665-aacc-6f285ae5f32a-19922945.jpg)