---
layout: post
---

To let FreeBSD virtual machine & jail manager CBSD work smoothly, you need to prepare something before install it.

1. Load some modules: `sysrc kld_list+="vmm if_tuntap if_bridge nmdm" && service kld restart`
2. Create a zfs dataset: `zfs create zroot/cbsd && zfs set mountpoint=/usr/cbsd zroot/cbsd `
3. Initialize it: `env workdir="/usr/cbsd" /usr/local/cbsd/sudoexec/initenv`

For the options during the initialization, you can consult the [official page](https://www.bsdstore.ru/en/installing_cbsd.html#initenv). Hereby are some tips:

* Please fill nodename - Node means one spot or one server. Example: rob.domain.home
* Please fill nodeip - Static IP of this server.
* Please fill nodeippool - Create a private Lan exclusively for these nodes like 10.0.0.0/24 or belong to existing Lan like 192.168.1.0/24.
* Please fill natip - The NAT like 192.168.1.1.

You can reconfigure it by `cbsd initenv-tui`.

After installation finished, it's time to create a Virtual Machine:

`cbsd bconstruct-tui`

Remember to choose the vm profile you want and set the VNC settings through **bhyve_vnc_options**, once inside you will find some settings:

* bhyve_vnc_tcp_bind: [127.0.0.1] => changed to [your node ip]
* vm_vnc_port: [0] => [5900]
* vnc_password:

Still you can reconfigure it by `cbsd bconfig jname=fnos`.

Finally you can list and start it with command:

`cbsd bls && cbsd bstart [vm-name]`

And connect it with VNC client, in MacOS just type *vcn://192.168.31.240:5900* into the Safari browser.
