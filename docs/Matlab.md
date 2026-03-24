
Install this Java Runtime before installing Matlab

Java Runtime Required

Native Apple silicon MATLAB requires a Java runtime be installed on your Mac. A supported Java 11 JRE is available free of charge with Amazon Corretto 11

https://ww2.mathworks.cn/en/support/requirements/apple-silicon.html


Had to install Xcode from App store


## Running MATLAB as a docker instance


Install matlab using this :

`docker run --init -it --name ppdt_matlab_container -p 5901:5901 -p 6080:6080 -v ~/matlab_projects:/home/matlab/projects --shm-size=512M mathworks/matlab:r2025a -vnc`

`ssh -L 6081:localhost:6080 pgt10.22`

- Replace pgt10.22 with the ssh configuration used for accessing the node 
- Replace port 6081 with a different port number if the port is already used

Run matlab without license and in detached mode, set to keep running unless it is stopped :

`docker run -d --name ppdt_matlab_container --restart unless-stopped -p 6080:6080 -p 5901:5901 -v /home/pgt/matlab_projects:/home/matlab/projects mathworks/matlab:r2025a -vnc`

Verify it is running:

`docker ps`

SSH port forwarding:

`ssh -L 6081:localhost:6080 pgt10.22`

Open the MATLAB desktop in your browser:

`http://localhost:6081`

Was still unable to access the MATLAB desktop noVNC viewer

## Troubleshooting steps:

Check if port 6080 is open on the node

`ss -tulpn | grep 6080`

Expected result:

`LISTEN 0 4096 0.0.0.0:6080`

If nothing appears, then the container is not exposing the port correctly.

3️⃣ Test access from the node itself

On pgt10.22 run:

`curl http://localhost:6080`

Expected response:

`<html> noVNC ...`

OR open with text browser:

`wget -qO- http://localhost:6080 | head`

If this fails, the MATLAB container did not start the VNC service.