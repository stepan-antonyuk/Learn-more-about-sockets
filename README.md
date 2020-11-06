# Learn-more-about-sockets

resource: https://realpython.com/python-sockets/

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



#Echo Client and Server

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

One thing that’s imperative to understand is that we now have a new socket object from accept(). This is important since it’s the socket that you’ll use to communicate with the client. It’s distinct from the listening socket that the server is using to accept new connections:

    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data)

After getting the client socket object conn from accept(), an infinite while loop is used to loop over blocking calls to conn.recv(). This reads whatever data the client sends and echoes it back using conn.sendall().

If conn.recv() returns an empty bytes object, b'', then the client closed the connection and the loop is terminated. The with statement is used with conn to automatically close the socket at the end of the block.

#Echo Client

Now let’s look at the client, echo-client.py:

    #!/usr/bin/env python3

    import socket

    HOST = '127.0.0.1'  # The server's hostname or IP address
    PORT = 65432        # The port used by the server

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        s.sendall(b'Hello, world')
        data = s.recv(1024)

    print('Received', repr(data))

In comparison to the server, the client is pretty simple. It creates a socket object, connects to the server and calls s.sendall() to send its message. Lastly, it calls s.recv() to read the server’s reply and then prints it. 

Open a terminal or command prompt, navigate to the directory that contains your scripts, and run the server:

    $ ./echo-server.py

Your terminal will appear to hang. That’s because the server is blocked (suspended) in a call:

    conn, addr = s.accept()

It’s waiting for a client connection. Now open another terminal window or command prompt and run the client:

    $ ./echo-client.py 
    Received b'Hello, world'

In the server window, you should see:

    $ ./echo-server.py 
    Connected by ('127.0.0.1', 64623)

In the output above, the server printed the addr tuple returned from s.accept(). This is the client’s IP address and TCP port number. The port number, 64623, will most likely be different when you run it on your machine.


Viewing Socket State
