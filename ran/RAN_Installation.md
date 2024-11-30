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

Run the gNB

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem --rfsim -O ~/ieee_ants2024_oai_tutorial/ran/conf/gnb.sa.band78.106prb.rfsim.conf
```


Run the UE from a second terminal:

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --ssb 516 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```

Verify that it is connected: you should see the following output at gNB:

```
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI c40a) Generate RRCReconfiguration (bytes 307, xid 3)
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI c40a) Received RRCReconfigurationComplete
[NR_RRC]   msg index 0, pdu_sessions index 0, status 2, xid 3): nb_of_pdusessions 1,  pdusession_id 10, teid: 2817854240
[NR_RRC]   NGAP_PDUSESSION_SETUP_RESP: sending the message

```

and nrUE:

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

Some other things to check:
- You see the RA procedure of the UE in the gNB logs

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
```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --ssb 516 -O ~/ieee_ants2024_oai_tutorial/ran/conf/ue.conf
```

