# Android Cheatsheet
## Useful commands

List the emulators available
```sh
android list avd
```

Launch the emulator through a proxy
```sh
emulator -avd nameofvirtualdevice -http-proxy localhost:8080 -debug-proxy
```

List connected devices/emulators
```sh
adb devices
```

Access to a device
```sh
adb shell
```

Get the mobile phone/ emulator IP
```sh
adb shell netcfg 
adb shell ifconfig
```

Access to a device (for multiple devices)
```sh
adb -s <device-name> shell
```

Install an application
```sh
adb install application.apk
```

Uninstall an application
```sh
adb uninstall com.example.appication
```

List applications installed
```sh
adb shell pm list packages
```

Get the path of an installed application
```sh
adb shell pm path <package-name>
```

Extract files from the device
```sh
adb pull <remote> <local>
```

Pull files into the device
```sh
adb push <local> <remote>
```

Extract the device’s log
```sh
adb logcat | grep "$(adb shell ps | grep <package-name> | awk '{print $2}')"
```

Additional logs
```sh
adb shell dumpstate > dumpstate.txt
adb shell dumpsys > dumpsys.txt
```

Forward ports
```sh
adb forward <tcp/udp>:<host_port> <tcp/udp>:<device_port>
```

Get the list of packages on a device
```sh
adb shell pm list packages
```

Get the full path name of the APK file 
```sh
adb shell pm path com.example.application
```

Unzip the APK file as a normal ZIP file
```sh
unzip –q <APK_name>.apk
```

Convert classes.dex into a JAR file
```sh
d2j-dex2jar classes.dex
or
jadx --export-gradle --deobf --show-bad-code -d <outputdir> <file.apk> //Usefull to get all the code (sometimes inconsistent code)
```

Open JAR file using a Java decompiler
```sh
jd-gui classes_dex2jar.jar
or use jadx
```

Extract the APK content, manifest.xml  and disassemble the smali code
```sh
apktool d application.apk
or use jadx
```

Rebuild APK file
```sh
apktool b <folder_with_content> -o application.apk
```

Align APK file
```sh
zipalign –f -v 4 application.apk application_align.apk
```

Generate smali code from ODEX file
```sh
baksmali –x file.odex –d /path-to-the-framework-folder/
```

Assemble smali code into a DEX file
```sh
smali out
```

Verify the application sign
```sh
jarsigner -verify -verbose -certs application.apk
```

Wireshark filter: network data leak
```sh
dns && ip.addr == <ip address>
tcp && ip.addr == <ip address> && (tcp.dstport == 80 || tcp.dstport == 443)
tcp && ip.addr == <ip address> && tcp.dstport != 80 && tcp.dstport != 443
udp && ip.addr == <ip address>
```

Generate keystore
```sh
keytool -genkey -v -keystore <keystore_name> -alias <alias> -keyalg RSA -keysize 2048 -validity 20000
```

Sign apk file
```sh
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore <keystore_name> application.apk <alias>
```

Add certificate (cacert.crt) to a keystore (apache.bks)
```sh
keytool -keystore apache.bks -storetype BKS -providerClass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-jdk15on-146.jar -importcert -v -trustcacerts -file cacert.crt -alias test
```

How to create a new self-signed certificate and keystore for BurpSuite
(https://unix.stackexchange.com/questions/347116/how-to-create-keystore-and-truststore-using-self-signed-certificate)
```sh
#1. Generate a private RSA key
openssl genrsa -out diagserverCA.key 2048

#2. Create a x509 certificate
openssl req -x509 -new -nodes -key diagserverCA.key -sha256 -days 500 -out diagserverCA.pem

#3. Create a PKCS12 keystore from private key and public certificate.
openssl pkcs12 -export -name server-cert -in diagserverCA.pem -inkey diagserverCA.key -out serverkeystore.p12

#4. Convert PKCS12 keystore into a JKS keystore
keytool -importkeystore -destkeystore server.keystore -srckeystore serverkeystore.p12 -srcstoretype pkcs12 -alias server-cert

#5. Import a client's certificate to the server's trust store.
keytool -import -alias client-cert -file diagclientCA.pem -keystore server.truststore

#6. Import a server's certificate to the server's trust store.
keytool -import -alias server-cert -file diagserverCA.pem -keystore server.truststore
```

How to Install Burp CA as a system-level trusted CA
(https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/)
1. Get the Burp Suite CA in DER format (cacert.der)
2. Use openssl to convert DER to PEM and rename it with the subject_hash_old as name:
```sh
openssl x509 -inform DER -in cacert.der -out cacert.pem  
openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1  
mv cacert.pem <hash>.0  
```
3. Move the certificate to the device and install it (root required)
```sh
adb shell
su
mount -o rw,remount /system  
exit
adb push <cert>.0 /sdcard/
adb shell
mv /sdcard/<cert>.0 /system/etc/security/cacerts/  
chmod 644 /system/etc/security/cacerts/<cert>.0
reboot
```

How to Debug
```sh
#Get the PID
adb shell 'ps | grep <app_name>' | awk '{print $2}'
#Port forwarding
adb forward tcp:8765 jdwp:<PID>
#Connect the debug
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=8765
```

## Using Drozer
(https://labs.mwrinfosecurity.com/tools/drozer/)

Shows devices availables
```sh
drozer console devices
```

Connect to a device
```sh
adb forward tcp:31415 tcp:31415
drozer console connect
```

Full list of commands
```sh
dz> ls
```

Provide the list of packages available
```sh
dz> run app.package.list
```

Provide details on the targeted package
```sh
dz> run app.package.info –a <package-name>
```

Identify exported components by the application
```sh
dz> run app.package.attacksurface <package-name>
```

Get exported activities
```sh
dz> run app.activity.info -a <package-name>
```

Start activity with weak permissions
```sh
dz> run app.activity.start –-component <package-name> <package.activity_name>
(adb shell) am start –m app.activity.start./ActivityName
```

Get exported services
```sh
dz> run app.service.info -a <package-name>
```

Interact with services
```sh
dz> run app.service.start --component <package> <package.service_name>
dz> run app.service.send <service_name> <message>
dz> run app.service.stop –-component <package> <package.service_name>
(adb shell) am startservice app.service.start/.ActivityName
```

Get exported receivers
```sh
dz> run app.broadcast.info -a <package-name>
```

Interact with receivers
```sh
dz> run app.broadcast.send --component <package> <package.receivers_name> --extra string <parameter>  <value of the parameter>
(adb shell) am broadcast -a <activity> –n app.broadcast.send/.ReceiverName –es <paramater> <value>
```

Find out the content URIs to extract info
```sh
dz> run scanner.provider.finduris -a <package_name>
```

Query to a URI
```sh
dz> run app.provider.query <URI>
```

Get the content provider’s information available
```sh
dz> run app.provider.info -a <package-name>
```

SQL injection module scanner
```sh
dz> run scanner.provider.injection -a <URI>
```

Attempt to exploit SQL injection
```sh
dz> run app.provider.query <URI> --projection “<payload>"
```

Enumerate all the table names
```sh
dz> run scanner.provider.sqltables -a <URI>
```

Directory traversal module scanner
```sh
dz> run scanner.provider.traversal -a <URI>
```

Read unauthorised files (if is vulnerable)
```sh
dz> run app.provider.read <URI>/etc/hosts
```

Download files
```sh
dz> run app.provider.download <URI>/<path_to_file> <local_folder>
```
