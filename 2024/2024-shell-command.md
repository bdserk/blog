
### ping 检测 
```
ping -c 86400 -i 1 -s 1024  baidu.com  | awk '{ print $0"\t" strftime("%D_%H:%M:%S",systime()) ;fflush() }' > 1.log

```
