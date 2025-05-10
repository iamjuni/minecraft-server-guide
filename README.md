# For this guide we wii be using openbsd

1. Create a Dedicated User
```sh
doas useradd -m -s /bin/ksh mcserver
```
```sh
doas passwd mcserver
```

2. Create Server Directory
```sh
doas mkdir -p /home/mcserver/minecraft
```
```sh
doas chown mcserver:mcserver /home/mcserver/minecraft
```

3. Install Java (via openjdk)
```sh
doas pkg_add jdk
```

4. Download Minecraft Server
As user mcserver:
```sh
doas su - mcserver
cd minecraft
fetch https://piston-data.mojang.com/v1/objects/e6ec2f64e6080b9b5d9b471b291c33cc7f509733/server.jar
```

5. Accept EULA
```sh
echo "eula=true" > eula.txt
```

6. Download Server Properties
```sh
fetch https://raw.githubusercontent.com/iamjuni/minecraft-server-guide/refs/heads/main/server.properties
```

7. Open Port in PF (Firewall)
Edit /etc/pf.conf and add:
```sh
pass in on egress proto tcp from any to any port 29920
```
Apply PF rules
```sh
doas pfctl -f /etc/pf.conf
doas pfctl -e
```

8. Start the Server
As mcserver:
```sh
cd ~/minecraft
java -Xmx4096M -Xms4096M -jar server.jar nogui
```

9. Run as Background Service
Create /etc/rc.d/minecraft:
```sh
doas vi /etc/rc.d/minecraft
```
Paste:
```sh
#!/bin/ksh

daemon="/usr/local/bin/java"
daemon_flags="-Xmx4096M -Xms4096M -jar /home/mcserver/minecraft/minecraft_server.jar nogui"
daemon_user="mcserver"
. /etc/rc.d/rc.subr

rc_cmd $1
```
Make it executable:
```sh
doas chmod +x /etc/rc.d/minecraft
```
Enable and start:
```sh
doas rcctl enable minecraft
doas rcctl start minecraft
```

Done!
Your Minecraft server is now running on port 29920 under the mcserver user.
