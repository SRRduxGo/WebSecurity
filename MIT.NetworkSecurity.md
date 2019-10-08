# [Network Security ➜](https://youtu.be/BZTWXl9QNK8?list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh)

> [Lecture Notes](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-858-computer-systems-security-fall-2014/lecture-notes/MIT6_858F14_lec12.pdf)

## Threat Model

- Intercept packets and modify them
- injecting packets
  - \_\_
- attacker has computer
- _attacker participates in distributed protocol_

### BUG in the Protocol

- _TCP Seq number_
  - _How Handshake happens in TCP_
    - _`Client C, Server S`_
    - _`C: C ➜ S`, client sends a packet, part of that packet is `[SYN flg][SYNC Number(client) - #cn]`_
    - _`S: S ➜ C`, `[SYN flg][SYNC Number(Server) - #n][Ack - SYNC Number(client) - #cn]` server also acknowledges clients Synchronization number in its response_
    - _`C: C ➜ S`, `[Ack - SYNC Number(Server) - #n]`_
    - _`C: C ➜ S`, `[DATA][SYNC Number(client) - #cn + 1]` start of the data transfer_
    - _next data packet to the server will have `#cn + 1` as sequence number (overly simplified)_
    - _It also has security implication that the `sequence of events - handshaking` in a way guarantees that right parties are speaking to each other when data exchange starts happening_
    - _SEQ No are 32 bit long in TCP_
    - _Initial SEQ No, over connections, is monotonically increasing so no two connections will collide in terms of sequence numbers_

#### Potential Attack

- _`Attacker A(ip-addr), Client C(ip-addr), Server S(ip-addr)`_
- _`A: C ➜ S` `[SYN flg][SYNC Number(client) - #cn]`_
- _`S: S ➜ C` `[SYN flg][SYNC Number(Server) - #n] Ack[#cn]` | this communication packet is going to `C` not to `A`, so attacker has no way to know `#n`_
  - _`C` if happens to be online might send a reset packet and the whole plan fails_
- _`A: C ➜ S` `Ack[#n]` - Attacker guesses the `#n` as per the protocol RFC_
- _`C: C ➜ S`, `[DATA][ #cn + 1]`_

#### Problems arising due to above loop hole

- _IP based Authorization - Attacker can forge the IP address in the packet_
- _Connection RESET Attack_
  - _Target: BGP router protocol_
  - _fix was to utilise `TTL value : 255`- Router rejected any packet which has less than `255` value (oO! requires little R&D)_
- _Data Injection in some existing connections_

### TCP SYN implementations to mitigate SEQUENCE number problems

- `Compute Initial Seq No# (old style) + ( F( Src Ip,Src Port,Dest Ip, Dest Port, SecretKey) ➜ [Random 32 bit offset] )`
- This gives a unique `Initial Sequence No` - `F` is a cryptographically secure hash function, cryptographically hard to invert it

## DNS protocol - is hugely vulnerable and Runs on UDP

- _udp stateless protocol_
- _Single round trip - no time to establish that you are talking to the right party_
- `C ➜ S (example.com)(DNS server listens on port 53)`, `S ➜ C (server sends the ip address - C is also listening at port 53)`
- An attacker can send the required data response, the IP Address of the site which client queried for, catch is Client should receive the data packet from the attacker before client receives the actual response from the DNS.
- Client Machine would start using the data which was received first (here from the attacker - being optimistic oO)! )
- **_To fix the issue how to bring the randomness, so it is difficult for the attacker to guess_**
  - **One Solution**
    - _Port Number - choosing a random 16 bit port number from the client side(source)_
    - _Query ID - Inside of the DNS request, which is also echoed back by the server_
  - **DNS-SEC protocol, NSEC-3**

> ### **_Denial of Service Attack_**
>
> **_During TCP handshake - server has to save / persist the client Sync-Sequence-Number
> Why server persists this value is because to know what Sequence-Number to accept later(when Data starts to coming in - *`C: C ➜ S`, `[DATA][ #cn + 1]`* )_**  
> **_And this data is stored in a table and is long-lived due to various reasons. An attacker can send millions of initial SYNC packets to JAM up the Server side tables. This is called `SYN` flooding in TCP_**

<br/>

> This problem arises because Server starts keeping state even before authenticating who the Client is.  
> Challenge is How to design a Backward compatible Solution:  
> Trick is to somehow make the server state-less.  
> That is choosing Server-Side `Sync-Sequence-Number` `SNS` during the handshake if server thinks that it is under an attack  
> `formula: TimeStamp(8 bits) + F(SrcIp, SrcPort, DistIp, DistPort, TimeStamp, Secret-Key)(24 bits) = **SNS**`  
> what this translates to is this
>
> - Don't store anything at all till the third step of TCP three way handshake
> - At the third step when the client sends `ACK[#n - SNS] + SNC` packet then procure the `timestamp` from the `SNS` and verify the received `SNS` against the above `formula`, if `SNS` stands valid then store it and the `SNC` in the table

<br/>

> **_General Rule To design a Protocol or a Service_**
>
> - **_Let the client do same amount of work as the server_**
