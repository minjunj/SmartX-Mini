# Lab#1. Box Lab

## 0. Objective

![Final Goal](./img/final_goal.png)

Box Lab에서는 \*베어 메탈에 os를 직접 설치해보고  
이 안에 가상 머신과 컨테이너를 띄운 뒤 가상 스위치로 서로를 연결시켜보는 것입니다.<br>
In the Box Lab, we will install OS directly on a *bare metal and build a virtual machine and a container inside the bare metal. Finally, we will connect two of them via a virtual switch.

\*베어 메탈: 하드웨어 상에 어떤 소프트웨어도 설치되어 있지 않은 상태<br>
\*bare metal: a hardware without any installed software

![Objective](./img/objective.png)

세부적인 구조를 보면 다음과 같습니다.
<br>let's take a close look at the overall structure.

## 1. Theory

![VM Container](./img/vm_container.png)

- KVM Hypervisor => Virtual Machine

  하나의 피지컬 머신을 여러개의 가상 머신으로 나눌 것입니다. 각각의 가상 머신은 모두 독립적이며 개별적인 자원을 할당받습니다. 또한, 피지컬 머신의 OS와 다른 OS를 사용자 마음대로 정할 수 있습니다. 가상 머신은 피지컬 머신과 비교할 때 차이가 거의 없지만, 그만큼 Container보다 무겁고 생성하는데 오래걸립니다.<br>
  We will divide a physical machine into several virtual machines. Each of the virtual machines is independent and allocated individual computation resources. Also, Each virtual machine can
  have a different OS from the host OS. There is almost no difference between **a virtual machine** and a physical machine. However, It takes more time to build a virtual machine than a container and requires more computation resources to operate a virtual machine.  

  저희는 가상 머신을 생성하기 위해 리눅스에 기본적으로 탑재되어있는 KVM Hypervisor를 사용할 것입니다.<br>
  To build a virtual machine, we will use the KVM hypervisor.

- Docker Runtime => Container

  가상 머신과 비교했을 때 Container의 가장 큰 특징은 OS층이 없다는 것입니다. Container는 가상 머신과 달리 피지컬 머신의 OS를 공유합니다. 그리고 가상 머신은 각각의 머신이 독립적이지만 Container는 그렇지 않습니다.<br>
  One of the most important properties is that a container does not have OS layer compared to a virtual machine. A container shares OS with its physical machine's OS. And each of the virtual machines is independent. Meanwhile, each of the containers does not.  

  Container를 생성하기 위해서 가장 널리 쓰이는 Docker Runtime을 사용할 것입니다.
  To build a containerm we will uses Docker Runtime.

![Virutal Switch](./img/switch.png)

- Open vSwitch => Virtual Switch

  가상 스위치는 OS안에서 실제 물리적인 스위치처럼 동작합니다. 이번 실습에서 Open vSwitch를 통해 가상 스위치를 구성할 것이고 이를 통해, 가상머신과 컨테이너를 연결할 것입니다.<br>
  A virtual switch operates just like a real physical switch in OS. In this experiment, we will make up a virtual switch using Open v Switch. and we will connect a container and a virtual machine.

  Open vSwitch 역시 linux에 기본적으로 포함돼있는 가상 스위치입니다.<br>
  Open vSwitch is an open-source virtual switch software designed for virtual servers.

  A software-based virtual switch allows one VM to communicate with neighbor VMs as well as to connect to Internet (via physical switch).
  Software-based switches (running with the power of CPUs) are known to be more flexible/upgradable and benefited of virtualization (memory overcommit, page sharing, …)
  VMs (similarly containers) have logical (virtual) NIC with virtual Ethernet ports so that they can be plugged into the virtual interface (port) of virtual switches.

## 2. Practice

> When mouse is hover on the code block, copy button is appeared right side of block. You can easily copy whole code using copy button.
> ![copy button](img/copy.png)

> please check allocated IP address of your NUC, VM, and container in the ribbon paper.
> <br> ex) yourname | student ID | NUC's IP | VM's IP | container's IP

### 2-1. NUC: OS Installation

OS : Ubuntu Desktop 20.04 LTS(64bit)
Download Site : <https://releases.ubuntu.com/20.04/>
Installed on NUC(i.e., bare metal)

#### 2-1-1. Updates and other software

- Select ‘Minimal installation’

#### 2-1-2. Installation type

- Select 'Erase disk and install ubuntu' <br>

- If an issue which is related booting occured, do the following thing.
- Select ‘Something else’
- On /dev/sda or /dev/nvme0n1

  - (UEFI), add 512MB EFI partition
  - Add empty partition with 20GB (20480MB) (Select ‘do not use the partition’)
  - Add Etc4 partition on leave memory

- Select Boot loader

  - BIOS: Ext4 partition
  - UEFI: EFI partition

- LVM 관련 오류 발생 시<br> If a issue which is related to LVM occured, do the following thing.

  1. 뒤로 이동하여, 첫 Installation type 화면으로 이동
  <br> go back to first installation type display.
  2. select Erase disk

     - choose none in advance.

  3. 시간대 선택 화면까지 진행
  <br> do the steps up to 'where are you?'

  4. 여기서 뒤로 돌아가, 다시 첫 Installation type 화면으로 이동
  <br> go back from here to first installation type display.

  5. Something else 선택하여 정상 진행
  <br> choose Something else and do the following steps

### 2-2. NUC: Network Configuration

- ‘Temporary’ Network Configuration using GUI

  ![Network Configuration](./img/network_configuration.png)

- Click the LAN configuration icon.
  <img src="./img/network_setting1.png" />

- Enter the network info.
  (IP address, subnet mask, gateway)
  <img src="./img/network_setting2.png" />

- **Set Prerequisites**


1. Update & Upgrade

   ```bash
   sudo apt update
   sudo apt upgrade
   ```
2. upgrade vim text editor

    ```bash
    sudo apt install vim
    ```

3. Install net-tools & ifupdown

   ```bash
   sudo apt install -y net-tools ifupdown
   ifconfig -a
   ```

   ![Network Configuration](./img/ifconfig.png)

  <br> **caution!** After you enter  ```ifconfig -a```.<br>
  If there exist `enp88s0` and `enp89s0`, **please reboot your NUC.**
  <br>**주의!** 만일 터미널에 ```ifconfig -a``` 작성 후 `enp88s0` 와 `enp89s0`가 존재한다면 **꼭 재부팅** 해주세요.
  ![two NIC](./img/two_NIC.png)
    
4. Install openvswitch-switch & make br0 bridge

   ```bash
   sudo apt -y install openvswitch-switch
   sudo ovs-vsctl add-br br0
   sudo ovs-vsctl show
   ```

   ![Ovs Vsctl Show](./img/ovs_vsctl_show.png)

- Disable netplan

  ```bash
  sudo su # Enter superuser mod
  systemctl stop systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
  systemctl disable systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
  systemctl mask systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
  apt-get --assume-yes purge nplan netplan.io
  exit # Exit superuser mod
  ```

- eno1 interface

  ```bash
  sudo vi /etc/systemd/resolved.conf
  ```

  DNS 왼편에 있는 주석표시 /# 을 제거해주고<br>
  delete # next to DNS.   
  DNS 주소를 명시해주세요<br>
  type the DNS addresses as below.

  > …
  >
  > DNS=203.237.32.100 203.237.32.101
  >
  > …

- Network interface

  Open /etc/network/interfaces

  ```bash
  sudo vi /etc/network/interfaces
  ```

  Configure the network interface `vport_vFunction` is a tap interface and attach it to your VM.

  !!!들여쓰기는 Tab 한번입니다!!!
  <br>**caution! one tab for indentation**

  `<your nuc ip>`에 현재 nuc의 ip와 `<gateway ip>`에 gateway ip를 입력해주세요.
  <br>type your nuc's ip in `<your nuc ip>`and gateway ip in `<gateway ip>`(In this experiment, **172.29.0.254** is the gateway IP.)
  

  주의!
  NUC에 이더넷 포트가 두 개 있는 경우 `eno1`이라는 인터페이스가 없습니다. `ifconfig` 명령으로 네트워크에 연결된 인터페이스(`enp88s0` 또는 `enp89s0`)를 확인합니다. (예를 들어, 터미널에 `ifconfig -a` 명령어를 입력하고 RX 및 TX 패킷이 0이 아닌 인터페이스를 선택합니다.) 그리고 아래 텍스트의 `eno1`을 모두 `enp88s0` 또는 `enp89s0`으로 변경합니다.
  <br>caution!
  <br>If your NUC has two ethernet ports, there is no interface named `eno1`. Check which interface(`enp88s0` or `enp89s0`) is connected to the network by `ifconfig` command.(e.g., Type the `ifconfig -a` command into the terminal and choose an interface with non-zero RX and TX packets.) And change all `eno1` in the below text to `enp88s0` or `enp89s0`.


  ```text
  auto lo
  iface lo inet loopback

  auto br0
  iface br0 inet static
      address <your nuc ip>
      netmask 255.255.255.0
      gateway <gateway ip>
      dns-nameservers 203.237.32.100

  auto eno1
  iface eno1 inet manual

  auto vport_vFunction
  iface vport_vFunction inet manual
      pre-up ip tuntap add vport_vFunction mode tap
      up ip link set dev vport_vFunction up
      post-down ip link del dev vport_vFunction
  ```

Save and quit the editor.
파일을 저장하고 나와주세요. 

**주의!** 만약 NUC 2개의 lan port가 있다면, `eno1` interface가 없습니다. 그러므로 하단의 block에서 `eno1`을 위에서 선택한 interface 중 하나로 변경해주세요(즉, `enp88s0` 또는 `enp89s0` 중에서 적절한 것을 선택해주세요.)
**caution!** Similarly, if your NUC has two ethernet ports, there is no interface named `eno1`. <br>Therefore, replace `eno1` at the bottom with the appropriate interface chosen above, either `enp88s0` or `enp89s0`.

  ```bash
  sudo systemctl restart systemd-resolved.service
  sudo ifup eno1  #change this if you are using two-port NUC
  ```


Restrart the whole interfaces 1<br>
전체 interface를 다시 시작해주세요.

```bash
sudo su # Enter superuser mod
systemctl unmask networking
systemctl enable networking
systemctl restart networking
exit # Exit superuser mod
```

vport_vFunction을 연결한 VM을 만들겠습니다. 이 탭(vport_vFunction)은 VM의 NIC(네트워크 인터페이스 카드)라고 생각하시면 됩니다.
<br>We will make VM attaching vport_vFunction. You can think this tap as a NIC(Network Interface Card) of VM.

'br0'에 포트 'eno1' 및 'vport_vFunction'을 추가합니다.<br>
**주의!** 만약 NUC 2개의 lan port가 있다면, `eno1` interface가 없습니다. 그러므로 하단의 block에서 `eno1`을 위에서 선택한 interface 중 하나로 변경해주세요(즉, `enp88s0` 또는 `enp89s0` 중에서 적절한 것을 선택해주세요.)

add port ‘eno1’ and ‘vport_vFunction’ to ‘br0’<br>
**caution!** Similarly, if your NUC has two ethernet ports, there is no interface named `eno1`. <br>Therefore, replace `eno1` at the bottom with the appropriate interface chosen above, either `enp88s0` or `enp89s0`.

```bash
sudo ovs-vsctl add-port br0 eno1   #change this if you are using two-port NUC
sudo ovs-vsctl add-port br0 vport_vFunction
sudo ovs-vsctl show
```

Below is the figure you configurated so far

![Vport VFunction](./img/vport_vFunction.png)

Restrart the whole interfaces 2<br>
전체 interface를 다시 시작해주세요.

```bash
sudo su # Enter superuser mod
systemctl unmask networking
systemctl enable networking
systemctl restart networking
exit # Exit superuser mod
```

### 2-3. NUC: Making VM with KVM

- Install dependency to upgrade KVM

  Install dependency & download Ubuntu 20.04.6 64bit server image.

  ```bash
  sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
  # upgrade KVM
  # qemu is open-source emulator

  wget https://ftp.lanet.kr/ubuntu-releases/20.04.6/ubuntu-20.04.6-live-server-amd64.iso
  ```
  

  Now we are ready to make VM. So, continue the setting.

- Prepare for Ubuntu VM

  To Make a VM image, enter this command

  ```bash
  sudo qemu-img create vFunction20.img -f qcow2 10G
  ```

  Boot VM image from Ubuntu iso file
  <br>**띄어쓰기 주의하기**
  <br>Be cautious about spacing.

  ```bash
  sudo kvm -m 1024 -name tt -smp cpus=2,maxcpus=2 -device virtio-net-pci,netdev=net0 -netdev tap,id=net0,ifname=vport_vFunction,script=no -boot d vFunction20.img -cdrom ubuntu-20.04.6-live-server-amd64.iso -vnc :5 -daemonize -monitor telnet:127.0.0.1:3010,server,nowait,ipv4 -cpu host
  ```

  Configure SNAT with iptables for VM network<br>

  **NUC's ip address**을 하단의 `<Your ip address>`에 기입해주세요(다만, **괄호는 지우고** 172.29.0.X의 형식으로 작성해주세요)

  **주의!** 만약 NUC 2개의 lan port가 있다면, `eno1` interface가 없습니다. 그러므로 하단의 block에서 `eno1`을 위에서 선택한 interface 중 하나로 변경해주세요(즉, `enp88s0` 또는 `enp89s0` 적절한 것을 선택해주세요.)

  please type your **NUC's ip address** in `<Your ip address>`
  <br>**caution!** Similarly, if your NUC has two ethernet ports, there is no interface named `eno1`. <br>Therefore, replace `eno1` at the bottom with the appropriate interface chosen above, either `enp88s0` or `enp89s0`.
  

  ```bash
  sudo iptables -A FORWARD -i eno1 -j ACCEPT
  sudo iptables -A FORWARD -o eno1 -j ACCEPT
  sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eno1 -j SNAT --to <Your ip address>
  ```

  ```bash
  sudo vi /etc/sysctl.conf
  ```

  remove annotation sign ( '#' )

  > #net.ipv4.ip_forward=1  
  > →  
  > net.ipv4.ip_forward=1

  ```bash
  sudo sysctl -p
  ```

- Install Ubuntu VM (control with ‘Enter key’ and ‘Arrow keys’)

  Install VNC viewer and see inside of VM

  ```bash
  sudo apt-get install tigervnc-viewer
  ```

  turn on this vm

  ```bash
  vncviewer localhost:5
  ```

  ![Install Ubuntu](./img/install_ubuntu.png)

- VM network configuration (control with ‘Enter key’ and ‘Arrow keys’)

  ![Ubuntu Network](./img/ubuntu_network.png)

  > select network device → Edit IPv4  
  > IPv4 Method → Manual
  >
  > subnet: 172.29.0.0/24  
  > Address: < your VM IP >  
  > Gateway: 172.29.0.254  
  > Name Servers: 203.237.32.100

  search domains는 공백으로 남겨주세요!
  <br> please leave search domains as empty.
  <br> 그리고 위의 `< your VM IP >` 작성 시에 **괄호는 지우고** 172.29.0.X의 형식으로 작성해주세요

- Installation Completed (control with ‘Enter key’ and ‘Arrow keys’)

  When ‘installation completed’ message is shown, terminate the VM

  ```bash
  sudo killall -9 qemu-system-x86_64
  ```

  boot VM again (mac should be different from others).

  ```bash
  sudo kvm -m 1024 -name tt -smp cpus=2,maxcpus=2 -device virtio-net-pci,netdev=net0 -netdev tap,id=net0,ifname=vport_vFunction,script=no -boot d vFunction20.img
  ```

### 2-4. OVS connects with KVM

- Check situation

  ```bash
  ovs-vsctl show
  ```

  ![Ovs Vsctl](./img/ovs-vsctl.png)

### 2-5. NUC: Installing ssh in VM

- Don’t forget to install ssh in VM

  ```bash
  sudo apt update
  sudo apt -y install ssh
  ```

  ssh로도 vm에 원격 접속할 수 있지만, 이 lab에서는 다루지 않겠습니다.
  <br> you can also access your VM using ssh, but we will not cover this in this experiment.

### 2-6. Install docker

Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers. The service has both free and premium tiers. The software that hosts the containers is called Docker Engine. It was first started in 2013 and is developed by Docker, Inc.

Set up the repository

Install packages to allow apt to use a repository over HTTPS

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```

Add Docker's official GPG key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Add the Docker apt repository

```bash
# For All NUCs
 echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update APT repos.

```bash
# For All NUCs
sudo apt-get update
```

Install Docker

```bash
sudo apt-get install -y --allow-downgrades \
          containerd.io=1.2.13-2 \
          docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
          docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
```

Create /etc/docker

```bash
sudo mkdir -p /etc/docker
```

Set up the Docker daemon

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

Create /etc/systemd/system/docker.service.d

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl start docker.socket
```

### 2-7. Check docker installation

```bash
sudo docker run hello-world
```

If it doesn’t work, please try several times. Nevertheless, if you are not successful, try running from the installing `docker-ce`, `docker-ce-cli`, `containerd.io`

![1](./img/1.png)

### 2-8. Make Container

Make a container named 'c1'

```bash
sudo docker run -it --net=none --name c1 ubuntu:20.04 /bin/bash
```

Press ctrl + p, q to detach docker container.

※ docker attach [container_name]: get into docker container console

### 2-9. Connect docker container

Install OVS-docker utility in host machine (Not inside of Docker container)
<br> **도커 외부에서**, 즉 host machine에서 OVS-docker utility를 하단의 명령어로 설치합니다

```bash
sudo docker start c1
sudo ovs-docker del-port br0 veno1 c1
sudo ovs-docker add-port br0 veno1 c1 --ipaddress=[docker_container_IP]/24 --gateway=[gateway_IP]
# please type gateway IP and docker container IP.
```
위의 --ipaddress=[docker_container_IP]/24 --gateway=[gateway_IP] 작성 시에 [ ]은 빼고, 172.29.0.X의 형식으로 작성해주세요.
<br> 예를 들어, --ipaddress=172.29.0.X/24 --gateway=172.29.0.254

Enter to docker container

```bash
sudo docker attach c1
```

In container,

```bash
apt update
apt install -y net-tools
apt install -y iputils-ping
```

### 2-10. Check connectivity: VM & Container

Check connectivity with ping command from docker to VM

```bash
ping <VM IP address>
# please type this command in the container.
```
예를 들어, ping 172.29.0.X 

> Do above command in both container and KVM VM
> <br> **Finally, you can check that the container and the VM are connected.**

### 2-Appendix. Keep Docker network configuration

Whenever NUC is rebooted, network configuration of Docker container is initialized by executing commands in `rc.local` file.
