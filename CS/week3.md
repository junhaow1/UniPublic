## Lecture 1 (Transport Layer-Services, Multiplexing & Demultiplexing, UDP)

### How does information go through end-to-end layers

- **Application Layer**

  - the message is sent as a whole, only one header will be added.

- Transport-layer protocol provides for <u>logical communication</u> between **application process** running on different hosts

- **Transport Layer** (provide logical communication between <u>process</u> on different hosts, but actually you need to go through the socket, so it is a <u>process-to-process/socket-to-socket</u> delivery)

  - the message is split into chunks, each chunks called a **segment**.
  - <u>Segment</u>
    1. Each segment will be added a header contains information about transport layer

- **Network Layer** (provide logical communication between <u>hosts</u>, therefore, it is a host-to-host delivery)

  - This will add another header to the segment, and now it called datagram

- Compare transport layer and network layer

  ![image-20190319211133616](assets/image-20190319211133616.png)

### Services

- **Network layer** provides IP(Internet Protocol)
  - IP provide <u>logical communication</u> between hosts.
  - However, IP is an **unreliable service**
    - Does not guarantee segment delivery
      - It may lost half way
    - Does not preserve the order of the segment
    - Does not guarantee the integrity of the data in segment
      - The content might be altered
- In order to extend process-to-process to host to host, we need **transport layer** (notice that each socket has a unique identifier and each segment has a set of fields for identifying socket in their header)
  - Multiplexing (analogous to  Bill's job)
    - At the <u>receiving end</u>, the <u>transport layer</u> examines the fields in the segment to identify the receiving socket and then directs the segment to the socket.
  - Demultiplexing (analogous to Ann's job)
    - Gather data chunks at the source host from different sockets
    - Encapsulate each data chunk with header to create segments
    - Pass the segments to the network layer

### Transport Layer Service

- UDP
  - It almost add nothing to the original data
  - Multiplexing and demultiplexing
    - In UDP, sockets are identified by port numbers.
  - Some light error checking
  - ![image-20190319215655007](assets/image-20190319215655007.png)
    - Source port and dest. Port (from … to ...)
    - Length is the total size of the segment, including the header
    - Use checksum to check whether the segment has changed
  - UDP is <u>connectionless</u> and <u>unreliable</u>
    - Inheriting from IP: No guarantees on <u>delivery</u>, <u>order</u>, and <u>integrity</u>
  - Why we still need UDP?
    - No congestion-control, data is sent immediately
    - Real-time application often:
      - Requires a minimum sending rate,
      - Do not want to overly delay segment transmission,
      - Can tolerate some data loss
    - No connection establishment and no connection state.
      - Low system overhead (no buffer, no parameters)
    - Small packet header overhead
      - UDP using 8, while TCP using 20
  - UDP checksum
    - Used to determine whether bits within the UDP segment have been altered.
    - Procedures
      - Divide the message into 16 bit words
      - Add them up
        - If there is a overflow bit, warp the overflow bit to the lowest bit of the sum
      - And then send the complement of the code we have
      - When receiving the code, we should simply those 16-bit number with the checksum, if all bits turns to 1, that's good.
    - We still cannot guarantee that there is no error.
      - 110111011010111<u>0</u> and 110111011010111<u>1</u>
      - Becomes 110111011010111<u>1</u> and 110111011010111<u>0</u>
      - The result are all 1, but the message actually changed.



## Lecture 2

### Principles of Reliable Data Transfer (RDT)

- Problem encountered
  - IP is provided <u>network layer</u>, but it is a unreliable service
  - We want a <u>reliable data transfer</u> building on transport layer which is on the top of the <u>unreliable IP</u>

### RDT 1.0

- Suppose there is a <u>perfect reliable channel</u> underneath
- What only work left for RDT is send/receive and make/extract packet

### RDT 2.0 (stop and wait protocols) 

- Bits in a packet may be corrupted
- The notetaker will send a feedback to the send when receives a message (positive acknowledgements, negative acknowledgements)
- This retransmission are know as <u>ARQ (Automatic Repeat reQuest)</u>
  - Error detection
    - Allow receiver to detect bit error (checksum)
  - Receiver feedback
    - ACK and NAK
    - One bit for each, 1 = ACK, 0 = NAK
  - Retransmission
    - The sender retransmit a packet when it is received in errors
    - When the sender is in the <u>wait-for-ACK-or-NAK state</u>, it cannot get more data from the upper layer
- Bit error
  - Fatal flaw about RDT 2.0
    -  the ACK or NAK packet could be corrupted
  - How to fix this?
    - checksum
  - How protocol should handle the errors
    - Retrainsmit the garbled ACK/NAK packet
      - However, the retransmission can also corrupted, and  the system may failed after that.
    - Use enough checksum so that the receiver can also recover from bit errors.
      - But it couldn't handle if the packets were lost
    - Resends the current data packet when it receives a garbled ACK or NAK packet
      - Add a <u>sequence number</u> so that the receiver knows whether the message is a old message or new one
        - In stop-and-wait protocol, a 1-bit sequence number will suffice. The sender keep alternates the 1-bit sequence number between 0 and 1.
        - 
