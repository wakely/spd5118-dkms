# spd5118-dkms
Stream10 kernels do not enable the spd5118 driver for checking DDR5 RAM temps.  This package extracts the driver from 
kernel-6.12.0-228.el10 source and adds a DKMS config so you can add it.


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

## Loading on boot
~~~bash
# add module and verify
echo "spd5118" | sudo tee /etc/modules-load.d/spd5118.conf
cat /etc/modules-load.d/spd5118.conf

# test that systemd is doing its thing
sudo systemctl restart systemd-modules-load.service
sudo systemctl status systemd-modules-load.service
~~~

## Read Temps with `lm_sensors`
~~~bash
sudo dnf install lm_sensors -y
sensors
~~~

## Adding missing DIMMs
If `sensors` only shows a single DIMM, you might need to look for the others
~~~bash
# look for the SMBus entry, eg, i2c-12
sudo i2cdetect -l
# scan that bus
sudo i2cdetect -y 12
# look for an entry UU and number following it.  The UU is the currently bound stick and the number is the next one, eg, `50: UU -- 52 --`
# manually bind it
echo "spd5118 0x52" | sudo tee /sys/bus/i2c/devices/i2c-12/new_device
# check sensors again
~~~

## Persisting bindings across reboots
~~~bash
cat << 'EOF' | sudo tee /usr/local/bin/bind-spd5118.sh
#!/bin/bash
# Find the SMBus I2C bus dynamically
BUS=$(i2cdetect -l | grep -i smbus | awk '{print $1}')
if [ -n "$BUS" ]; then
    # Bind the second RAM stick at address 0x52
    echo "spd5118 0x52" > /sys/bus/i2c/devices/$BUS/new_device
fi
EOF

sudo chmod +x /usr/local/bin/bind-spd5118.sh
~~~
Create a systemd service to run it on boot:
~~~bash
cat << 'EOF' | sudo tee /etc/systemd/system/bind-spd5118.service
[Unit]
Description=Bind hidden spd5118 RAM sensors
After=systemd-modules-load.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/bind-spd5118.sh

[Install]
WantedBy=multi-user.target
EOF
# enable and test.
sudo systemctl daemon-reload
sudo systemctl enable --now bind-spd5118.service
~~~


