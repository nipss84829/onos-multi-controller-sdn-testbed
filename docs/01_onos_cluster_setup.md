# 01. ONOS Multi-Controller Cluster Setup

This document describes how to build a 4-node ONOS multi-controller cluster using Docker and Atomix.

## Reference

- YouTube: https://www.youtube.com/watch?v=vjmhVE50G_c

---

## Overview

This setup uses:

- **ONOS 2.2.2**
- **Atomix 3.1.5**
- **Docker**
- **4 Atomix containers**
- **4 ONOS containers**

> Note: ONOS and Atomix versions must match properly.  
> Example: **ONOS 2.2.2 ↔ Atomix 3.1.5**

---

## Step 1. Pull Atomix Docker Image

docker pull atomix/atomix:3.1.5

## Step 2. Run Four Atomix Containers

docker run -t -d --name atomix-1 atomix/atomix:3.1.5<br>
docker run -t -d --name atomix-2 atomix/atomix:3.1.5<br>
docker run -t -d --name atomix-3 atomix/atomix:3.1.5<br>
docker run -t -d --name atomix-4 atomix/atomix:3.1.5

## Step 3. Check Docker IPs of Atomix Instances

docker inspect atomix-1 | grep -i ipaddress<br>
docker inspect atomix-2 | grep -i ipaddress<br>
docker inspect atomix-3 | grep -i ipaddress<br>
docker inspect atomix-4 | grep -i ipaddress

## Step 4. Get ONOS Source Code
git clone https://gerrit.onosproject.org/onos

Note: You should choose an ONOS source version that is actually compatible and usable in your environment.

## Step 5. Set Environment Variables for Atomix Docker IPs

export OC1=172.17.0.2<br>
export OC2=172.17.0.3<br>
export OC3=172.17.0.4<br>
export OC4=172.17.0.5

## Step 6. Generate Atomix Configuration Files

You need to modify the output path based on your environment.

Also, because the original Python script is old, you may need to modify the atomix-gen-config script:

change print output to print(output)<br>

Generate the configuration files:<br>
./tools/test/bin/atomix-gen-config 172.17.0.2 /home/atomix-1.conf 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5<br>
./tools/test/bin/atomix-gen-config 172.17.0.3 /home/atomix-2.conf 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5<br>
./tools/test/bin/atomix-gen-config 172.17.0.4 /home/atomix-3.conf 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5<br>
./tools/test/bin/atomix-gen-config 172.17.0.5 /home/atomix-4.conf 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5

## Step 7. Copy Atomix Configuration Files into Containers
docker cp /home/wcmc/atomix-1.conf atomix-1:/opt/atomix/conf/atomix.conf<br>
docker cp /home/wcmc/atomix-2.conf atomix-2:/opt/atomix/conf/atomix.conf<br>
docker cp /home/wcmc/atomix-3.conf atomix-3:/opt/atomix/conf/atomix.conf<br>
docker cp /home/wcmc/atomix-4.conf atomix-4:/opt/atomix/conf/atomix.conf

## Step 8. Restart Atomix Containers
docker restart atomix-1<br>
docker restart atomix-2<br>
docker restart atomix-3<br>
docker restart atomix-4
## Step 9. Pull ONOS Docker Image
You can either pull an official image or build your own.<br>
docker pull onosproject/onos:2.2.2
## Step 10. Run Four ONOS Containers
### Official image example:
docker run -t -d --name onos1 onosproject/onos:2.2.2<br>
docker run -t -d --name onos2 onosproject/onos:2.2.2<br>
docker run -t -d --name onos3 onosproject/onos:2.2.2<br>
docker run -t -d --name onos4 onosproject/onos:2.2.2<br>
### Self-built image example:
docker run -t -d --name onos1 myonos:2.2.2<br>
docker run -t -d --name onos2 myonos:2.2.2<br>
docker run -t -d --name onos3 myonos:2.2.2<br>
docker run -t -d --name onos4 myonos:2.2.2
## Step 11. Check Docker IPs of ONOS Instances
docker inspect onos1 | grep -i ipaddress  
docker inspect onos2 | grep -i ipaddress  
docker inspect onos3 | grep -i ipaddress  
docker inspect onos4 | grep -i ipaddress
## Step 12. Generate ONOS Cluster Configuration Files
./tools/test/bin/onos-gen-config 172.17.0.6 /home/wcmc/cluster-1.json -n 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5  
./tools/test/bin/onos-gen-config 172.17.0.7 /home/wcmc/cluster-2.json -n 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5  
./tools/test/bin/onos-gen-config 172.17.0.8 /home/wcmc/cluster-3.json -n 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5  
./tools/test/bin/onos-gen-config 172.17.0.9 /home/wcmc/cluster-4.json -n 172.17.0.2 172.17.0.3 172.17.0.4 172.17.0.5  
## Step 13. Create Configuration Directory Inside ONOS Containers
docker exec onos1 mkdir /root/onos/config  
docker exec onos2 mkdir /root/onos/config  
docker exec onos3 mkdir /root/onos/config  
docker exec onos4 mkdir /root/onos/config
## Step 14. Copy Cluster Configuration into ONOS Containers
docker cp /home/cluster-1.json onos1:/root/onos/config/cluster.json  
docker cp /home/cluster-2.json onos2:/root/onos/config/cluster.json  
docker cp /home/cluster-3.json onos3:/root/onos/config/cluster.json  
docker cp /home/cluster-4.json onos4:/root/onos/config/cluster.json
## Step 15. Restart ONOS Containers
Restart the ONOS containers so the cluster configuration can take effect.  
docker restart onos1  
docker restart onos2  
docker restart onos3  
docker restart onos4
## Step 16. Access the ONOS GUI
URL: http://172.17.0.7:8181/onos/ui  
Username: onos  
Password: rocks  
## Step 17. Enable OpenFlow-Related Apps
You need to enable OpenFlow-related functionality so ONOS can detect the Mininet topology.  
Reference:https://www.youtube.com/watch?v=F7YXWuNCWlg
## Step 18. Enter Each Controller
Example using onos2:  
docker exec -it onos2 bash  
cd ~/onos/apache-karaf-4.2.8/bin  
./client  
After entering the ONOS CLI, you can activate the GUI app if needed:  
app activate gui2  
If it works. It will be like the following picture  
![圖片1](../images/圖片1.png)
## Cleanup and Restart Commands
### If Docker shuts down unexpectedly or you want to rebuild the environment, you can run the following commands:  
sudo docker stop onos1 onos2 onos3 onos4 atomix-1 atomix-2 atomix-3 atomix-4  
sudo docker rm onos1 onos2 onos3 onos4 atomix-1 atomix-2 atomix-3 atomix-4  
sudo pkill -f karaf  
sudo pkill -f onos  
sudo pkill -f atomix  
sudo pkill -f java  
sudo lsof -i :6653 -i :8101 -i :8181 -i :8182 -i :9876 -i :9877 -i :5701  
sudo ovs-vsctl --all destroy Controller  
sudo ovs-vsctl --all destroy Manager  
for br in $(sudo ovs-vsctl list-br); do sudo ovs-vsctl del-br $br; done  
sudo systemctl restart openvswitch-switch  
sudo fuser -k 6653/tcp 8101/tcp 8181/tcp 8182/tcp 9876/tcp 9877/tcp 5701/tcp  
sudo systemctl restart docker  
sudo docker network prune -f  
sudo systemctl restart openvswitch-switch  

# Rebuild Procedure
After cleanup, repeat Step 1 to Step 16.  
If the environment still does not work:  
Reboot the virtual machine  
Check whether NAT networking is broken  
Restart the VMware network adapters  
Reset VM networking entirely if needed  
VMware networking notes  
VMnet8 = NAT  
VMnet1 = Host-only  
If necessary:  
shut down the VM  
reinitialize the NAT adapter  
reset the VM network  

## Configure Flow Table via REST API
### Use the ONOS REST API to install flow rules.
Note: If you do not use this method and directly apply rules with sudo, they may be cleared later.  
curl -u onos:rocks -X POST \
  -H "Content-Type: application/json" \
  http://172.17.0.6:8181/onos/v1/flows/of:0000000000000001 \
  -d '{
"priority": 0,
"timeout": 0,
"isPermanent": true,
"deviceId": "of:0000000000000001",
"appId": "org.onosproject.core",
"selector": { "criteria": [] },
"treatment": { "instructions": [ { "type": "OUTPUT", "port": "CONTROLLER" } ] }
}'
### Check Switch Flow Table
sudo ovs-ofctl dump-flows s1
### Delete Switch Flow Table
sudo ovs-ofctl del-flows s1