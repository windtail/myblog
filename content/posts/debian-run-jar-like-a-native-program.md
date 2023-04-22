---
title: "Debian run jar like a native program"
date: 2018-05-22T19:02:00+08:00
draft: false
---



```
sudo apt install binfmt-support jarwrapper
```


比如 swagger-codegen




```
wget -O ~/.local/bin/swagger-codegen https://oss.sonatype.org/content/repositories/releases/io/swagger/swagger-codegen-cli/2.2.1/swagger-codegen-cli-2.2.1.jar
chmod a+x ~/.local/bin/swagger-codegen

# run
swagger-codegen
```


 


