# spd5118-dkms
spd5118 driver from kernel-6.12.0-228.el10 with DKMS config


## Install
~~~bash
git clone git@github.com:wakely/spd5118-dkms.git
sudo cp -R ./spd5118-dkms /usr/src/spd5118-1.0
sudo dkms add -m spd5118 -v 1.0
sudo dkms build -m spd5118 -v 1.0
sudo dkms install -m spd5118 -v 1.0
sudo modprobe spd5118
lsmod | grep spd5118
~~~
