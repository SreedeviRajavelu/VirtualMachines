keypair was created using command like below on windows :
 
ssh-keygen-t ed25519 ...
 
had to convert the private key that was generated on windows using dos2unix as the line endings of the text file generated on windows will be based on carriage return 
can check using cat -v ~/.ssh/id_ed25519_private
If you see ^M at the end of lines → that’s a Windows CR (\r).
 
command to ssh to ncl node 10.13
this does not work (as need to create a config file ):
ssh -i ~/.ssh/id_ed25519_private -J ADSC_SmartGrid_5@gateway.ncl.sg -i ~/.ssh/id_ed25519_private pgt@10.10.10.13
 
Create config file first

```
# Gateway host
Host gateway
    HostName gateway.ncl.sg
    User ADSC_SmartGrid_5
    IdentityFile ~/.ssh/id_ed25519_private

# Intermediate host
Host intermediate
    HostName 172.18.178.17
    User pgt
    Port 4696
    IdentityFile ~/.ssh/id_ed25519_private
    ProxyJump gateway

# Final internal host
Host pgt10.13
    HostName 10.10.10.13
    User pgt
    IdentityFile ~/.ssh/id_ed25519_private
    ProxyJump intermediate
 
```
 

`chmod 600 ~/.ssh/config`
`ssh -v pgt10.13`
 
