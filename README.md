# Create your own custom and authenticated APT repository

### Prerequisites

This tutorial assumes you are using Ubuntu, and that the following packages are installed:
```
sudo apt-get install -y gcc dpkg-dev gpg
```
# Step 0: Create a Simple Hello World Program

Before getting started with packaging,

let’s create a basic **hello world** program under **~/example/hello-world-program**.

### Create a Directory called "hello-world-program"

```
mkdir -p ~/example/hello-world-program
```

 

###  you can copy and paste the following commands into your terminal to create simple **hello.c** program

```
echo '#include <stdio.h>
int main() {
    printf("hello packaged world\n");
    return 0;
}' > ~/example/hello-world-program/hello.c
```

### Then, you can compile it with:

```
cd ~/example/hello-world-program
gcc -o hello-world hello.c
```
There’s no technical reason for picking C for this example – the language doesn’t matter. 
It’s the binary we will be distributing in our deb package.

### Run and check its working properly or not.
```
./hello-world
```

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

This directory will be the root of the package. Since we want our hello-world binary to be installed system wide, we’ll have to store it under **usr/bin/hello-world** with the following commands:

```
cd ~/example/hello-world_0.0.1-1_amd64
mkdir -p usr/bin
cp ~/example/hello-world-program/hello-world usr/bin/.
```

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

Note that we’re assuming an amd64 Architecture for this tutorial, if your binary is for a different architecture, adjust accordingly. If you’re distributing a platform-independent package, you can set the architecture to all.

### To build the .deb package, run:
```
dpkg --build ~/example/hello-world_0.0.1-1_amd64
```

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

### You can also view the contents by running
```
dpkg-deb --contents ~/example/hello-world_0.0.1-1_amd64.deb
```

which will show:
```
drwxrwxr-x shamim/shamim         0 2021-05-17 16:21 ./
drwxrwxr-x shamim/shamim         0 2021-05-17 16:18 ./usr/
drwxrwxr-x shamim/shamim         0 2021-05-17 16:18 ./usr/bin/
-rwxrwxr-x shamim/shamim     16696 2021-05-17 16:18 ./usr/bin/hello-world
```

### This package can then be installed using the **-f** option under **apt-get install**
```
sudo apt install -f ~/example/hello-world_0.0.1-1_amd64.deb
```

### To check the program run hello-world by command:
```
./hello-world
```

which should output **hello packaged world** respectively.


### Finally, if you want to remove it, you can run:
```
sudo apt-get remove hello-world
```

#### This concludes the first step of building a .deb package


# Step 2: Creating an apt Repository

In this step, we will show how to create your own apt repository which can be used to host one or more deb packages.

### Let’s start with creating a directory to hold our debs:
```
mkdir -p ~/example/apt-repo/pool/main/
```
### Then copy our deb(s) into this directory:
cp ~/example/hello-world_0.0.1-1_amd64.deb ~/example/apt-repo/pool/main/.


Once you’ve copied in all of your debs, we will create a different directory to contain a list of all available packages and corresponding metadata
```
mkdir -p ~/example/apt-repo/dists/stable/main/binary-amd64
```

If you want to support multiple architectures, make a directory above for each type (e.g. i386, amd64, etc).

### Next, we will generate a Packages file, which will contain a list of all available packages in this repository. We will use the dpkg-scanpackages program to generate it, by running:
```
cd ~/example/apt-repo
dpkg-scanpackages --arch amd64 pool/ > dists/stable/main/binary-amd64/Packages
```

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

### Let’s take a quick look at the contents of the Packages file:

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


### Next let’s run the generate-release.sh script with:
```
cd ~/example/apt-repo/dists/stable
~/example/generate-release.sh > Release
```

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
When server is running you need to configure this apt repository by :
```
echo "deb [trusted=yes] http://136.243.196.122:8000/apt-repo stable main" >> /etc/apt/sources.list.d/example.list
```

Then finally update apt and install our new hello-world package:
```
sudo apt-get update 
```
### Finally you can install your package from your own repo:
```
sudo apt-get install hello-world
```
### To check the program run hello-world by command:
```
./hello-world
```


# Step 3: Signing your apt Repository With GPG/PGP

### Creating a New Public/Private PGP Key Pair

```
echo "%echo Generating an example PGP key
Key-Type: RSA
Key-Length: 4096
Name-Real: Shamim
Name-Email: me@shamim.app
Expire-Date: 0
%no-ask-passphrase
%no-protection
%commit" > /tmp/example-pgp-key.batch
```

then we will generate it under a new temporary gpg keyring:
```
export GNUPGHOME="$(mktemp -d ~/example/pgpkeys-XXXXXX)"
gpg --no-tty --batch --gen-key /tmp/example-pgp-key.batch
```
Since we overrode the GNUPGHOME to a temporary directory, we can keep this key separate from our other keys. Let’s take a quick look at the contents of the directory:
```
ls "$GNUPGHOME/private-keys-v1.d"
```

will show something along the lines of
```
A3FB218BA1929542FF110C7D1B077B6469F769C9.key
```

which contains binary data. We can also view all of our loaded keys with:
```
gpg --list-keys
```
which will show something similar to
```
/home/alex/example/pgpkeys-pyml86/pubring.kbx
-------------------------------
pub   rsa4096 2021-05-18 [SCEA]
      B4D5C8B003C50A38A7E85B5989376CAC59892E72
uid           [ultimate]
```
This PGP key is comprised of both a public key, and a private key. Let’s start with exporting the public key:
```
gpg --armor --export example > ~/example/pgp-key.public
```

Next let’s export the private key so we can back it up somewhere safe.
```
gpg --armor --export-secret-keys example > ~/example/pgp-key.private
```
Now that we’ve generated a PGP key pair, let’s move on to signing files with them.

### Signing the Release File
Before we start signing with out keys, let’s make sure that we can import the backup we made. To do that, we will create a new GPG keyring location:
```
export GNUPGHOME="$(mktemp -d ~/example/pgpkeys-XXXXXX)"
```

Next we will import our backed up private key:
```
cat ~/example/pgp-key.private | gpg --import
```
which should show a similar imported message:
```
gpg: key 4E793BC948F34C6F: public key "example <example@example.com>" imported
gpg: key 4E793BC948F34C6F: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

Ok, let’s get around to signing the Release file now.
```
cat ~/example/apt-repo/dists/stable/Release | gpg --default-key example -abs > ~/example/apt-repo/dists/stable/Release.gpg
```

Now when an apt client performs an update, it will fetch both Release and Release.gpg and will verify the signature is valid; However to increase the speed, we will create a third file InRelease which will combine both the contents of Release and the PGP signature:
```
cat ~/example/apt-repo/dists/stable/Release | gpg --default-key example -abs --clearsign > ~/example/apt-repo/dists/stable/InRelease
```

### Testing It Out
We need to tell apt which public pgp key to use when verifying the apt repository. We will add a new signed-by attribute to our apt config:
```
echo "deb [arch=amd64 signed-by=$HOME/example/pgp-key.public] http://136.243.196.122:8000/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/example.list
```

Next start back up your web server Again:

```
cd ~/example
```
### Run python server on a specific port on this directory as a web directory
```
python3 -m http.server -b your-server-ip  8000
```

Then finally update apt and install our new hello-world package:
```
sudo apt-get clean
sudo apt-get update 
```
### Finally you can install your package with signed-key:
```
sudo apt-get install hello-world
```
### To check the program run hello-world by command:
```
./hello-world
```
