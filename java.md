## Create memory dump for Java app

```
/usr/java/openjdk-13/bin/jmap -dump:file=/root/heap.data $(ps faux | grep java | grep -v grep |  awk '{print $2}')
```