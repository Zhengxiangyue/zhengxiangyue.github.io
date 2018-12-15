## Understand TCP protocol 

### Overview

TCP is a **connection-oriented** and **reliable transport** protocol. 

Before two processes can communicate with each other, a **connection** needs to be established. The connection here is only some cache and state variables allocated in the end system, the middle switch does not maintain any connection status information.

The connection starts with three data transfer. First, client send a piece of special TCP segment to server. Then, server response with a piece of special TCP segment. Finally, client response to server with another piece of special TCP segment. We will understand them later.

Who is using and understanding the TCP packets? Which program in what kind of device is responsible for interpreting?  I would say it's the end computer and the program is implemented by Operating System. To understand that, we should first look at the form of a TCP segment and what is each bit used for.

### TCP Segment Form

As we know, in TCP/IP protocol stack, each layer add its own header to the lower layer data. The total length of a TCP packet is uncertain, determined by a 4-bit part in the header. 

The header of TCP consists of several parts:

- 16 bits source port address, identifying which process is sending this data packet
- 16 bits destination port address. IP protocol makes sure the data packet is received by the end computer and then the operating system or the driver application distributes the data segment to a specific process based on this port number
- 32 bits sequence number indicating the order of the segments
- 32 bits acknowledgement number
- 4 bits header length, the interpreting program segment header and data based on this length
- 6 bits
- 1 bit URG, indicating if **urgent pointer** is valid
- 1 bit ACK, indicating if acknowledgement number is valid
- 1 bit PSH, request for push
- 1 bit RST, reset the connection
- 1 bit SYN, Synchronize sequence numbers
- 1 bit FIN, 
- 16 bits window size, control the amount of data sent by the other end in bytes. One end of TCP connection determines the size of its receiving window according to the size of the set buffer space, and then notifies the other party to determine the upper limit of the sending window of the other party
- 16 bits Checksum, the scope of checking includes the header and data. When calculating the checksum, a 12-byte pseudo header is added to the front of the TCP segment
- 16 bits urgent pointer, indicates the sequence number of the last byte of the urgent data in this segment
- Optional field, of which the length is not fixed. The TCP header can have up to 40 bytes of optional information for passing additional information to the destination of for aligning other options. This part contains up to 40 bytes, because the TCP header is up to 60 bytes(contains the previous 20-byte fixed part)

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### TCP connection status

TCP connection life cycle status

1. LISTEN, wait for a TCP connection request. The most important system calls to implement this step are `socket() `, `bind`,`listen()`
2. SYN-SENT sent a request to establish a connection, waiting for a confirmation response
3. SYN-RECIEVED received a request to establish a connection and sent a request to establish a connection (confirm that the other party established a request to establish a connection). Waiting for the other party to confirm the connection request they sent
4. ESTABLISHED connection has been established for data transmission
5. FIN-WAIT-1, Waiting for the other end to confirm the request to close the connection that was just sent
6. FIN-WAIT-2 Received confirmation to close the connection request, waiting for the other end to send a request to close the connection
7. CLOSE-WAIT confirms the other end's close connection request, waiting for the local user to close the connection command
8. LAST-ACK The passively closed end receives the user's instruction to close the connection in the CLOSE-WAIT state, sends a close connection request, and waits for confirmation.
9. TIME-WAIT After the end that actively closes the connection receives the confirmation message that the other end sends to close the connection, it waits for a long enough time (2MSL) to ensure that the other end receives the ACK packet. Finally, it returns to the CLOSED state and releases the network resources.
10. CLOSING Under normal circumstances, after sending the FIN packet, it should receive the ACK packet of the other end first, and then receive the FIN packet of the other end. The CLOSING status indicates that the ACK packet of the other end has not been received after the FIN packet is sent, rather received the other end's FIN package. There are two situations that can lead to this state: First, if the two ends close the connection at the same time, then there may be cases where both ends send FIN packets at the same time; second, if the ACK packet is lost and the other party's FIN packet is sent out, FIN will arrive before ACK
11. CLOSED

```
                              +---------+ ---------\      active OPEN
                              |  CLOSED |            \    -----------
                              +---------+<---------\   \   create TCB
                                |     ^              \   \  snd SYN
                   passive OPEN |     |   CLOSE        \   \
                   ------------ |     | ----------       \   \
                    create TCB  |     | delete TCB         \   \
                                V     |                      \   \
                              +---------+            CLOSE    |    \
                              |  LISTEN |          ---------- |     |
                              +---------+          delete TCB |     |
                   rcv SYN      |     |     SEND              |     |
                  -----------   |     |    -------            |     V
 +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
 |         |<-----------------           ------------------>|         |
 |   SYN   |                    rcv SYN                     |   SYN   |
 |   RCVD  |<-----------------------------------------------|   SENT  |
 |         |                    snd ACK                     |         |
 |         |------------------           -------------------|         |
 +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
   |           --------------   |     |   -----------
   |                  x         |     |     snd ACK
   |                            V     V
   |  CLOSE                   +---------+
   | -------                  |  ESTAB  |
   | snd FIN                  +---------+
   |                   CLOSE    |     |    rcv FIN
   V                  -------   |     |    -------
 +---------+          snd FIN  /       \   snd ACK          +---------+
 |  FIN    |<-----------------           ------------------>|  CLOSE  |
 | WAIT-1  |------------------                              |   WAIT  |
 +---------+          rcv FIN  \                            +---------+
   | rcv ACK of FIN   -------   |                            CLOSE  |
   | --------------   snd ACK   |                           ------- |
   V        x                   V                           snd FIN V
 +---------+                  +---------+                   +---------+
 |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
 +---------+                  +---------+                   +---------+
   |                rcv ACK of FIN |                 rcv ACK of FIN |
   |  rcv FIN       -------------- |    Timeout=2MSL -------------- |
   |  -------              x       V    ------------        x       V
    \ snd ACK                 +---------+delete TCB         +---------+
     ------------------------>|TIME WAIT|------------------>| CLOSED  |
                              +---------+                   +---------+

                      TCP Connection State Diagram
```

### Why 3-way handshake when establishing connection

```
      TCP A                                                TCP B

  1.  CLOSED                                               LISTEN

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

  3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

  4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

  5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED

          Basic 3-Way Handshake for Connection Synchronization
```

To prevent invalid connection segment not being recognized by the server end.

Here is an example when "invalid connection segment" occurs: The first connection segment sent by the client is stuck for a long time(but not lost). The client resends a request to establish the connection and successfully communicates, and then the connection is released. After that, the initial segment arrives. The server mistakenly believe that the client is sending a new connection request. The server would response and wait for receiving data. But the client is not responding.

If 3-way handshake is being used, the server knows that the client is not asking for a new connection because the server doesn't get the confirmation from the client.

### Why 4-way handshake when closing/releasing connection

Closing a connection means: "I don't have data to send", but the closed connection can still receive data until the other end closes the connection.

Since TCP connection is full duplex, each direction must be closed separately

Three situations when closing a connection:

1. The FIN message is sent to the other end and the status changes to FIN_WAIT_1. Now the data can still be received. And the client waits for the other end to confirm the FIN message and waits for the other end to send FIN
2. The other party sends a FIN message to receive the FIN message sent by the other party, sends an ACK confirming the FIN, comes to the CLOSE_WAIT state; notifies the local user, if the local user sends a CLOSE command, sends a FIN message, enters the LAST_ACK state, and waits for the FIN message to be acknowledged. If the ACK message is received to delete the connection, if not received, the user will be notified;
3. Two ends close at the same time

Four times communications for close the connection because the socket in the LISTEN state of the server receives the connection request of the SYN message, it can send the ACK and SYN (the ACK responds and the SYN synchronizes) in one message. However, when the connection is closed, when receiving the FIN message from the other party, it only means that the other party has no data to send to you; but not all of your data has been sent to the other party, so you may not immediately close the socket. That is, you may need to send some data to the other party, then send a FIN message to the other party to indicate that you agree to close the connection now, so the ACK message and FIN message here are sent separately in most cases.

#### Reference 

https://tools.ietf.org/pdf/rfc793.pdf