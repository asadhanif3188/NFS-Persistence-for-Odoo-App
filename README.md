# NFS Persistence for Odoo Application 
In this demo project we are going to deploy an Odoo application along with Postgres database, on Kubernetes. To persist the data of application and database we'll be using provisioning storage from NFS with the help of PVC. 

## Step 1: Setup NFS Server 
Network File System (NFS) is a distributed file system protocol that allows us to mount remote directories on machines over a network. This lets us manage storage space in a different location and write to that space from multiple clients. NFS provides a relatively standard and performant way to access remote systems over a network and works well in situations where the shared resources must be accessed regularly.

### Step 1 (A): Installing the NFS Server 
The NFS server package provides user-space support needed to run the NFS kernel server. To install the package, run following commands:

`sudo apt update`

`sudo apt install nfs-kernel-server`

Once the installation is completed, the NFS services will start automatically.

We can verify the versions of NFS by running the following cat command:

`sudo cat /proc/fs/nfsd/versions`

<img src="./screenshots/1-NFS-Server-versions.png" width="80%" />

### Step 1 (B): Make shared NFS directory
We will create a directory named **nfs_shared** that is going to be shared by all client systems. To do so, run the following command:

`sudo mkdir -p /mnt/nfs_shared`

### Step 1 (C): Set directory permissions
Set the permissions of the created **nfs_shared** directory so that all client machines can easily access it:

`sudo chown -R nobody:nogroup /mnt/nfs_shared/`

<img src="./screenshots/2-shared-directory-permissions.png" width="80%" />

### Step 1 (D): Set file permissions
Set the file permissions as required. In our case, we have allocated the read, write, and execute permissions to the **nfs_shared** directory files:

`sudo chmod 777 /mnt/nfs_shared/`

### Step 1 (E): Grant NFS access
In this step, we will grant access to the client system for accessing the NFS server. To do so, open **/etc/exports** in the **nano** editor:

`sudo nano /etc/exports`

Now, it is up to us whether we want to grant access to the entire **subnet**, **single** or **multiple** clients. For instance, we will permit can entire subnet **192.168.56.101/24** to access the NFS shared:

Following statement is used to grant access to a subnet. 

`/mnt/nfs_share 192.168.56.101/24(rw,sync,no_subtree_check)`

<img src="./screenshots/3-NFS-Access-grant.png" width="80%" />

### Step 1 (F): Exporting NFS directory
Following command is used to export the NFS shared directory:

`sudo exportfs -a`

### Step 1 (G): Restart NFS server
Now restart the NFS server using following command: 

`sudo systemctl restart nfs-kernel-server`

### Step 1 (H): RestartGrant Firewall access
It is time to grant the Firewall access to the client system with the following **ufw** command:

`sudo ufw allow from 192.168.56.101/24 to any port nfs`

### Step 1 (I): Enable Firewall
Enable Firewall with **ufw** command and **enable** option:

`sudo ufw enable`

Status of the `ufw` firewall can be seen in following screenshot. 

<img src="./screenshots/4-ufw-firewall.png" width="60%" />

## Step 2: NFS Client Configuration 
Now we need to configure our NFS client so that it can access the shared directory of NFS Server. 

In our case we want to access the directory on our **Minikube** environment. So for our case Minikube is the client. 

### Step 2 (A): SSH into Minikube
Minikube provides a nice command to SSH into it. To SSH into our Minikube instance use: `minikube ssh` command.

<img src="./screenshots/5-minikube-ssh.png" width="60%" />

### Step 2 (B): Verify Communication between Minikube and NFS Server
It is a good idea to verify the Minikube VM can communicate with the NFS server host. Use ping to verify communication by using the IP address of the Linux host that is hosting the NFS server, i.g. `192.168.56.102` . Limit the ping command to 2 with `-c 2` so it will stop pinging.

`ping 192.168.56.102 -c 2`

<img src="./screenshots/6-minikube-ping-nfs-server.png" width="60%" />

As we can see from the figure, our minikube can communicate sucessfully with the NFS server. 

### Step 2 (C): Mount in Minikube as the NFS client
Previous step has ensured that the client and server can communicate, lets set up the NFS client in Minikube.

Create a **mount** in Minikube as the NFS client. Kubernetes will handle mapping the directories and mounts for your containers and volumeMounts.

Create an NFS mount using following command:

`mount -t nfs 192.168.56.102:/mnt/nfs_shared /mnt`

The mount command needs the source that is the NFS server IP address, i.e. `192.168.56.102` and absolute path to the shared directory, i.e. `/mnt/nfs_shared`. And this will mount in `/mnt` in Minikube so the Pods will have access.

