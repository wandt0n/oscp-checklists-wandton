<% tp.file.rename("1_NFS-RPCBIND_")%>

> RPCBind (Port 111) is used to communicate between Unix systems. Can be used for enumeration.

#### rpcinfo
```bash
rpcinfo -s $hip
```

> Look for nfs, ypbind and ruserd

# NFS

#### NSE enum
```bash
nmap -p 111 --script nfs* $hip -oN nmap_NFS.txt
```
	
#### Show mount
```bash
sudo showmount -a $hip
```
(and `-e` and `-d`)
	
#### Mount NFS share
```bash
sudo mount -o nolock $hip:/$pathtoshare ~/nfs1/ 
```
	
#### Change UUID to access the files
```bash
sudo sed -i -e 's/$oldUserID/$newUserID/g' /etc/passwd 
```
	


# Findings