# Expressroute 802.1ad (Q-in-Q)- draft
Some Expressroute service providers require customers to use 802.ad tunneling (aka Q-in-Q) for circuit termination. Other Expressroute Service Providers have the ability to terminate Q-in-Q for the customer and provide Q trunks, VLAN tagged, or untagged packets to the customer. Expressroute details are outside the scope of this document. Other important items to consider when terminating Q-in-Q for Expressroute:

- Ethertype must be 0x8100
- A single Expressroute circuit is made up of 2 physical paths. Each physical path requires BGP peering to the Micrsoft Edge Routers (MSEEs).
- It's highly recommended that customers work with their provider to determine if the customer will receive 1 or 2 physical hand offs and the impact on path redundancy.
- Expressroute requires EBGP and does not support multihop.
- C-tags (inner tags) is a customer configurable VLAN you assign to your Expressroute circuit. Both paths for your circuit will be assigned the same VLAN (ex VLAN 100). More on that later.
- S-tags (outer tags) are not customer configurable in Expressroute. S-tags are handled by your provider and Azure to ensure they are unique throughout their network. (ex VLAN 1000). There are 2 Micrsoft edge routers (MSEEs) that provide connectivity for a single Expressroute circuit. MSEE1/2 will send out the same S-tag for both paths.
- If the provider terminates Q-in-Q on behalf of the customer, Azure and the provider will take care of adding/removing the S-tag.
- If the customer is terminating Q-in-Q, the customer edge gear will receive S-tag 1000, strip the outer tag and will be presented the inner tag 100. It's CRITICAL to understand the physcal handoffs you receive from the provider, the hardware/software you are using for termination and if you will also be terminating BGP on the same gear (not all hardware/software supports that combination). 
- You can acquire the S-tag from your provider if you are terminating Q-in-Q on customer equipment. The S-tag between Azure and the provider can't be changed. Some providers will allow you to change the S-tag between the provider edge and customer edge if the customer needs a specific S-tag.


This guide will show Q-in-Q basic configs as well as more advanced configurations specific to Expressroute topologies. All configurations are done using simulation software so syntax may be slightly different. 

# Basic Q-in-Q topology, configuration and order of operations. This is not an Expressroute topology.
![alt text](https://github.com/jwrightazure/lab/blob/master/Expressroute-Q-in-Q/q-in-q-topo.PNG)

In the above topology, R1 and R2 interfaces are on the same subnet seperated by the service provider Q-in-Q network. R1 and R2 interfaces will tag packets as VLAN 100. The service provider switches will tunnel any tagged packets it receives from the customer with S-tag 1000. The service provider network knows nothing about customer VLAN 100 (C-tag) and simply switches VLAN 1000 (S-tag) throughout their network. In this topology, the service provider switches facing the customer are responsible for stripping the S-tag.

# Tracing a packet when pinging from R1 to R2
![alt text](https://github.com/jwrightazure/lab/blob/master/Expressroute-Q-in-Q/packet-capture-summary.PNG)

The trace shows that R1 will tag packets with VLAN 100 when pinging R2 and ethertype 0x0800. SW1 will apply S-tag 1000 with ethertype 0x8100 on top of VLAN 100 and forward it to SW2. SW2 is simply switching VLAN 1000 packets. SW3 will strip the S-tag 1000, and forward the packet to R2 with a tag of 100 ethertype 0x0800.

# Back to back Q-in-Q 
![alt text](https://github.com/jwrightazure/lab/blob/master/Expressroute-Q-in-Q/q-in-q-b2b.PNG)

In the back to back example, you can see that you can have multiple S-tags (1000,2000) arrive on different logical interfaces and present the same C-tag (100). The 0x9100 ethertype can be ignored as this is specific to my lab. When a customer terminates Q-in-Q on a single physical interface (2 logical interfaces), the S-tag must be unique. If the customer terminates Q-in-Q on seperate physical interfaces on the same device, the S-tag can be the same. Please check your hardware/software to validate.
