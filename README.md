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
    
A listening socket listens for connections from clients. connect() to establish a connection to the server and initiate the three-way handshake. Data is exchanged between the client and server using calls to send() and recv(). 



Echo Client and Server

The server will simply echo whatever it receives back to the client.

Here’s the server, echo-server.py:

    #!/usr/bin/env python3

    import socket

    HOST = '127.0.0.1'  # Standard loopback interface address (localhost)
    PORT = 65432        # Port to listen on (non-privileged ports are > 1023)

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((HOST, PORT))
        s.listen()
        conn, addr = s.accept()
        with conn:
            print('Connected by', addr)
            while True:
                data = conn.recv(1024)
                if not data:
                    break
                conn.sendall(data)


socket.socket() creates a socket object that supports the context manager type. There’s no need to call s.close():

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        pass  # Use the socket object without calling s.close().

socket() specify the address family and socket type. AF_INET is the Internet address family for IPv4. SOCK_STREAM is the socket type for TCP, the protocol that will be used to transport our messages in the network.

bind() is used to associate the socket with a specific network interface and port number:

    HOST = '127.0.0.1'  # Standard loopback interface address (localhost)
    PORT = 65432        # Port to listen on (non-privileged ports are > 1023)

    # ...

    s.bind((HOST, PORT))

The values passed to bind() depend on the address family of the socket. In this example, we’re using socket.AF_INET (IPv4). So it expects a 2-tuple: (host, port).

host can be a hostname, IP address, or empty string. If an IP address is used, host should be an IPv4-formatted address string. The IP address 127.0.0.1 is the standard IPv4 address for the loopback interface, so only processes on the host will be able to connect to the server. If you pass an empty string, the server will accept connections on all available IPv4 interfaces.

port should be an integer from 1-65535 (0 is reserved). It’s the TCP port number to accept connections on from clients. Some systems may require superuser privileges if the port is < 1024.

It could be anything. The first time you run your application, it might be the address 10.1.2.3. The next time it’s a different address, 192.168.0.1. The third time, it could be 172.16.7.8, and so on.

Continuing with the server example, listen() enables a server to accept() connections. It makes it a “listening” socket:

    s.listen()
    conn, addr = s.accept()

listen() has a backlog parameter. It specifies the number of unaccepted connections that the system will allow before refusing new connections. Starting in Python 3.5, it’s optional. If not specified, a default backlog value is chosen.

If your server receives a lot of connection requests simultaneously, increasing the backlog value may help by setting the maximum length of the queue for pending connections. The maximum value is system dependent. For example, on Linux, see /proc/sys/net/core/somaxconn.

accept() blocks and waits for an incoming connection. When a client connects, it returns a new socket object representing the connection and a tuple holding the address of the client. The tuple will contain (host, port) for IPv4 connections or (host, port, flowinfo, scopeid) for IPv6. See Socket Address Families in the reference section for details on the tuple values.
