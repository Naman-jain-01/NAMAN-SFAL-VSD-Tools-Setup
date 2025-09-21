
````markdown
# SFAL-VSD-Tools-Setup

This repository documents the setup and installation of tools required for VLSI design as part of the **SFAL-VSD (Skywater Foundry ASIC Lab - VLSI System Design)** workshop. It includes a summary of the introductory video and step-by-step instructions for installing necessary tools on an Ubuntu 20.04+ system with the specified machine configuration.

---

## 📌 Day 0: Tools Installation

### 🎥 Video Summary

The introductory video (Day 0) provides an overview of the VLSI design flow and the importance of open-source tools in modern chip design. It covers:

- Introduction to the SFAL-VSD workshop objectives.  
- Overview of tools like Yosys, Icarus Verilog, GTKWave, ngspice, Magic, and OpenLane for synthesis, simulation, and physical design.  
- System requirements for setting up a virtual machine with adequate resources:
  - 6GB RAM  
  - 50GB HDD  
  - 4 vCPUs  
- Tool roles in VLSI design:
  - Yosys → RTL to netlist synthesis  
  - Icarus Verilog (iverilog) → Verilog simulation  
  - GTKWave → Waveform viewing  
  - ngspice → Analog/mixed-signal simulation  
  - Magic → Layout & physical design  
  - OpenLane → Automated RTL-to-GDSII flow  

The video emphasizes the need for a proper environment setup to ensure smooth execution of the workshop tasks.

---

## 💻 System Requirements

- Virtual Machine: [Oracle VirtualBox](https://www.virtualbox.org/wiki/)  
- OS: Ubuntu 20.04+  
- Hardware:
  - 6GB RAM  
  - 50GB HDD  
  - 4 vCPUs  

---

## 🔧 Tool Installation Instructions

### 1️⃣ Yosys (Synthesis Tool)

```bash
sudo apt-get update
git clone https://github.com/YosysHQ/yosys.git
cd yosys
sudo apt install make
sudo apt-get install -y build-essential clang bison flex \
libreadline-dev gawk tcl-dev libffi-dev git \
graphviz xdot pkg-config python3 libboost-system-dev \
libboost-python-dev libboost-filesystem-dev zlib1g-dev
make config-gcc
make
sudo make install
````

✅ Verify:

```bash
yosys --version
```

---

### 2️⃣ Icarus Verilog (Simulation Tool)

```bash
sudo apt-get update
sudo apt-get install -y iverilog
```

✅ Verify:

```bash
iverilog -V
```

---

### 3️⃣ GTKWave (Waveform Viewer)

```bash
sudo apt-get update
sudo apt install -y gtkwave
```

✅ Verify:

```bash
gtkwave --version
```

---

### 4️⃣ ngspice (Circuit Simulator)

```bash
wget https://sourceforge.net/projects/ngspice/files/ngspice-37.tar.gz
tar -zxvf ngspice-37.tar.gz
cd ngspice-37
mkdir release && cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install
```

✅ Verify:

```bash
ngspice --version
```

---

### 5️⃣ Magic (Layout Tool)

```bash
sudo apt-get install -y m4 tcsh csh libx11-dev tcl-dev tk-dev \
libcairo2-dev mesa-common-dev libglu1-mesa-dev libncurses-dev
git clone https://github.com/RTimothyEdwards/magic
cd magic
./configure
make
sudo make install
```

✅ Verify:

```bash
magic --version
```

---

### 6️⃣ OpenLane (RTL-to-GDSII Flow)

#### Install Dependencies

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

#### Install Docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```

Add user to Docker group:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot
```

#### Install OpenLane & PDKs

```bash
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane
cd OpenLane
make
make test
```

✅ Verify:

```bash
docker --version
./flow.tcl -design test
```

---

## 📷 Tool Snapshots

Verify each tool’s installation by running version checks. Expected outputs:

* *Yosys*: `yosys --version`
* *Icarus Verilog*: `iverilog -V`
* *GTKWave*: `gtkwave --version`
* *ngspice*: `ngspice --version`
* *Magic*: `magic --version`
* *OpenLane*: `./flow.tcl -design test`

Snapshots are stored in the **snapshots/** directory:

* Yosys Version
* Iverilog Version
* GTKWave Version
* ngspice Version
* Magic Version
* OpenLane Test

---

## 📝 Notes

* Ensure **system reboot** after Docker installation.
* Tools must be installed on **Ubuntu 20.04+ with specified configuration**.
* **OpenSTA is excluded** (not required for SFAL participants).

---

```

👉 You can directly copy-paste this into your **README.md** file, and it will render with clean formatting, code blocks, and section separation.  

Would you like me to also **add badges (build, version, license)** at the top of the README for a more professional GitHub look?
```
# SFAL-VSD-Tools-Setup  

![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2020.04+-orange?logo=ubuntu)  
![License](https://img.shields.io/badge/License-MIT-green.svg)  
![Tools](https://img.shields.io/badge/VLSI%20Tools-Yosys%20%7C%20Iverilog%20%7C%20GTKWave%20%7C%20ngspice%20%7C%20Magic%20%7C%20OpenLane-blue)  
![Status](https://img.shields.io/badge/Setup-Completed-success)  

This repository documents the setup and installation of tools required for VLSI design as part of the **SFAL-VSD (Skywater Foundry ASIC Lab - VLSI System Design)** workshop. It includes a summary of the introductory video and step-by-step instructions for installing necessary tools on an Ubuntu 20.04+ system with the specified machine configuration.  

---
