---
title: gpg unusable key
date: 2020-02-08 8:18:15
tags: tools
---

# fix gpg unusable key 

when we import gpg keys from backup, and try to use it for decript/encript, it will tell that the key is not usable,
we need to trust the keys first

use `gpg --list-keys` to list all keys 

the output would like this 
```
pub   2048R/B660885G 2013-05-16 [expires: 2030-12-31]
uid       [] SAP Admin <xxx@xx.com>
```

we can get the keys id here `B660885G`

and then `gpg>trust`

If you're sure about the authenticity of your key, select the trust level 5.
