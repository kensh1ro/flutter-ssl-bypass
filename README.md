# flutter-ssl-bypass

### tested on linux and osx
## i will assume that you already have burp's certificate installed to system CA store, if not then follow along

ATTENTION: this needs a rooted device

after exporting Burp's DER certificate, we need to change it to PEM format, although this isn't necessary, but it seems that Android CA store doesn't like DER format

`openssl x509 -inform der -in $CERT_NAME -out certificate.pem`

`openssl x509 -inform PEM -subject_hash_old -in certificate.pem | head -1` #my output was 9a5ba575

rename "certificate.pem" to the above command's output, for example:
`mv certificate.pem 9a5ba575.0`

now make sure you have an ADB connection to your device

`adb push 9a5ba575.0 /sdcard`

`adb shell`

`su`

`mount -o rw,remount /system`

`cp /sdcard/9a5ba575.0 /system/etc/security/cacerts`

`chmod 644 /system/etc/security/cacerts/9a5ba575.0`

`chown root:root /system/etc/security/cacerts/9a5ba575.0`

after that you should REBOOT!

**i will demonstrate by example to make the method easier to apply**

You will need:
- Android device (rooted physical phone preferred)
- a PC with linux installed
- a brain ;)

lets assume that our android device has the IP address 192.168.1.7
and linux PC got 192.168.1.9

## 1) **First step is to add the linux PC as a gateway for our Android device**

![image](https://user-images.githubusercontent.com/46089361/115142889-ccde2000-a04c-11eb-859f-9d5501a83f5c.png)


## 2) **Second step is to forward http/https traffic to Burp Suite** 
this step here only differs for osx users

### for linux:
`iptables -t nat -A PREROUTING -p tcp -j REDIRECT --dport 443 --to-ports 8080`

`sysctl -w net.ipv4.ip_forward=1`

NOTE: you can also add a redirect for port 80 to 8080 by changing `--dport 443` to `--dport 80`
### for osx:
`sudo sysctl -w net.inet.ip.forwarding=1`

`echo 'rdr pass inet proto tcp from any to any port {80,443} -> 127.0.0.1 port 8080' > pfctl.txt`

`sudo pfctl -f pfctl.txt -e`



## 3) **Third step is to let Burp Suite capture the forwarded traffic**
make sure to enable invisible proxying

![image](https://user-images.githubusercontent.com/46089361/115143520-868ac000-a050-11eb-9fad-5e3829a1f7da.png)


also you might want to listen on all interfaces, to avoid ip configuration hastle


![image](https://user-images.githubusercontent.com/46089361/115143599-0ca70680-a051-11eb-85b8-1eda41c4fce3.png)

at the end it should look something like this (pay attention to the User Agent):

![image](https://user-images.githubusercontent.com/46089361/115143751-dae26f80-a051-11eb-9d25-a74a9219d451.png)

## NOTE: This is applicable on all non-proxy aware libraries (Xamarin, Okhttp3..)
