[TOC]



## 摘要

目的是为了支持高校的研究者运行它们的网络实验协议，评估他们的想法在现实的设置。斯坦福已经有两座大楼将会很快运行OpenFlow，他们接下来也会鼓励其他学校这么部署。

OpenFlow is based on an Ethernet switch, with an internal flow-table, and a standardized interface to add and remove flow entries .



## 1 对可编程网络的需要

There is almost no practical way to experiment with new network protocols in sufficiently realistic settings to gain the confidence needed for their widespread deployment . The result is that most new ideas from the networking research community go untried and untested （）

Having recognized the problem, the networking community is hard at work developing programmable networks 

Virtualized programmable networks could lower the barrier to entry for new ideas, increasing the rate of innovation in the network infrastructure. But the plans for nationwide facilities are ambitious (and costly), and it will take years for them to be deployed. 

This whitepaper focuses on a shorter-term question closer to home: **As researchers, how can we run experiments in our campus networks**? 

To meet this challenge, several questions need answering,including: 

1. In the early days, how will college network administrators get comfortable putting experimental equipment (switches, routers, access points, etc.) into their network?

2. How will researchers control a portion of their local network in a way that does not disrupt others who depend on
   it? 

3.  what functionality is needed in network switches to enable experiments? 

Our goal here is to propose a new switch feature that can help extend programmability into the wiring closet of college campuses. 

One approach - that we do not take - is to persuade commercial “name-brand” equipment vendors to provide an
open, programmable, virtualized platform on their switches and routers so that researchers can deploy new protocols, while network administrators can take comfort that the equipment is well supported 

the commercial solutions are too closed and inflexible, and the research solutions either have insufficient performance or fanout, or are too expensive 

## THE OPENFLOW SWITCH 





The datapath of an OpenFlow Switch consists of a Flow Table, and an action associated with each flow entry . The
set of actions supported by an OpenFlow Switch is extensible, but below we describe a minimum requirement for
all switches. 



### Dedicated OpenFlow switches

![1573973429117](image/1573973429117.png)

An OpenFlow Switch consists of at least three parts :

1. A Flow Table, with an action associated with each flow entry, to tell the switch how to process the flow 
2. A Secure Channel that connects the switch to a remote control process (called the controller) 
3. The OpenFlow Protocol, which provides an open and standard way for a controller to communicate with a switch 

Each flow-entry has a simple action associated with it; the three basic ones (that all dedicated OpenFlow switches must support) are: 

1. Forward this flow’s packets to a given port (or ports) 
2. Encapsulate and forward this flow’s packets to a controller 
3. Drop this flow’s packets 

An entry in the Flow-Table has three fields: 

1. A packet header that defines the flow 
2. The action, which defines how the packets should be processed 
3. Statistics, which keep track of the number of packets and bytes for each flow, and the time since the last packet matched the flow (to help with the removal of inactive flows 

In the first generation “Type 0” switches, the flow header is a 10-tuple shown in Table 1 :

![1573975826231](image/1573975826231.png)

Each header field can be a wildcard to allow for aggregation of flows .

### OpenFlow-enabled switches 

![1573976096365](image/1573976096365.png)

为了实现让实验测试流量和生产流量分开来，OpenFlow-enabled switches 必须实现以下两种方法之一：

1. 增加第4个action：Forward this flow’s packets through the switch’s normal processing pipeline 
2. define separate sets of VLANs for experimental and production traffic 

### Additional features 

