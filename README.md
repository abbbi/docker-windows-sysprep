# Sysprepped /snapshot images using dockur/windows

```
 cd /tmp/
 git clone https://github.com/abbbi/docker-windows-sysprep
 cd /tmp/docker-windows-sysprep
 docker compose up
```

-> This will install windows server 2025 and as last step finalize the image
using sysprep. The docker Container will shut down after this. (note that
the Username/Password for the compose action are overwritten by oem/2025/unattend.xml)

Now you can use the sysprepped image "data.qcow2" as base image for further
startups, like so:

```
 rsync -a --progress --exclude=data.qcow2 /tmp/docker-windows-sysprep/* /tmp/clone-windows
 cd /tmp/clone-windows
 ```

create a snapshot image with the same size as base image using unsafe option:

```
 qemu-img create -b /base-image/data.qcow2 data.qcow2 -u -F qcow2 -f qcow2 68719476736B
```

now change the file in /tmp/clone-windows/compose.yaml to share the following volumes
and set another unique container name:

    container_name: windows_clone
    volumes:
      - /tmp/docker-windows-sysprep/:/base-image/
      - /tmp/clone-windows:/storage 

And again

```
 docker compose up
```

This will result in the base image located in /tmp/socker-windows-sysprep to be
used as backing image and the snapshot in /tmp/clone-windows to be used for
write operations.

This way you can spin up multiple instances using the same base image, like
for example vagrant does.
