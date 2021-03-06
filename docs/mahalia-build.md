[PDK](https://github.com/64studio/pdk) is the Platform Development Kit that we use for creating the [Mahalia](https://github.com/mahalia-bone/) distro images, based on Debian GNU/Linux.

Most Mahalia users will not need to create their own distro images. To download and flash ready-made Mahalia images, please follow the instructions in the [mahalia-distro](https://github.com/mahalia-bone/mahalia-utils/blob/master/docs/mahalia-distro.md) documentation.

If you would like to create a new Mahalia image with your own modifications, read on! The following instructions are based on using a Debian GNU/Linux 9 'stretch' build server with _sudo_ installed.

## Install PDK

```bash
sudo apt install apt-transport-https rng-tools make
echo "deb https://apt.64studio.net stretch main" | sudo tee /etc/apt/sources.list.d/64studio.list
wget -qO - https://apt.64studio.net/archive-keyring.asc | sudo apt-key add -
sudo apt update
sudo apt install pdk pdk-mediagen
```
- Ignore the `rng-tools` service if it fails to start with "Cannot find a hardware RNG device to use." You won't need this to run as a service for now.

## APT Repository key

- Start up the random number daemon which we'll use in a minute for key generation, then create a user called _apt_ with sudo rights to use for building images and serving packages.

```bash
sudo rngd -r /dev/urandom
sudo adduser apt sudo
```
- Now log in to the build machine as the _apt_ user with the password you just set, and ensure you are in the apt user's home directory.

```
cd ~
```
- When using gpg to generate an APT repository key, make the email address apt@your-domain where _your-domain_ is the domain name used by your project. This will not necessarily be the existing project domain name if you are serving packages and images from somewhere else.
- You will be prompted to create a signing password, twice. To enable automated builds later, you can automate signing with [gpg-preset-passphrase](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dpreset_002dpassphrase.html) running on each boot.

```bash
gpg --gen-key
```

## Download PDK project 'Mahalia'

- Download the archive containing the PDK project to your build server. We'll assume this file is called _mahalia-distro_master.tar.gz_ and it will be downloaded into the home directory of the 'apt' user that we just created.

```bash
wget http://your-domain/mahalia-distro_master.tar.gz
mkdir -p ~/pdk/projects
tar zxvf ~/mahalia-distro_master.tar.gz -C ~/pdk/projects
```

## Initialise project

- Change to the project directory and reinitialise the local Git repository for the project.

```bash
cd ~/pdk/projects/mahalia
git init
```
- Optionally, set the MEDIAGEN_PATH to the location of PDK's mediagen script on your system, if using a modified version or a non-standard path. If you have installed pdk-mediagen from the 64 Studio apt repository mentioned above, this path will be _/usr/share/pdk-mediagen/mediagen_ and will be included by default, so you can skip this step.

```
export MEDIAGEN_PATH=/usr/share/pdk-mediagen/mediagen
```

## Modify project for your needs

- Copy any updated Mahalia applications or other local packages into the _debs_ directory of the PDK workspace.
- In the Makefile, you should change the \<mediagen.root-password\> for the generated image to something more secure.
- While editing the Makefile, you should also change the \<apt-deb.key\> to the email address of the repository key apt@your-domain that you created earlier.

```bash
nano Makefile
make
```
- Running _make_ will run a PDK channel update, resolve dependencies for the local packages which are custom for Mahalia, and write out the mahalia.xml project file.


## Build image

```bash
make image
```
- This step uses `sudo`, so you may be prompted for the 'apt' user's password as well as your GnuPG passphrase.

- All the standard Debian packages required will be downloaded, and the distro image created. This can take ten minutes to half an hour, depending on the speed of the build server's CPU. 

- After this, you should find the _out.img_ file and a compressed version _out.img.gz_ in your PDK workspace.

- PDK will create a checksum file _out.img.md5sum_ for users to compare against when the image is downloaded. 

- You may find it useful to rename this image to give it a distinct version number. In this case you might prefer to recreate the checksum file and compressed image to reflect the new filename.


## Flash image

- Please refer to the [Mahalia flashing instructions](https://github.com/mahalia-bone/mahalia-utils/blob/master/docs/mahalia-distro.md) to get the image onto the BeagleBone.


## Further reading

See the [PDK project on GitHub](https://github.com/64studio/pdk) for more details of how to use PDK.
