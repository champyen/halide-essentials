# Chapter 2 - Setup Your Halide Language Environment

Before writing Halide langugae applications we have to setup our development environment. In this book, we use Ubuntu 21.04 as our default platform.
The offical Github page of Halide provides the building/installation procedure. However the flow is a little complicated. In this chapter a simpler flow is provided.

---

## 2.1 Install Compilers
Since Halide-language is an DSL embedded in C++, C++ compiler is required for development. Here we use Clang by default.
In a ubuntu system, gcc/g++ is the default compilers for C/C++. You can install essential building environment and clang compiler suite easily with the command:
```
$ sudo apt install build-essential clang
```

---

## 2.2 Install Halide Headers and Libraries
There are two major ways to install and setup Halide environment:
1. **Using release binary tarball directly.**
2. **Build Halide environment from source.**

---

### 2.2.1 Using Release Tarball Directly
You can get official releases tarball packages from the following URL:
```
https://github.com/halide/Halide/releases/
```
Currently, the latest released version is 12.0.1.
Therefore, let's download the release with following command:
```
$ wget https://github.com/halide/Halide/releases/download/v12.0.1/Halide-12.0.1-x86-64-linux-5dabcaa9effca1067f907f6c8ea212f3d2b1d99a.tar.gz
```

Then we need to extract to a directory, you can select a folder you like to extract the tarball but in this book we use **~/halide.**.
```
$ mkdir -p ~/halide
$ tar zxvf Halide-12.0.1-x86-64-linux-5dabcaa9effca1067f907f6c8ea212f3d2b1d99a.tar.gz -C ~/halide
```

Here let's take of look of the folder structure of Halide package.
There are four directories under ~/halide/Halide-12.0.1-x86-64-linux/:
* bin - the Halide-related tools
* lib - static and shared libraries used for Halide application development
* include - the C/C++ Halide headers used for Halide application development
* share - Halide documents

For path consistency to **"Build Halide environment from source"** part, let's create a symbolic link "~/halide-sdk" which points to the folder:
```
ln -sf ~/halide/Halide-12.0.1-x86-64-linux ~/halide-sdk
```

---

### 2.2.2 Build Halide environment from source
Altough we have installed Clang compiler, but we still needs extra components to build Halide development environments:
* LLVM
* Git
* Zlib

You may notice that LLVM is also built from source in the Halide offical document. But for convenience, we install LLVM binary pacakges directly. Besides, we also need Git tool to fetch latest Halide source code from Github. And Halide libraries will be linked with Libz.

Please install LLVM, Git and Zlib by the command:
```plaintext
$ sudo apt install llvm git zlib1g-dev
```

Now, Let's fetch the Halide source code from official git repo:
```
$ git clone https://github.com/halide/Halide.git
```

And then prepare the folder for the building:
```
$ mkdir halide-sdk
$ cd halide-sdk
```

We have to specify the following environment variables before building:
```
$ export LLVM_CONFIG=llvm-config
$ export CLANG=clang
```

After all steps are done, Halide development environment can be built by the command:
```
$ make -f ../Halide/Makefile -j$(nproc)
```

As the building finished, it will also have corresponding four folders in the released binary package:
```plaintext
$ ls
bin  distrib  include  lib
```

---

### 2.2.3 Windows

Of course, you can install Halide files by following official instructions in your windows machine. But using Windows Subsystem for Linux (WSL) for Halide development is highly recommanded. Since you can keep your Windows VS environment clean and LLVM building is not needed. Most important of all is that you can setup linux container image to provide portable Halide environment across Linux and Windows. (Singularity is highly recommanded.) It is a huge advantage over dedicated Windows Halide environment.

## 2.3 Extra libraries for PNG and JPEG image formats

To make use of Halide's image tool for JPEG/PNG manipulation, we need to install libpng and libjpeg. Please install the

```plaintext
apt-get install libpng-dev libjpeg-dev
```

## 2.4 Working directory

In this book, all the excutables of examples are built under **~/halide-sdk/example**

```
$ mkdir -p ~/halide-sdk/example
$ cd ~/halide-sdk/example
```

## 2.5 Singularity container

You can download the following pre-configured ubuntu 21.04 singularity container image to save your time:
```
https://github.com/champyen/NOT_AVAILABLE_YET
```

Please init your environemt by following commmands:
```
$ mkdir -p ~/halide-sdk/example
$ cd ~/halide-sdk
$ ln -sf /opt/halide-sdk/* ./
```
