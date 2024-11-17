# TUT2: Innovation and Prototyping in O-RAN using Open-Source Testbeds

Welcome to our IEEE ANTS 2024 tutorial! On this page, you will find all the information needed to participate in the OpenAirInterface (OAI) hands-on session.
You should carefully follow the instructions below and set up the environment prior to the hands-on session.

## 1. Minimum Computing Requirements
To run an end-to-end 5G network with OAI Core Network (CN),
Radio Access Network (RAN) including base station (gNB) and user equipment (nrUE) in RFsim mode, we need the following minimum recommended computing requirements: 
-  Operating System: Ubuntu 22.04 LTS
-  CPU: 8 virtual cores x86_64 @ 3.5 GHz
-  RAM: 16 GB
  
You can use your own laptop if it fulfils the above requirments, or you can use a Virtual Machine (VM) in a cloud platform.
For VM setup, we recomend using the Google Computing Platform (GCP).
In either case, all users must bring a laptop that has access to internet.

## 1. Environment Setup

Please follow these instrunctions in the order they appear:

1. To setup a VM in the GCP, follow the instructions at [vmsetup/Setup_VM_GCP.md](./vmsetup/Setup_VM_GCP.md)
2. Install the required software using [utils/Util_Installation.md](./utils/Util_Installation.md) in your VM.
3. Install the OAI CN by following the instructions at [cn/CN_Installation.md](./cn/CN_Installation.md)
4. Finally, install the RAN using the instructions at [ran/RAN_Installation.md](./ran/RAN_Installation.md)
