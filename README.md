# Welcome to PennCloud :cloud:
- Adam Gorka @AdamEGorka, Andrew Lukashchuk @lukashchu, Hassan Rizwan @hrizwan3, Jiwoong (Matt) Park @mtp0326
- 11/08/2024 -

## Introduction
PennCloud is a storage and communication platform that allows users to store files, send emails, and chat with other users. It is designed to be user-friendly and efficient, with distributed servers and fault tolerance to ensure that the platform is always available. The platform is built with a frontend load balancer, frontend servers, backend servers, and an SMTP server to handle the various functionalities.

## Deployed Link
Coming Soon!

## Features

### Storage Drive
<img src="media/storagedrive.gif" width="960" alt="file"/>

In chronological order:
- Add folders to the root directory
- Add files to a specified folder
- Move a file to a different path
- Rename a file
- Download a file

### Send Email to Another PennCloud User
![email](media/sendemail(internal).gif)
- We see two separate tabs, where one in the yellow tab is the sender and the other in the black tab is the receiver. The email is sent and received through SMTP protocol.

### Send Email to an Exernal Email Address
![email](media/sendemail(external).gif)
- The emails can also be sent to external email addresses such as Gmail.

### Chat
![chat](media/chat.gif)
- We again see two separate tags, where the yellow tab is Adam and the black tab is Admin. The chat is sent and received in real time and ordered chronologically.

## Flowchart of the Architecture
Our design overall consists of various components. There is a frontend load balancer, which the user connects to, which then redirects the user to one of the available frontend servers. The frontend servers then interact with the backend coordinator, which gives them an appropriate backend storage server to connect to and handle the request. Additionally, there is an extra frontend server that acts as the SMTP server to support retrieving mail from external sources and then directing that mail to the backend by first going to the backend coordinator as with the other frontend servers.

![flowchart](media/flowchart.png)

### General Workflow of Transaction:
1. When the user sends a transaction request from the webpage, the request is sent to the frontend load balancer. The frontend load balancer then provides the user with the ip address and port of the frontend server.
2. The user then sends a connection and transaction request to the indicated frontend server.
3. The frontend server then sends a transaction request to the backend load balancer to get the ip address and port of an available backend group. The load balancer then provides the ip address and port of the primary node for the available backend group.
4. The frontend server then sends the transaction request to the primary node of the backend group.
5. The backend group performs 2PC protocol to process the request (will be specified in the 2PC Backend Transaction Process).
6. The primary node will then return the result to the frontend server.
7. The frontend server will then return the result to the user.

### 2PC Backend Transaction Process
**Initial Configuration:** the backend load balancer will first assign the primary and subordinate nodes to the primary backend node for each group, to which the primary backend node will then send the configuration to the subordinate nodes.

**Transaction:**
1. The frontend server will first send the transaction request to the primary node of the backend group. If the transaction is a GET request, the primary node immediately returns the result to the frontend server. If the transaction is a PUT, PUTV, or DELETE request, the primary node will perform a 2PC protocol with the other subordinate nodes in the backend group. For the 2PC protocol, the primary node will first send a PREP message to the subordinate nodes, to which the nodes respond with a ACK or REJECT message.
2. If all the subordinate nodes respond with ACK, the primary node will perform the transaction then send a COMMIT message to the subordinate nodes for them to perfrom their transactions. If any of the subordinate nodes respond with REJECT, the primary node will send an ABORT message to the subordinate nodes.


## 2PC Protocol and Fault Tolerance
The logs represent the three nodes in the same backend group. The leftmost log is the primary node, and the other two are the subordinate nodes.

### Fully functioning group (3 Nodes):
![3nodes](media/3nodes.gif)
- From the login, we can see the transaction with the 2PC protocol is working well. We see the primary node sending the PREP message to the subordinate nodes, and the subordinate nodes responding with ACK. The primary node then sends a COMMIT message to the subordinate nodes before doing its transaction, and the subordinate nodes perform their transactions.

### 1 Faulty Node (2 Nodes):
![2nodes](media/2nodes.gif)
- We shut down the primary node, which will be caught by the heartbeat check. The two subordinate nodes then are assigned a new primary node as seen in "New configuration". We can see that changing the user's password still works well with 2 nodes and their interaction.

### 2 Faulty Nodes (1 Node):
![1node](media/1node.gif)
- We shut down the primary node from the 2 nodes once again. We see that the node with the log to the right is the new primary node and is able to perform the chat transaction with ease.
