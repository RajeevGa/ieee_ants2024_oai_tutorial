% OAI Hands-On Session
% Rajeev Gangula, Chandra R. Murthy, Rakesh Mundlamuri, Vinay Kulkarni, Venkatareddy Akumalla
% December 15, 2025

**What do we cover in the OAI Hands-On Session?**

The aim of this tutorial is to:

1. Set-up of end-to-end monolithic 5G/NR SA setup with RFsimulator from source
2. Add a custom UE
3. Configuring TDD pattern
4. Configuring bandwidth
5. CU-DU split
6. Working with multiple UE's

# Install required software utilities

## 1. Git
```
sudo apt-get update
sudo apt-get install git
```
## 2. Wireshark

Make sure to select `yes` when the Wireshark installation asks you whether non-superusers should be able to capture packets.
Otherwise, you will have to run in sudo mode.

```
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install -y git net-tools wireshark
```

## 3. Networking
```
sudo apt install -y iperf3
```

# Install required software utilities
## 4. Docker

```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
sudo apt update
sudo apt install -y docker docker-ce
```

Add your username to the docker group, otherwise you will have to run in sudo mode.

```
sudo usermod -a -G docker $(whoami)
```
Now, close the terminal tab and re-open it.


# OAI 5G core: Procedure to pull and run docker images

clone the repository

```bash
git clone https://github.com/RajeevGa/ieee_ants2024_oai_tutorial.git
cd ieee_ants2024_oai_tutorial/cn
```

If you dont have a docker account, create a docker account in [docker signup](https://www.docker.com/)

Login to docker

```bash
docker login -u <username>
```

Enter your password


# OAI 5G core: Procedure to pull and run docker images
Now, pull the 5G core docker images
```bash
docker compose pull
```

start core network
```bash
docker compose -f docker-compose.yaml up -d
```

watch status of the core network
```bash
watch docker compose -f docker-compose.yaml ps -a
```
All the docker containers should be `healthy`

# Procedure to install and run OAI basestation (gNB) and the user equipment (nrUE)

Open a new ssh terminal and clone the ran repository
```bash
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
```
compile the gNB and nrUE

```bash
cd openairinterface5g/
git checkout 2024.w46
source oaienv
cd cmake_targets/
./build_oai -I  
./build_oai -w SIMU --gNB --nrUE --ninja
```


---

# Key parameters to check in the configuration files: mcc, mnc, sst, sd

5G Core:
```
~/ieee_ants2024_oai_tutorial/cn/conf/config.yaml
```

gNB:
```
~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.106prb.rfsim.conf
```

nrUE:
```
~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```

</p>
<p align="center">
<img style="display: block; margin: auto;" src="./vmsetup/resources/imsi.png" alt="drawing" width="800"/>
</p>


---

- Run the gNB

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.106prb.rfsim.conf
```

- Verify gNB connected to 5G Core:
```bash
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   3584 -> 0000e000
[NGAP]   Received NGSetupResponse from AMF
[GNB_APP]   [gNB 0] Received NGAP_REGISTER_GNB_CNF: associated AMF 1
```


- Run the UE  from a second terminal:

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --ssb 516 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```

---

Verify end-to-end connectivity: you should see the following in the logs:

**gNB:** 

Check for `RRC_CONNECTED` state:

```
[NR_MAC]    330. 0 UE b010: Received Ack of Msg4. CBRA procedure succeeded!
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI b010) Received RRCSetupComplete (RRC_CONNECTED reached)
```

Check for `Authentication`:
```
[NR_RRC]   UE 1 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI b010) Received Security Mode Complete
```

Check for `PDU sessions`:

```
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI c40a) Generate RRCReconfiguration (bytes 307, xid 3)
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI c40a) Received RRCReconfigurationComplete
[NR_RRC]   msg index 0, pdu_sessions index 0, status 2, xid 3): nb_of_pdusessions 1,  pdusession_id 10, teid: 2817854240 
[NR_RRC]   NGAP_PDUSESSION_SETUP_RESP: sending the message
```

---

**nrUE:**

Check for `RRC_CONNECTED` state:
```
[NR_RRC]   State = NR_RRC_CONNECTED
[NR_RRC]   [UE 0][RAPROC] Logical Channel UL-DCCH (SRB1), Generating RRCSetupComplete (bytes33)
```


Check for `Authentication`:
```
[NAS]   [UE] Received REGISTRATION ACCEPT message
```

Check for `oaitun_ue1` creation:
```
[NR_RRC]    Logical Channel UL-DCCH (SRB1), Generating RRCReconfigurationComplete (bytes 2)
[OIP]   Interface oaitun_ue1 successfully configured, IPv4 10.0.0.7, IPv6 (null)
```

Correspondingly, an interface should have been brought up:
```
oaitun_ue1: flags=209<UP,POINTOPOINT,RUNNING,NOARP>  mtu 1500
        inet 10.0.0.7  netmask 255.255.255.0  destination 10.0.0.7
        inet6 fe80::60bd:b117:1bb0:47c4  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 240 (240.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

# Wireshark traces

Log the traces using `tcpdump` as follows
```
sudo tcpdump -i oai-cn5g -w monolithic.pcap
```

---

# 5G Standalone Call flow

</p>
<p align="center">
<img style="display: block; margin: auto;" src="./vmsetup/resources/callflow.png" alt="drawing" width="800"/>
</p>

---

# Ping test


- Check the UE's IP address: interface `oaitun_ue1` using `ifconfig`
- The IP address `192.168.70.135` is the IP address of the `oai-ext-dn` container.
- Ping:
```
ping -I oaitun_ue1 192.168.70.135                 # from host, "UL", to oai-ext-dn
docker exec -it oai-ext-dn ping <UE IP address>   # from container, "DL"
```

# Iperf test

- Iperf3 testing between `oai-ext-dn` (in Docker) and the UE (running locally)
```
docker exec -it oai-ext-dn bash
iperf3 -s
```
- on the host in new terminal, run iperf3 client and bind the UE IP address:
```
iperf3 -B <UE IP ADDRESS> -c 192.168.70.135 -u -b 50M -R # DL
iperf3 -B <UE IP ADDRESS> -c 192.168.70.135 -u -b 20M    # UL
```


# Add a custom UE

Create a UE config file [ue2.conf](./conf/ue2.conf) with the following structure.
```
uicc0 = {
  imsi = "001010000000002";
  key = "fec86ba6eb707ed08905757b1bb44b8f";
  opc= "C42449363BBAD02B66D16BC975D77CC1";
  dnn= "oai";
  nssai_sst=1;
}
```
**Add this UE infomation to the core database in** [cn/database/oai_db.sql](./../cn/database/oai_db.sql)

---

Add the `imsi`, `key` and `opc` infomation of the UE to the `AuthenticationSubscription` table as follows
```
INSERT INTO `AuthenticationSubscription` (`ueid`, `authenticationMethod`, `encPermanentKey`, `protectionParameterId`, `sequenceNumber`, `authenticationManagementField`, `algorithmId`, `encOpcKey`, `encTopcKey`, `vectorGenerationInHss`, `n5gcAuthMethod`, `rgAuthenticationInd`, `supi`) VALUES
    ('001010000000002', '5G_AKA', 'fec86ba6eb707ed08905757b1bb44b8f', 'fec86ba6eb707ed08905757b1bb44b8f', '{\"sqn\": \"000000000000\", \"sqnScheme\": \"NON_TIME_BASED\", \"lastIndexes\": {\"ausf\": 0}}', '8000', 'milenage', 'C42449363BBAD02B66D16BC975D77CC1', NULL, NULL, NULL, NULL, '001010000000002');
```

Add the `imsi`, `dnn` and `nssai_sst` infomation of the UE to the `SessionManagementSubscriptionData` table as follows
```
INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
('001010000000002', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.3\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
```

Restart the `5G core` and use the config file [ue2.conf](./conf/ue2.conf) while running OAI UE

---

# Most common errors:

1. NGAP setup failure
2. Unidentified/Illegal UE
3. PDU session reject

**Debug:** wireshak traces and the logs

---

# Configuring TDD Pattern

The TDD pattern can be found in the ran [config file](./conf/gnb.sa.band78.106prb.rfsim.conf), an example TDD pattern with a periodicity 5ms can be seen below
```
# dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                 = 6;   # periodicity of the TDD Pattern
      nrofDownlinkSlots                                             = 7;   # DL slots per each TDD period
      nrofDownlinkSymbols                                           = 6;   # DL symbols in a mixed slot
      nrofUplinkSlots                                               = 2;   # UL slots per each TDD period
      nrofUplinkSymbols                                             = 4;   # UL symbols in a mixed slot
```

</p>
<p align="center">
<img style="display: block; margin: auto;" src="./vmsetup/resources/tdd_pattern_1.png" alt="drawing" width="900"/>
</p>

**Note:** The `subcarrier spacing` choosen for the above mentioned configurations is `30 KHz`

---

This can be configured as per our requirements, for example a TDD pattern with a periodicity 2.5ms can be seen below

```
# dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                 = 5;   # periodicity of the TDD Pattern
      nrofDownlinkSlots                                             = 2;   # DL slots per each TDD period
      nrofDownlinkSymbols                                           = 6;   # DL symbols in a mixed slot
      nrofUplinkSlots                                               = 2;   # UL slots per each TDD period
      nrofUplinkSymbols                                             = 4;   # UL symbols in a mixed slot
```

</p>
<p align="center">
<img style="display: block; margin: auto;" src="./vmsetup/resources/tdd_pattern_2.png" alt="drawing" width="1000"/>
</p>

**Excercise:** Perform iperf tests in both DL and UL. What do you observe?

---

# Changing bandwidth configuration

Change the bandwidth to `20MHz` configuration

Parameters that need to configured in the ran configuration file are

```
absoluteFrequencySSB                                        = 640704;  # SSB GSCN
dl_carrierBandwidth                                         = 51;
ul_carrierBandwidth                                         = 51;
initialDLBWPlocationAndBandwidth                            = 13750;   # RIV
initialULBWPlocationAndBandwidth                            = 13750;   # RIV
```

To run gNB

```
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.51prb.rfsim.conf
```

To run UE

```
sudo ./nr-uesoftmodem -r 51 --numerology 1 --band 78 -C 3609300000 --rfsim --ssb 228 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```

Resources for calculating [SSB GSCN](https://5g-tools.com/5g-nr-gscn-calculator/) and resources for calculating [RIV](https://www.sqimway.com/rb_calc.php)

---

similarly, a configuration file for `100MHz` has been provided [gnb.sa.band78.273prb.rfsim.conf](./conf/gnb.sa.band78.273prb.rfsim.conf)

To run gNB
```
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.273prb.rfsim.conf
```

To run UE
```
sudo ./nr-uesoftmodem -r 273 --numerology 1 --band 78 -C 3649260000 --rfsim --ssb 516 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```
---

# CU-DU F1 split

To start CU:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb-cu.sa.f1.conf
```

To start DU:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb-du.sa.band78.106prb.rfsim.conf
```
Run the UE:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --ssb 516 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```
---

# Multiple UE's

To start gNB:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.106prb.rfsim.conf
```

To start UE1:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ~/ieee_ants2024_oai_tutorial/ran/multi-ue.sh -c1 -e
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim -O /home/rakeshmundlamuri7/ieee_ants2024_oai_tutorial/ran/conf/ue1.conf --rfsimulator.serveraddr 10.201.1.100
```

To start UE2:
```
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ~/ieee_ants2024_oai_tutorial/ran/multi-ue.sh -c2 -e
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim -O /home/rakeshmundlamuri7/ieee_ants2024_oai_tutorial/ran/conf/ue2.conf --rfsimulator.serveraddr 10.202.1.100
```

---

To login to ue1 namespace

```
sudo ip netns exec ue1 bash
```

To login to ue1 namespace

```
sudo ip netns exec ue2 bash
```

You can now perform ping and iperf tests

ping from UE1

```
ping -I oaitun_ue1 192.168.70.135                 # from host, "UL", to oai-ext-dn
```

ping from UE2

```
ping -I oaitun_ue1 192.168.70.135                 # from host, "UL", to oai-ext-dn
```


To remove the namespaces

```
sudo ~/ieee_ants2024_oai_tutorial/ran/multi-ue.sh -d1 -d2
```

