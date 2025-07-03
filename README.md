# bsideskrs2025-badge-writeup
Write up for the badge hacking challenge at BSides Kristiansand. All the challenges are basic and easy for beginners to get into.

![{BD89495E-61A3-43A7-8EAC-F27B1AA6205E}](https://github.com/user-attachments/assets/40dfa9dd-3098-43f6-b12c-54a9f4fd18ee)

## Inital Challenge - Connecting to the Badge
The badge becons out a network. The password you can find by either: 
- Decipher the morse code the badge is blinking
- Find it in the serial console during boot 

There are probably other ways too, e.g. reverse engineering it, or debugging it out of the device. 

Here are the steps to connect to the badge from Windows: 

- Find the device in device manager (devmgmt.msc):
![{3D9EA862-D1CF-49DD-A841-0725B2EC60F8}](https://github.com/user-attachments/assets/cff80aba-20c5-42d6-8bd3-2813f21a2e98)

Considering I just connected it, I find it as COM7. I use Putty to connect to COM7 over the baud rate of 115200:

![{9307DB32-8C52-4BC6-B1BA-F8EC3E4F4A6C}](https://github.com/user-attachments/assets/c369e3a2-cc6c-4b30-9569-53052bf5d039)

The console prints out useful information during the boot, including the wifi password. 

![{72934B70-08EC-455D-ABF0-AD981ED3EFFD}](https://github.com/user-attachments/assets/494223c3-96f1-4c1f-822d-2d9fe8137577)


## Easy Flag
This one here is on the front-page of the webserver... simple as that. If you don't know where to connect to, you would want to check DHCP and see which network you are in. Then port scan the default gateway (the badge) and find that it is listening with a web-server on port 80. 

![{A3CA03AA-6E3B-424E-8489-0B7E3CFF0FEA}](https://github.com/user-attachments/assets/e0b1994d-6339-429c-b962-877cc7e5384f)

This reveals the potential format for other flags too, namely: `DUCK_`. 

## Firmware Challenge
On the badge there is a link to the repo: https://github.com/So11Deo6loria/BSidesKristiansand2025Badge 

```git clone```
![{2D24C715-467C-4CC9-B19A-FDD97B0D3E65}](https://github.com/user-attachments/assets/088fe9a6-892e-406d-b2a7-8f43b497755e)

Considering we know what the needle looks like, we can potentially bruteforce the haystack: 

![{4762B43A-68C0-4A01-989D-8969BAFAD2C0}](https://github.com/user-attachments/assets/ba4d9334-5d27-4a2d-881d-9c38be8a2ff6)

## Comms Flag
Challenge presents this: 
![{5287A35F-F64B-4778-82C8-BA23B77853B8}](https://github.com/user-attachments/assets/e6e81939-df8d-43d9-9cff-e95e971bc5ea)

From firmware on Github we see this in the code: 

![{AE3BF1A1-8E11-45D6-A799-F1F0DB63C004}](https://github.com/user-attachments/assets/f7a3a09f-8a95-4be3-a61f-4b79fce7ca6e)

Take a look at the transmit_comms function: 

![{0B937BD9-994F-4FA8-9AB9-9CC5901BA5D7}](https://github.com/user-attachments/assets/2ee18cbd-5337-49b9-9377-84db45e169a6)

It is transmitted over UART. Connect with UART. I am using a buspirate: 

![{A5BB612A-C787-40C5-A2B6-51D65BEE7F60}](https://github.com/user-attachments/assets/08afb19f-ee54-48f4-a611-dcb3d88ffac8)

## Credits flag 
Simply in the source-code of the credits page. 

![{A7891264-615F-4060-BA63-C08135336586}](https://github.com/user-attachments/assets/699dee05-64b6-4056-a078-4831d694d5a3)

## Authorized Flag 
Quick recon on the firmware revealed `/admin` route in the web-application. Opening /admin in browser reveals it is asking for a username and password. 
![{4D5BBA99-4458-4C4C-B9B3-6AD62D97CC33}](https://github.com/user-attachments/assets/e3569263-2404-4820-bf38-9a94ed545ead)

Correlate down to the source-code we see the username is static and password is the device_id input template parameter: 

![{6C4676C3-7691-47F4-9F21-BD81EB826806}](https://github.com/user-attachments/assets/500ab4ed-c3fa-43ed-ae7a-8de81c30dfed)

Applying both username and password reveals flag and admin console: 

![{64A345BA-1466-48FF-B752-A28AFE51E1E4}](https://github.com/user-attachments/assets/3644ebc8-daf8-4edf-9893-c2773e0fc8da)

## Respond Flag
The badge is trying to connect to an access point called DUCK. I was thinking this one was going to get hard, so I used `hostapd` to set up a fake access point, listening to the SSID the challenge is beaconing for. I expected I had to do some MITM tricks to somehow win a flag. However once my fake access point was up and runing, the challenge simply beaconed for another SSID, then finally a third SSID. If it could connect to all three SSID's within a given time period, it would produce the flag. 

This one you can easily solve by setting up a open hotspot on your phone, however I already had my hostapd configuration up and running (because of previous engagements), so it was not a big deal. 

## Secured Flag 
The flags have to be stored on device somewhere. The clue sais: ```If you get the **Secured** flag you get every flag at once!```. Another clue is hinting about `rshell`. The clues are conclusive, we should connect to the board over serial connection (over USB), using rshell, a utility to connect to MicroPython enabled devices. 

First install, then connect: 
```
pip install rshell
rshell -p COM7
```

Once connected, navigate to the /pyboard mount and explore: 

![{E8E160BA-6DD5-4837-8AB1-F4AAE402A90C}](https://github.com/user-attachments/assets/a42a46b0-604c-46a7-9414-d2582fb291d4)

encrypted_db.json immediatley caught the attention. Flags are encrypted. Let us inspect source-code to figure out cryptography details: 

![{A60CE619-2F89-4846-87D8-F92338C73FCF}](https://github.com/user-attachments/assets/058b7bf2-fb4d-4858-a4d0-a703ecebdb0e)

It uses AES in ECB (mode 1). Cyberchef next: 

![{C77D6DA8-DF76-43BE-B412-BCD6E9C5E43D}](https://github.com/user-attachments/assets/b811cef0-4d30-440c-9d43-b4e1592eb549)

## All flags captured
Thanks for fun challenge.

![{945D37C0-C647-46CE-AF68-8DB88C6103AC}](https://github.com/user-attachments/assets/fa26b25d-e2e7-4c66-baf3-aa89510dd5ed)

https://youtube.com/shorts/zKIwzFcXHzs

Questions? Reach out on LinkedIn or somewhere else: http://linkedin.com/in/chrisdale 




