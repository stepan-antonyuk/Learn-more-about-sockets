# Learn-more-about-sockets
The primary socket API functions and methods in this module are:

    socket()
    bind()
    listen()
    accept()
    connect()
    connect_ex()
    send()
    recv()
    close()
    
 Transmission Control Protocol (TCP)
 
 Starting in the top left-hand column, note the API calls the server makes to setup a “listening” socket:

    socket()
    bind()
    listen()
    accept()
    
A listening socket listens for connections from clients.
connect() to establish a connection to the server and initiate the three-way handshake. 
data is exchanged between the client and server using calls to send() and recv().


