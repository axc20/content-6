# Network Protocols

Network Protocols are standard methods of transferring data between two computers in a network.

1. **HTTP (HyperText Transfer Protocol)**: Protocol for fetching resources such as HTML documents. It is the foundation of any data exchange on the Web and is a client-server protocol
2. **HTTP/3**: The next major revision of HTTP. It runs on QUIC, a new transport protocol designed for mobile-heavy internet usage. It relies on UDP instead of TCP, which enables faster web page responsiveness. VR applications demand more bandwidth to render intricate details of a virtual scene and will likely benefit from migrating to HTTP/3.
3. **HTTPS (HyperText Transfer Protocol Secure)**: Extends HTTP and uses encryption for secure communications.
4. **WebSocket**
5. **TCP (Transmission Control Protocol)**: TCP is designed to send packets across the internet and ensure the successful delivery of data and messages over networks. Many application-layer protocols build on top of TCP.
6. **UDP (User Datagram Protocol)**: Sends packets directly to a target computer, without establishing a connection first. UDP is commonly used in time-sensitive communications where occasionally dropping packets is better than waiting. Voice and video traffic are often sent using this protocol.
7. **SMTP (Simple Mail Transfer Protocol)**: Standard protocol to transfer electronic mail from one user to another.
8. **FTP (File Transfer Protocol)**: Used to transfer computer files between client and server. It has separate connections for the control channel and data channel.

## HTTPS

1. Server Certificate Check
   - Client and Server communicate, with server sending a Server Certificate
   - Client checks that the certificate is valid by checking with a Certificate Authority Server
2. Key Exchange
   - Client extracts server public key from certificate, and creates a session key
   - Client communicates to server "I know A, B, C, D cipher suites"; Server responds with "OK, let's use C cipher suite"
   - Client encrypts session key using server public key and C cipher suite; Server decrypts using server private key
3. Encrypted Tunnel for data transmission
   - At this point, Client and Seerver both have a common key (session key)
   - Client and Server both encrypt/decrypt data with session key
