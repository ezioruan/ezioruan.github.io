---
title: how to acquire ssl certificate for oracle cloud loadbalcer
date: 2022-07-27 8:18:15
tags:
---


for example we will acquire ssl certificate for ezioruan.com

# install 

```
sudo apt update && sudo apt install certbot
```



# get ssl certificate by dns change

```
sudo certbot -d ezioruan.com --manual --preferred-challenges dns certonly
```


it will ask you to create a `txt` record in your dns provider
![image](https://user-images.githubusercontent.com/631411/181157340-1af15699-6163-4c60-9613-24b08927cccb.png)



after add the record press enter and you will get two files `fullchain.pem` and  `privkey.pem`

![image](https://user-images.githubusercontent.com/631411/181256803-67bb54a8-9bd1-4c0a-bfbf-bde05fb30bd4.png)


# import to loadbalcer

parse fullchain.pem in  SSL Certificate and privkey.pem to Private Key

and you'll able to use in loadbalcer's Certificates manager


