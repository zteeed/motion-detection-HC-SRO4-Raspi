# motion-detection-HC-SRO4-Raspi

Lien de l'article original: https://wiki.minet.net/wiki/divers/internet

## Internet 

Vous vous endormez pendant la perm et avez peur de ne pas être assez réactif à l'arrivée d'un adhérent ? Ne vous inquiétez pas, ce projet inutile est fait pour vous. Objectif: Faire gueuler INTERNET sur la sono quand un adhérent passe la porte. [Musique Originale](https://www.youtube.com/watch?v=emfEwJyPIgw)

L'utilité du projet n'est pas l'objectif, c'est de faire joujou afin de remplir le palmarès de la ZoulouTech.

### Setup localement

La vidéo du résultat du setup local est sur [nextcloud](https://nextcloud.minet.net)

Matériel dont vous avez besoin: \
- Une pointe de notion unix \
- Une Raspi \
- Un capteur HC-SR04 \

En principe, il faut réaliser un pont diviseur de tension afin que l'alimentation de la raspi (5V) soit en phase avec celle dont à besoin le capteur (3.3V) MAIS, après avoir fait sans (#zouloute) ça marche quand même donc osef :D

Il faut noter que les numéros des pins sont reliés à des numéros différents de GPIO (Input/Output).
Donc pour les branchements faire la même chose pour les puristes sinon bypass le pont diviseur et brancher: \
- VCC: Pin 2 \
- Trig: Pin 18 (GPIO 23) \
- Echo: Pin 20 (GPIO 24) \
- Gnd: Pin 6 \
![Schéma explicatif](https://wiki.minet.net/_media/wiki/divers/interfacing-raspberry-pi-with-hc-sr04-circuit-diagram-600x361.png?w=800&tok=e5c951)

Le code python pour récupérer la distance et lancer la musique est ci-dessous. Si il manque des librairies, il faut utilier pip. Je précise que ce script tourne sous Python2.7 (cc IB)
```bash
sudo apt install python-pip
pip install *python_lib*
```

```python
#Libraries
import RPi.GPIO as GPIO
import time
import subprocess

#GPIO Mode (BOARD / BCM)
GPIO.setmode(GPIO.BCM)

#set GPIO Pins
GPIO_TRIGGER = 23
GPIO_ECHO = 24

#set GPIO direction (IN / OUT)
GPIO.setup(GPIO_TRIGGER, GPIO.OUT)
GPIO.setup(GPIO_ECHO, GPIO.IN)

dist_min = 70 # min distance to detect

def distance():
    # set Trigger to HIGH
    GPIO.output(GPIO_TRIGGER, True)

    # set Trigger after 0.01ms to LOW
    time.sleep(0.00001)
    GPIO.output(GPIO_TRIGGER, False)

    StartTime = time.time()
    StopTime = time.time()

    # save StartTime
    while GPIO.input(GPIO_ECHO) == 0:
        StartTime = time.time()

    # save time of arrival
    while GPIO.input(GPIO_ECHO) == 1:
        StopTime = time.time()

    # time difference between start and arrival
    TimeElapsed = StopTime - StartTime
    # multiply with the sonic speed (34300 cm/s)
    # and divide by 2, because there and back
    distance = (TimeElapsed * 34300) / 2

    return distance

if __name__ == '__main__':
    try:
        while True:
            dist = distance()
            print ("Measured Distance = %.1f cm" % dist)
            if dist<dist_min:
                print "Detected"
                start mplayer
                song = 'soung.mp3'
                cmd = ['mplayer', '-slave', '-quiet', song]
                p = subprocess.Popen(cmd, stdout=subprocess.PIPE, stdin=subprocess.PIPE)
            time.sleep(0.1)

        # Reset by pressing CTRL + C
    except KeyboardInterrupt:
        print("Measurement stopped by User")
        GPIO.cleanup()
```

Changer le keyboard en français sur la raspi: `loadkeys fr` Si ça ne fonctionne pas:
```bash
apt-get update
apt-get install console-data
```

Pour forcer l'output du son sur le jack et/ou HDMI: [ici](https://www.raspberrypi.org/documentation/configuration/audio-config.md)
```bash
sudo raspi-config
Select Advanced Options and press Enter
Select Audio and press Enter
```


### Setup via Ethernet

La raspi connectée à la sono [raspbarrywhite](https://wiki.minet.net/wiki/divers/raspbarrywhite|raspbarrywhite) est dans le vlan 147. Il faut donc statiquement config l'IP de notre raspi dans le même vlan. Il faut également modifier le vlan du port du switch du local (le 43 par exemple) sur laquelle vous branchez la raspi.

Se connecter sur le switch du local (derrière le VPN)
```bash
ssh minet@192.168.102.219 -oKexAlgorithms=diffie-hellman-group1-sha1 -oCiphers=aes128-cbc
```

Ensuite on regarde la conf du port sur lequel vous êtes branché, on flush tout et on reconfig:
```bash
switch-local>enable
Password: *type_password*
switch-local#show run | begin interface GigabitEthernet1/0/43
switch-local#config terminal
switch-local(config)#default interface gigabitEthernet 1/0/43
switch-local(config)#interface gigabitEthernet 1/0/43
switch-local(config-if)#switchport mode access
switch-local(config-if)#switchport access vlan 147
```

Maintenant que le port est configuré on choisit une IP non utilsée (suffit de ping et de voir si on a pas de réponse). Si on choisit 192.168.147.5 par exemple, on modifie /etc/network/interfaces:

```bash
auto eth0
iface eth0 inet static
  address 192.168.147.5
  netmask 255.255.255.0
  gateway 192.168.147.1
```

On restart le service
```bash
/etc/init.d/networking restart
```

Dans le vlan 147, ne pas oublier pour avoir internet
```bash
export http_proxy="http://192.168.147.61:82"
export https_proxy="http://192.168.147.61:82"
```

Pour la fin: suivre le tutoriel d'IB [raspbarrywhite]([https://wiki.minet.net/wiki/divers/raspbarrywhite|raspbarrywhite)

Article rédigé par votre troll préféré: zTeeed
