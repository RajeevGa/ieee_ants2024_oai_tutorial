# Required Software Installation

# 1. Git
```shell
sudo apt-get update
sudo apt-get install git
```


# 2. Wireshark

:exclamation: Make sure to select `yes` when the Wireshark installation asks you whether non-superusers should be able to capture packets.
Otherwise, you will have to run in sudo mode.

```bash
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install -y git net-tools wireshark
```

# 3. Docker

```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
sudo apt update
sudo apt install -y docker docker-ce
```

:exclamation: Add your username to the docker group, otherwise you will have to run in sudo mode. 

```bash
sudo usermod -a -G docker $(whoami)
```

:exclamation: Now, close the terminal tab and re-open it.

# 3. Networking
```bash
sudo apt install -y iperf3
```
# 4. Tutorial Repository
```bash
git clone https://github.com/RajeevGa/ieee_ants2024_oai_tutorial.git
```
