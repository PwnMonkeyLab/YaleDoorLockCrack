# Description of the vulnerability

Yale provides an App named Yale Bluetooth Key to control their doorlock by cellphone. The Yale App communicates with Yale doorlock via BLE. In their communication, the doorlock uses a 8-byte key to authenticate user’s cellphone. Once we get this key, we can unlock this door on any cellphone.

By sniffing the BLE communication between cellphone and doorlock, the key can be easily calculated.

# Analysis of the vulnerability

When cellphone tries to connect to doorlock, there are 2 packets for authentication procedure:  
<br>
![Authentication Request](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/Authentication%20request.png)

![Authentication Response](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/Authentication%20response.png)

<br>
The structure of request and response packets are shown below:
<br>
<br>

![Structure of Request and Response Packets](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/Structure%20of%20packets.png)

<br>
Authentication request is sent from doorlock to cellphone, and after processing payload of request, cellphone generates a authentication response to doorlock.

By reversing analyzing the corresponding app, we can find an essential function named "encodeCounter", as shown below:
<br>
<br>

![Function encodeCounter Part 1](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/encodeCounter_Part1.png)

<br>
Parameter counterData is the payload of authentication request, keyString is the 8-byte authentication key we mentioned above.

keyString is split into 6 parts, stored in variable key1 ~ key6.
<br>
<br>

![Function encodeCounter Part 2](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/encodeCounter_Part2.png)

<br>
Each part of keyString is added to the corresponding counterData, and the result is saved in variables c1r1 ~ c3r4.
<br>

![Function encodeCounter Part 3](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/encodeCounter_Part3.png)

<br>
Finally, the function returns with HexString(c1r1 + c1r2 + c1r3 + c1r4 + c2r1 + c2r2 + c2r3 + c2r4 + c3r1 + c3r2 + c3r3 + c3r4). This is the payload of authentication response.
<br>

# Exploit of the vulnerability
As we analyzed in section 2. In order to calculate the authentication key, we only need to sniff the BLE communication, which is pretty easy. Here’s an example:
<br>
<br>

![Sniffing BLE Communication](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/BLE%20sniff.png)

<br>
The BLE communication is sniffed by a USB dongle, 5 dollars from ebay.

In this case, payload of request is "FB 2B 30 5B 06 31 2B 5C 11 42 31 73" and payload of response is "EA 83 2C 29 71 2D C5 2A 0D 9A 20 0D". Authentication key is calculated like this.
<br>
<br>

![Calculating Authentication Key](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/Auth%20key%20calculate.png)

<br>
So, key1 = 0xEF, key2 = 0x58, key3 = 0xFC, key4 = 0xCE, key5 = 0x6B, key6 = 0x9A.

And the authentication key in this case should be "EF58FCCE6B9A0000". We also append 2 bytes of 0x00 because authentication key should be 8 bytes but only 6 bytes are used.

Using this authentication key, we can unauthorizedly unlock this doorlock by any cellphone.
<br>
<br>

![Cracked Doorlock](https://github.com/PwnMonkeyLab/Pictures/blob/master/YaleBluttoothDoorlock/Doorlock.jpg)
