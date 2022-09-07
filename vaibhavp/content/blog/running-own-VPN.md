---
title: "Running Own VPN"
date: 2022-09-06T19:54:08+02:00
draft: false
---

# Introduction

Of course, we all do know what is VPN and why we need one. Here I am not going to explain to you guys what is VPN and how it works and all. Here I am going to talk about how you can set up your own VPN server in any geographic location. 

Recently I moved to a new country. But after arriving here I realised that now I won't be able to watch all my digital content like web series, movies and some mobile applications which work in my home country only.  To fix this problem I explore a lot of VPN service providers but I could not get one which can fix my problem and also which is cost-effective. So I decided to go with setting up my own VPN server with an open-source solution. 

After googling a lot I found Outline VPN Manager, which allows you to set up your own VPN server in your choice of the cloud provider at your choice geological location. So now let me show you how you can also set up your VPN server using the outline  VPN manager. 

# Outline VPN Manager

Outline is an open-source VPN manager which is developed and maintained by Jigwas.  Outline allows the user to create their personal VPN server in their choice of location and it is also resistant to blocking. The major issue with using OpenVPN is that after some time it is blocked by the specific country.  Outline server works with Shadowsocks which is a secure split proxy loosely based on **[SOCKS5](https://tools.ietf.org/html/rfc1928).** If you more details about it then you can read more about it [here](https://shadowsocks.org/guide/what-is-shadowsocks.html). So it's more how to guide for setting up your VPN server we will not go into more details of outline work in the background. So let's move on to how you will install Outline Server and client to use it on your devices. 

# Installation of Outline VPN Server and Client

Before installing your VPN server you will need infrastructure to host it and then you can connect to your server from anywhere in the world. so let's first create an account in any cloud provider. For this, I will use DigitalOcean because it gives me some free credit to be used for some time and after or above the free credit I will be charged.  If you already had an account on it then you will be charged for your server.  But if you don't have then it’s quite easy to open an account on it. [Here](https://docs.digitalocean.com/products/getting-started/) you will find how to open an account, please remember it will require your Credit Card details for billing purposes.   

So now you have your account set up let's move to install the outline VPN server and client.

Basically, Outline has 2 parts one is Outline Manager which will manage your VPN server and another part is Outline Client which will connect to your VPN server and set up the secure tunnel. 

So now let's start with installing the outline manager first.  

### Installing Outline Manager.

1. You can download the outline manager from [here](https://getoutline.org/en-GB/get-started/#step-1) for your system.
2. Now open that downloaded installer it will launch an installation wizard.
3. Follow the instruction and you will be able to install it successfully. 

{{< video src="/img/Outline_Manager.webm" type="video/webm" preload="auto" >}}

## Installing Outline Client

1. You can download the outline client from the same [link](https://getoutline.org/en-GB/get-started/) as above for your system.
2. Now open that downloaded installer it will launch an installation wizard.
3. Follow the instruction and you will be able to install it successfully. 

{{< video src="/img/Outline_Client.webm" type="video/webm" preload="auto" >}}

## Configuring Outline Manager and Client with DigitalOcean

Once you have installed Outline manager successfully, it's time to configure it to create your VPN server in your Digitalocean account. 

1. Open your Outline manager application which you just install on your system. 
2. Once you open your installed outline manager, it will show you different options of cloud providers to configure your VPN server on. Select the DigitalOcean option to login, it will redirect you to the login page of DigitalOcean, use your credentials for login. 
3. Once login is successfully done, click on create server button and follow the on-screen instruction for your VPN server setup. Wait for some time, Outline manager will automatically create the VPN server for you and it will appear on Outline manager. 

Now you have successfully set up your own VPN server but to establish the connection with your VPN server now you will have to configure your Outline Client as well. Let us configure our Outline client now. 

1. Once your VPN server is set up by the Outline manager it will be shown on the Outline manager dashboard.  
2. On the VPN server dashboard, you can click on Add new key or you will see a pre-existing key named `My access key` there you can click on the laptop 💻 symbol button. A pop-up screen will appear, now click on `Connect This Device` button.
3. It will open a link which you can copy or click on the copy button. 
4. Now it will ask you to install the Outline Client if you have not done it previously then you can do it now or you can also click on `ADD Server` button which will add the server key to your Outline client on the local system. 

{{< video src="/img/OutlineManager-1.mp4" type="video/mp4" preload="auto" >}}


## Reference

For your reference here are some links. 

 

- [https://www.digitalocean.com/solutions/vpn](https://www.digitalocean.com/solutions/vpn)
- [https://getoutline.org/](https://getoutline.org/)
- [https://github.com/Jigsaw-Code/outline-server](https://github.com/Jigsaw-Code/outline-server)
- [https://gfw.report/talks/imc20/en/](https://gfw.report/talks/imc20/en/)
- [https://shadowsocks.org/guide/what-is-shadowsocks.html](https://shadowsocks.org/guide/what-is-shadowsocks.html)