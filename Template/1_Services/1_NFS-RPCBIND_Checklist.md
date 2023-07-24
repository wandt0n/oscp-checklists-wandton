<% tp.file.rename("1_NFS-RPCBIND_")%>

## Show mount
```bash
sudo showmount -a $hip
```
(and $-e$ and $-d$)
	

## NSE script
```bash
nmap -sV -p 111 --script=rpcinfo $subnet
```
	

## Mount NFS share
```bash
sudo mount -o nolock $hip:/$pathtoshare ~/nfs1/ 
```
	

## NFS enumeration with NSE scripts
```bash
nmap -p 111 --script nfs* $hip -oN nmap_NFS.txt
```
	

## Change UUID to access the files
```bash
sudo sed -i -e 's/$oldUserID/$newUserID/g' /etc/passwd 
```
	


# Findings