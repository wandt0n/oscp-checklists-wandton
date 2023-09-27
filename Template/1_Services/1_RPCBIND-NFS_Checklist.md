#service 
<% tp.file.rename("1_RPCBIND-NFS_111-2049")%>
☑️

#### rpcinfo
```bash
rpcinfo -s $hip
```
Look for nfs, ypbind and ruserd. Ignore nlockmgr, status, portmapper, and mountd

# NFS

#### NSE enum
```bash
nmap -p 445 --script nfs* $hip
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