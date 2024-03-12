# network_namespaces-
A network namespace in Linux is a feature that allows you to create multiple isolated instances of network stacks on a single Linux system. Each network namespace has its own set of network interfaces, routing table, firewall rules, and sockets.

To show all the namespaces in your machine:

	 ip netns
	
To add a namespace :

	ip netns add namespace_name

For reasoning, let's make two namespaces :

	sudo ip netns add one
	sudo ip netns add two

![docker image 1](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/ed53efc9-cab7-47e1-b9e9-d200b873c42a)

If you wanna check if they are actually isolated or not, you can just compare the outputs of the commands:

	ip link // to check your whole machine 
	sudo ip netns exec one ip link  or  sudo ip -n one link // to check the namespace one
	
It's time to Establish Network Connectivity. We will create a cable to connect it from namespace one to two. To do that :

	sudo ip link add veth-one type veth peer name veth-two
 ![image 2](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/41250efe-06b7-4a53-b0e6-da456f306233)


The Connection is established . We have to assign each interfaces ( cable's endpoint) to their respective namespaces. To do that:

	sudo ip link set veth-one netns one
	sudo ip link set veth-two netns two
 
 ![image 3](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/05f26535-e6f3-4019-ac0c-5fa9b57620ee)


Now Assign ip address to each namespaces :

	sudo ip -n one addr add 192.168.1.1/24 dev veth-one
	sudo ip -n two addr add 192.168.1.2/24 dev veth-two
 
 ![image 4](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/5b81542a-1930-401a-899b-7aa4e1774cac)

	
To Bring up the connection from both end:

	sudo ip -n one link set veth-one up
	sudo ip -n two link set veth-two up
	
Now to check if one can communicate with two :

	sudo ip netns exec one ping 192.168.1.2
	
	
Connection between two namespaces is set up. But if we have multiple namespaces then we should use a gateway to communicate with one another . So create a bridge, you need to first delete the previous connection between interfaces.
	sudo ip -n one link del veth-one

Create a Bridge and bring it up:

	sudo ip link add v-net-0 type bridge
	sudo ip link set dev v-net-0 up

 ![image 7](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/585798e3-1fb9-43ed-b21a-ab7b092f2fa6)

	
Now new set up for interfaces connection :

	sudo ip link add veth-one type veth peer name veth-one-br
	sudo ip link add veth-two type veth peer name veth-two-br
 
![i8](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/67ff5d65-f525-4a1d-907e-ab2062c3fccb)
 
	
Assigning interfaces to their respective namespaces:

	sudo ip link set veth-one netns one
	sudo ip link set veth-two netns two
 
![i9](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/cd7872a8-9490-45c4-94c8-6b5c89715892)

 
	
Assigning interfaces (another terminal to v-net-0/Bridge):

	sudo ip link set veth-one-br master v-net-0
	sudo ip link set veth-two-br master v-net-0

 ![i10](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/c09816db-a6d7-4c95-ae82-1a2b0b09c052)

	
Assigining ip addresses to the namespaces and bridge:

	sudo ip -n one addr add 192.168.15.1/24 dev veth-one
	sudo ip -n two addr add 192.168.15.2/24 dev veth-two
	sudo ip addr add 192.168.15.5/24 dev v-net-0
 ![i11](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/e443c984-15d4-44e4-9831-0b834f42f9f2)

	
Bring up the interfaces:

	sudo ip -n one link set veth-one up
	sudo ip -n two link set veth-two up
	sudo ip link set dev veth-one-br up                                                                            
	sudo ip link set dev veth-two-br up
	
Now ping to check if from local machine, the interfaces can be reached or not:

	ping 192.168.15.1
	
As of now, we have successfully communicated from local machine to any namespaces. Now what if the local namespaces want to connect with outside internet, like google 

If you try 

	sudo ip netns exec one ping 8.8.8.8 
	
it will show the network is unreachable , beacuse there is no entry in routing table for default gateway

To recognize your private ip to the outer network , you should enable NAT, for that:

	iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
	
So we have to add entry to the routing table

	sudo ip netns exec one ip route add 192.168.0.0/24 via 192.168.15.5
 ![image shesh hoy na](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/d2bd2d0f-180d-45b5-84ac-23cd3d29599a)


	sudo ip netns exec one ip route add default via 192.168.15.5
 ![image not last](https://github.com/RafsanBinAli/network_namespaces-/assets/154937557/25cf28e8-19aa-4da4-bd5c-6758cced6ce7)




