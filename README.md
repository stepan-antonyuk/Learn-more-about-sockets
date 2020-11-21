# Learn-more-about-sockets

Why am i doing this: just to remember it

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
 
API calls the server makes to setup a “listening” socket:

    socket()
    bind()
    listen()
    accept()
    
A listening socket listens for connections from clients. 

connect() to establish a connection to the server and initiate the three-way handshake. 

Using calls to send() and recv() to exchanged between the client and server. 



#Echo Client and Server

Server echo whatr it receives back to the client.

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


socket.socket() creates a socket object that supports the context manager type.

There’s no need to call s.close():

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        pass  # Use the socket object without calling s.close().

TODO--------------->socket() specify the address family and socket type. AF_INET is the Internet address family for IPv4. SOCK_STREAM is the socket type for TCP, the protocol that will be used to transport our messages in the network.

bind() is used to associate the socket with a specific network interface and port number:

    HOST = '127.0.0.1'  # Standard loopback interface address (localhost)
    PORT = 65432        # Port to listen on (non-privileged ports are > 1023)

    # ...

    s.bind((HOST, PORT))

The values passed to bind() depend on the address family of the socket. In this example, we’re using socket.AF_INET (IPv4). So it expects a 2-tuple: (host, port).

host can be a hostname, IP address, or empty string. If an IP address is used, host should be an IPv4-formatted address string. The IP address 127.0.0.1 is the standard IPv4 address for the loopback interface, so only processes on the host will be able to connect to the server. If you pass an empty string, the server will accept connections on all available IPv4 interfaces.

port should be an integer from 1-65535 (0 is reserved). It’s the TCP port number to accept connections on from clients. Some systems may require superuser privileges if the port is < 1024.

The first time you run your application, it might be the address 10.1.2.3. The next time it’s a different address, 192.168.0.1. The third time, it could be 172.16.7.8, and so on.

Continuing with the server example, listen() enables a server to accept() connections. It makes it a “listening” socket:

    s.listen()
    conn, addr = s.accept()

listen() has a backlog parameter, it specifies the number of unaccepted connections that the system will allow before refusing new connections. Starting in Python 3.5, it’s optional. If not specified, a default backlog value is chosen.

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


To see the current state of sockets on your host, use netstat. It’s available by default on macOS, Linux, and Windows.

Here’s the netstat output from macOS after starting the server:

    $ netstat -an
    Active Internet connections (including servers)
    Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
    tcp4       0      0  127.0.0.1.65432        *.*                    LISTEN

Notice that Local Address is 127.0.0.1.65432. If echo-server.py had used HOST = '' instead of HOST = '127.0.0.1', netstat would show this:

    $ netstat -an
    Active Internet connections (including servers)
    Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
    tcp4       0      0  *.65432                *.*                    LISTEN

Local Address is *.65432, which means all available host interfaces that support the address family will be used to accept incoming connections. In this example, in the call to socket(), socket.AF_INET was used (IPv4). You can see this in the Proto column: tcp4.

I’ve trimmed the output above to show the echo server only. You’ll likely see much more output, depending on the system you’re running it on. The things to notice are the columns Proto, Local Address, and (state). In the last example above, netstat shows the echo server is using an IPv4 TCP socket (tcp4), on port 65432 on all interfaces (*.65432), and it’s in the listening state (LISTEN).

Another way to see this, along with additional helpful information, is to use lsof (list open files). It’s available by default on macOS and can be installed on Linux using your package manager, if it’s not already:

    $ lsof -i -n
    COMMAND     PID   USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
    Python    67982 nathan    3u  IPv4 0xecf272      0t0  TCP *:65432 (LISTEN)

lsof gives you the COMMAND, PID (process id), and USER (user id) of open Internet sockets when used with the -i option. Above is the echo server process.

netstat and lsof have a lot of options available and differ depending on the OS you’re running them on. Check the man page or documentation for both. They’re definitely worth spending a little time with and getting to know. You’ll be rewarded. On macOS and Linux, use man netstat and man lsof. For Windows, use netstat /?.

Here’s a common error you’ll see when a connection attempt is made to a port with no listening socket:

    $ ./echo-client.py 
    Traceback (most recent call last):
      File "./echo-client.py", line 9, in <module>
        s.connect((HOST, PORT))
    ConnectionRefusedError: [Errno 61] Connection refused

Either the specified port number is wrong or the server isn’t running. Or maybe there’s a firewall in the path that’s blocking the connection, which can be easy to forget about. You may also see the error Connection timed out. Get a firewall rule added that allows the client to connect to the TCP port!

Communication Breakdown

When using the loopback interface, data never leaves the host or touches the external network. The loopback interface is contained inside the host. This represents the internal nature of the loopback interface and that connections and data that transit it are local to the host. This is why you’ll also hear the loopback interface and IP address 127.0.0.1 or ::1 referred to as “localhost.”

Applications use the loopback interface to communicate with other processes running on the host and for security and isolation from the external network. Since it’s internal and accessible only from within the host, it’s not exposed.

If you have an application server that uses its own private database. If it’s not a database used by other servers, it’s probably configured to listen for connections on the loopback interface only. If this is the case, other hosts on the network can’t connect to it.

When you use an IP address other than 127.0.0.1 or ::1 in your applications, it’s probably bound to an Ethernet interface that’s connected to an external network. This is your gateway to other hosts outside of your “localhost” kingdom:

Be sure to read the section Using Hostnames before venturing from the safe confines of “localhost.” There’s a security note that applies even if you’re not using hostnames and using IP addresses only.

Handling Multiple Connections

The echo server definitely has its limitations. The biggest being that it serves only one client and then exits. The echo client has this limitation too, but there’s an additional problem. When the client makes the following call, it’s possible that s.recv() will return only one byte, b'H' from b'Hello, world':
