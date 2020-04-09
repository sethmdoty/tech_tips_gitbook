# IOCTL Recovery

```text
Ifconfig and find nextcloud interface
 Ifconfig bridge 0 destroy
Ifconfig bridge 1 addm vnet0:1 up
Exec 1 /bin/tcsh
Add to rc.conf to see if it fixes this:
Variable : cloned_interfaces ; 
Value : bridge0 Variable : ifconfig_bridge0 ; 
Value : addm igb0 up
```

