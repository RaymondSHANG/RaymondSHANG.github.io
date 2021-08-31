---
layout: post
title: "ssh keygen ssh add"
subtitle: "ssh kegen"
date: 2021-08-30 17:30:04
header-style: text
catalog: true
author: "Yuan"
tags: [system]
---

>ssh-keygen

ssh-keygen pairs:

1. Generate key pairs:

The algorithm is selected using the -t option and key size using the -b option. The following commands illustrate:
ssh-keygen -t rsa -b 4096 ssh-keygen -t dsa ssh-keygen -t ecdsa -b 521 ssh-keygen -t ed25519
ssh-keygen -f ~/tatu-key-ecdsa -t ecdsa -b 521
</br>
2. Copy the pubkey to the server

ssh-copy-id -i ~/.ssh/tatu-key-ecdsa user@host
Once the public key has been configured on the server, the server will allow any connecting user that has the private key to log in.
</br>
3. Add private key to ssh agent

Firstly, run ssh-agent:</br>
    eval `ssh-agent`
Then, add private key:(You may need to change privatekey into a 400 access code)</br>
    ssh-add ~/.ssh/privatekey
Finally, Test:</br>
    ssh -vT git@github.com
</br>
---
