
# Create your own custom and authenticated APT repository

### Prerequisites

This tutorial assumes you are using Ubuntu, and that the following packages are installed:
```
sudo apt-get install -y gcc dpkg-dev gpg
```
![install -y gcc dpkg-dev gpg](https://raw.githubusercontent.com/shamim4s/Own-Custom-APT/master/images/install-gcc-dpkg-dev-gpg.jpg)


#Install golang from apt repository 
```
sudo apt install golang-go
```
![install golang-go](https://raw.githubusercontent.com/shamim4s/Own-Custom-APT/e17e7c812b14fd4988058695740c7739e4f6fedb/images/installgolang.png)



Check golang version after install to make sure golang installed properly
```
go version
```
![go version](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/Screenshot_20230125_011746.png?raw=true)


# Step 0: Create a Simple Hello World Program

Before getting started with packaging,

let’s create a basic **hello world** program under **~/example/hello-world-program**.

### Create a Directory called "hello-world-program"

```
mkdir -p ~/example/hello-world-program
```
![mkdir -p ~/example/hello-world-program](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/make-hello-world-program-folder.png?raw=true)
 

###  you can copy and paste the following commands into your terminal to create simple **hello.go** program

```
echo 'package main

import (
        "fmt"
)

func main() {
    fmt.Println("Hello, world!")
}' > ~/example/hello-world-program/hello.go
```
![create simple hello.go](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/create-hello.go.png?raw=true)


### Then, you can compile it with:

```
cd ~/example/hello-world-program

```
![cd ~/example/hello-world-program](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/cd-hello-folder.png?raw=true)
<br/>


### Run and check its working properly or not.
```
go build hello.go
./hello
```
#### This will print **Hello, World**
![go build hello.go](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/build-go-file.png?raw=true)


# Step 1: Creating a deb Package
Debian, and Debian-based Linux distributions use .deb packages to package and distribute programs. 
To start we will create a directory in the form:

```
<package-name>_<version>-<release-number>_<architecture>
```
### where the :
* **package-name** is the name of our package, hello-world in our case,

* **version** is the version of the software, 0.0.1 in our case,
* **release-number** is used to track different releases of the same software version; it’s usually set to **1**, but hypothetically if there was an error in the packaging (e.g. a file was missed, or the description had an error in it, or a post-install script was wrong), this number would be increased to track the change, and
* **architecture** is the target architecture of the platform, amd64 in this example; however if your package is architecture-independent (e.g. a python script), then you can set this to **all**.

### So for this example, we’ll create the directory
```
mkdir -p ~/example/hello-world_0.0.1-1_amd64
```
![mkdir hello-world_0.0.1-1_amd64](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/make-hello-world-program-folder.png?raw=true)

This directory will be the root of the package. Since we want our hello-world binary to be installed system wide, we’ll have to store it under **usr/bin/hello-world** with the following commands:

```
cd ~/example/hello-world_0.0.1-1_amd64
mkdir -p usr/bin
cp ~/example/hello-world-program/hello-world usr/bin/.
```
![mkdir bin](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/mkdir-bin.jpg?raw=true)

Each package requires a control file which needs to be located under the DEBIAN directory. 

```
mkdir -p ~/example/hello-world_0.0.1-1_amd64/DEBIAN
```

### You can copy and paste the following to create one:
```
echo "Package: hello-world
Version: 0.0.1
Maintainer: Shamim <me@shamim.app>
Depends: libc6
Architecture: amd64
Homepage: http://shamim.app
Description: A program that prints hello" \
> ~/example/hello-world_0.0.1-1_amd64/DEBIAN/control
```
![Create debian and control file](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/mkdevian-create-control.png?raw=true)


Note that we’re assuming an amd64 Architecture for this tutorial, if your binary is for a different architecture, adjust accordingly. If you’re distributing a platform-independent package, you can set the architecture to all.

### To build the .deb package, run:
```
dpkg --build ~/example/hello-world_0.0.1-1_amd64
```
![build deb](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/build-deb-package.png?raw=true)


This will output a deb package under **~/example/hello-world_0.0.1.deb**.

### You can inspect the info of the deb by running:
```
dpkg-deb --info ~/example/hello-world_0.0.1-1_amd64.deb
```

which will show:
```
new Debian package, version 2.0.
size 2832 bytes: control archive=336 bytes.
    182 bytes,     7 lines      control
Package: hello-world
Version: 0.0.1
Maintainer: Shamim <me@shamim.app>
Depends: libc6
Architecture: amd64
Homepage: http://shamim.app
Description: A program that prints hello
```
![deb info](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/deb-info.png?raw=true)



### You can also view the contents by running
```
dpkg-deb --contents ~/example/hello-world_0.0.1-1_amd64.deb
```

which will show:
```
drwxrwxr-x shamim/shamim         0 2021-05-17 16:21 ./
drwxrwxr-x shamim/shamim         0 2021-05-17 16:18 ./usr/
drwxrwxr-x shamim/shamim         0 2021-05-17 16:18 ./usr/bin/
-rwxrwxr-x shamim/shamim     16696 2021-05-17 16:18 ./usr/bin/hello
```
![deb contents](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/deb%20contents.png?raw=true)



### This package can then be installed using the **-f** option under **apt-get install**
```
sudo apt install -f ~/example/hello-world_0.0.1-1_amd64.deb
```
### To check the program run hello-world by command:
```
hello
```
which should output **hello, world** respectively.

![install deb file](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/install-f-deb.png?raw=true)







### Finally, if you want to remove it, you can run:
```
sudo apt-get remove hello-world
```
![remove hello](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/apt%20remove%20hello.png?raw=true)



#### This concludes the first step of building a .deb package

# Step 2: Creating an apt Repository

In this step, we will show how to create your own apt repository which can be used to host one or more deb packages.

### Let’s start with creating a directory to hold our debs:
```
mkdir -p ~/example/apt-repo/pool/main/
```
![mkdir repo](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/mkdir%20repo%20pool%20main.png?raw=true)


### Then copy our deb(s) into this directory:
```
cp ~/example/hello-world_0.0.1-1_amd64.deb ~/example/apt-repo/pool/main/.
```
![cp deb main](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/cp%20deb%20main.png?raw=true)

Once you’ve copied in all of your debs, we will create a different directory to contain a list of all available packages and corresponding metadata
```
mkdir -p ~/example/apt-repo/dists/stable/main/binary-amd64
```
![mkdir binary amd](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/mk%20binary%20amd.png?raw=true)

If you want to support multiple architectures, make a directory above for each type (e.g. i386, amd64, etc).

### Next, we will generate a Packages file, which will contain a list of all available packages in this repository. We will use the dpkg-scanpackages program to generate it, by running:
```
cd ~/example/apt-repo
dpkg-scanpackages --arch amd64 pool/ > dists/stable/main/binary-amd64/Packages
```
![cd repo dpkg packages](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/cd%20repo%20dpkgscanpkg%20package.png?raw=true)

### If you get any error something like  LC_ALL=0 or similer, just fix this error by command :

```
sudo locale-gen "en_US.UTF-8"
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
```
### or
```
sudo dpkg-reconfigure locales
```

It’s also good practice to compress the packages file, as apt will favour downloading compressed data whenever available. Let’s do this by running:
```
cat dists/stable/main/binary-amd64/Packages | gzip -9 > dists/stable/main/binary-amd64/Packages.gz
```
![compress the packages file](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/cat%20package%20gzip.png?raw=true)


### Let’s take a quick look at the contents of the Packages file:

```
cat dists/stable/main/binary-amd64/Packages
```

```
Package: hello-world
Version: 0.0.1
Architecture: amd64
Maintainer: Shamim <me@shamim.app>
Depends: libc6
Filename: pool/main/hello-world_0.0.1-1_amd64.deb
Size: 2832
MD5sum: 3eba602abba5d6ea2a924854d014f4a7
SHA1: e300cabc138ac16b64884c9c832da4f811ea40fb
SHA256: 6e314acd7e1e97e11865c11593362c65db9616345e1e34e309314528c5ef19a6
Homepage: http://shamim.app>
Description: A program that prints hello

```
![quick look at the contents of the Packages](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/cat%20binary%20amd%20packages.png?raw=true)



### if you had multiple deb files, you would have an entry for each package

The contents of the Packages file is a list of all available packages along with metadata from the **DEBIAN/control** file, and some hashes which can be used to validate the integrity of the package.

Next we will create a Release file. Unfortunately dpkg-scanpackages does not create Release files. Some people use programs like apt-ftparchive; however in this example I’ll cover an alternative to apt-ftparchive by using a small bash script.
### let’s write some bash. Just copy and paste the following in your terminal to get a copy of the script: 

```
echo '#!/bin/sh
set -e

do_hash() {
    HASH_NAME=$1
    HASH_CMD=$2
    echo "${HASH_NAME}:"
    for f in $(find -type f); do
        f=$(echo $f | cut -c3-) # remove ./ prefix
        if [ "$f" = "Release" ]; then
            continue
        fi
        echo " $(${HASH_CMD} ${f}  | cut -d" " -f1) $(wc -c $f)"
    done
}

cat << EOF
Origin: Example Repository
Label: Example
Suite: stable
Codename: stable
Version: 1.0
Architectures: amd64 arm64 arm7
Components: main
Description: An example software repository
Date: $(date -Ru)
EOF
do_hash "MD5Sum" "md5sum"
do_hash "SHA1" "sha1sum"
do_hash "SHA256" "sha256sum"
' > ~/example/generate-release.sh && chmod +x ~/example/generate-release.sh
```
![create a Release file](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/echo%20release.png?raw=true)

### Next let’s run the generate-release.sh script with:
```
cd ~/example/apt-repo/dists/stable
~/example/generate-release.sh > Release
```
![un the generate-release.sh](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/run%20release.sh%20to%20release.png?raw=true)

At this point, you can try hosting this repo for yourself. In this example we’ll use python’s simple HTTP server; 
however in practice you’ll want to use a production-ready server. 
Here’s how you can start it up for testing:

```
cd ~/example
```
### Run python server on a specific port on this directory as a web directory
```
python3 -m http.server -b your-server-ip  8000
```
![Run python webserver](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/run%20python%20server.png?raw=true)


When server is running you need to configure this apt repository by :
```
echo "deb [trusted=yes] http://136.243.196.122:8000/apt-repo stable main" >> /etc/apt/sources.list.d/example.list
```

```
cat  /etc/apt/sources.list.d/example.list
```
![add repo list and check](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/add%20repo%20list.png?raw=true)



Then finally update apt and install our new hello-world package:
```
sudo apt-get update 
```
![apt update](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/apt%20update.png?raw=true)



### Finally you can install your package from your own repo:
```
sudo apt-get install hello-world
```
![apt install hello](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/apt%20install%20hello.png?raw=true)
### To check the program run hello-world by command:
```
hello
```
### it will print Hello, world!
```
Hello, world!
```
![run hello](https://github.com/shamim4s/Own-Custom-APT/blob/master/images/run%20hello.png?raw=true)

