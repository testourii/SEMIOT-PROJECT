# I Introduction

Node-RED is a powerful tool used in building Internet of Things (IoT) applications with
a focus on simplifying the ’wiring together’ of code blocks to carry out tasks. It also uses
a visual programming approach that allows developers to connect predefined code blocks,
known as ’nodes’, together to perform a task.
When wired together, the connected input, processing or output nodes make up ’flow’. Im-
portantly, Node-RED has rapidly developed a significant and growing user base and an active
developer community which is helping to create new nodes that allow programmers to reuse
Node-RED code for a wide variety of tasks.
In order to secure all parts (nodes) contained in the work flow, end-to-end security is re-
quired and implies tracking information flow through all the system services and analyzing
data dependencies.
Typically, forwarding confidential data to unauthorized services has to be detected. Con-
sequently, data dependence has to be identified and explicitly exposed, otherwise, security
composition can induce security leaks called interference.
For data dependency, we propose SEMIoT (Security Manager for Internet of things) that
generates a security data graph. Starting from a Node-RED work-flow and its hidden steps,
in fact SEMIoT calculates this data flow and creates a new Node-RED work-flow representing
the DDG^1.
This document classifies Node-RED nodes and explains the SEMIoT graph generation .A
use-case is illustrated

## I.1 SEMIoT Node

In order to secure the work flow ,the creation of a security graph generator by Node-RED is
necessary.
Each node is called SEMIoT node and has two attributes :

**Security label :** represents the security degree of the data, this degree increases with the
confidentiality of the inputs.

**Label type :** can be either required or provided labels depending on the type of services
(managed or external). Required labels are immutable and represent external con-
straints that simply cannot be changed by the administrator. In contrary, Provided
labels are set on managed resources and can be modified by the administrator when
needed.




Figure1 represents SEMIoT user interface for configuring a SEMIoT node.
This interface contains the graphical representation of a SEMIoT node, the security label
and type are displayed (in the figure, security label is 3 and label type is required)
![](http://image.noelshack.com/fichiers/2018/38/4/1537473863-readme-000.png)
```
Figure 1: A SEMIoT node configuration
```
## II.1 installation
In order to install the **SEMIoT** node module :

 1. Download **SEMIoT Node directory**
 2. In the downloaded directory containing the node’s package.json file, run: ***sudo npm link***
 3. In your node-red user directory, typically **~/.node-red**,  run: ***npm link node-red-contrib-example-SEMIOT***

      
        
## II.2 Nodes classification

In order to generate a security graph from the work flow, we need to classify all the IoT
system’s nodes into three groups:

**Input nodes:** an input node is easy to identify as it only has one connecting dot on the
right-hand side (the output connector) and a missing connecting dot on the left (the
input connector). Though it can’t be connected to another input node due to a lack of
input connector , such nodes has also the ability to :

- Trigger the start of a flow and create the initial information packet
- To get triggered from an external operation such as a HTTP request or a trigger
    button.

![](http://image.noelshack.com/fichiers/2018/38/4/1537473997-readme-002.png)
```
Figure 2: Input node
```
**Input/Output Nodes :** For example, we have as illustartes in figure 3, a function node
generally contains:

- A single input connector on which it receives data ,executes the function code
    using the input information.
- One(or more) output connector(s) used to send data to the next node.

![](http://image.noelshack.com/fichiers/2018/38/4/1537473997-readme-003.png)
```
Figure 3: Function Node
```
**Output Nodes :** an output node sends data from the work-flow to external services. It
can be spotted as it is missing the output connector.

![](http://image.noelshack.com/fichiers/2018/38/4/1537473997-readme-004.png)
```
Figure 4: Nodes
```
**Note:** In a work-flow, the system architecture, node properties and connectors are discriped
in JSON^2 code.
This JSON code containes other extra hidden objects like flow ID, MQTT broker and
KAFKA server. These objects need to be explicitly exposed when generating the DDG.

## II.3 Security graph generator

For SEMIoT user, the security graph generation process is descriped as Node-RED work-flow.
As presented In figure 5, it contains 3 nodes:

**IoT Work-Flow:** is a flow reader node. It retrieves the user IoT work-flow from a Node-
Red server and returns the JSON code of the work-flow.

**Graph Generator:** is a function node that contains a JavaScript function block that takes
the work flow’s JSON code and uses it to generate the DDG to finally return it in
another JSON code.

**SEMIoT Graph:** is a file node that writes the returned code on a file, exports it and gets
the DDG.
![](http://image.noelshack.com/fichiers/2018/38/4/1537473997-readme-005.png)
```
Figure 5: A graph generation process described as a Node-RED work-flow
```
## II.4 Algorithm

**Input :** an IoT work-flow’s JSON code.
The figure 6-a represents an extract of input code which contains an array of objects
.Each object presents a node’s definition (name, type, position in the flow (x and y),
wires...)

**Output :** a DDG JSON code. It contains the same objects but after removing the extra
nodes, generating the DDG. Just like in the figure 6-b.

![](http://image.noelshack.com/fichiers/2018/38/4/1537473997-readme-006.png)
```
Figure 6: a-Extract of an input JSON Code
b-Extract of an output JSON Code
```

**Graph generator code :** Our generator function contains an implementation in java-
script of this algorithm below(Algorithm 1)
**Input** : IoTG (IoT graph)
**Output** : DDG (Data Dependence Graph)

**Algorithm 1** DDG generator

```
1: for each node n ∈ IoTG do. make wires between mqtt(or kafka) nodes with the same
topic
2: if n is MQTT ( or KAFKA ) input Node then
3: for each node M ∈ IoTG do
4: if M is MQTT ( or KAFKA ) output Node then
5: if ( M and n ∈ Same Broker ) and ( M and n ∈ Same Topic ) then
6: add ( M, wires ( n )). wires(n): list of succesors nodes in the IoTG
7: end if
8: end if
9: end for
10: end if
11: end for
12: for each n ∈ IoTG do. begin the generation
13: if n is input Node then
14: for each outputi ∈ n do
15: create node ni
16: add ( ni, DDG )
17: end for
18: else if n is output Node then
19: add ( n, DDG )
20: else if n is input/output Node then
21: add ( n, DDG )
22: for each outputi ∈ n do
23: create node ni
24: add ( ni, DDG )
25: for each j ∈ wires ( outputi ) do
26: add ( j, wires ( ni ))
27: end for
28: add ( ni, wires ( n ))
29: end for
30: end if
31: end for
32: return DDG
```

```
The algorithm 1 handles 2 cases:
```
**output nodes:** in this case, this node just replaced by a single SEMIoT node without any
modification just like represents in the figure below.
![](http://image.noelshack.com/fichiers/2018/38/4/1537474892-readme-008.png)
```
Figure 7: Output node and his correspendant in DDG
```
**Input/ouput nodes:** in this case, each output connector will replaced by a SEMIoT node
without forget to set to this node the same wires of the connector.

![](http://image.noelshack.com/fichiers/2018/38/4/1537474892-readme-009.png)
```
Figure 8: Input/output node and his correspendant in DDG
```

# II Use-case: Home automation

In figure 9, we present a work flow of home automation use case. To make it simple, we
created a sub-flow that contains all the nodes necessary to make a smart house. This smart
house(presented in figure 10)holds the sensors and the actuators necessary for home obser-
vation remote.
”House 01” and ”House 02” are 2 sub-flows deployed in the gateways.
Master work-flow is deployed in the server-side. It communicates with a set of remote services
like:

- healthcare service
- emergency service
- storage service
- payment service
- research lab

![](http://image.noelshack.com/fichiers/2018/38/4/1537474892-readme-010.png)
```
Figure 9: Home automation work-flow
```
![](http://image.noelshack.com/fichiers/2018/38/4/1537474892-readme-011.png)
```
Figure 10: House 01 subflow
```
In figure 11, we present the DDG which creates wires between the MQTT nodes and
replace each one with an SEMIoT input node and SEMIoT output node to set the security
parameters without changing the subflow nodes. But as we can see in figure 12, the nodes
in the sub flow were also replaced with an SEMIoT node.
![](http://image.noelshack.com/fichiers/2018/38/4/1537475035-readme-012.png)
```
Figure 11: DDG for the master flow
```
![](http://image.noelshack.com/fichiers/2018/38/4/1537475036-readme-013.png)
```
Figure 12: DDG for the sub-flow
```

