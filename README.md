# TFT28-dietpi
Comment utiliser un écrant tactile TFT 2.8" sur Raspberry Pi + dietpi

1. Installer dietpi https://dietpi.com/
2. Installer Python3 git raspi-config
3. sudo raspi-config > Enable Remode GPIO (daemon pigpiod)
4. git clone https://github.com/adafruit/Raspberry-Pi-Installer-Scripts.git
5. cd Raspberry-Pi-Installer-Scripts
6. sudo pip3 install Click
7. chmod +x adafruit-pitft.py
8. sudo python3 adafruit-pitft.py --display=28r --rotation=90 --install-type=console -u /home/dietpi
9. sudo reboot
10. echo "test" > /dev/tty1
11. Code pour Tests boutons (avec RPi.GPIO : sudo apt install python3-rpi.gpio)

#!/usr/bin/python3
import RPi.GPIO as GPIO
import time
def bouton_2(channel):
  print ("Bouton 2 (GPIO" + str(channel) + ") appuyé")
def bouton_3(channel):
  print ("Bouton 3 (GPIO" + str(channel) + ") appuyé")
def bouton_4(channel):
  print ("Bouton 4 (GPIO" + str(channel) + ") appuyé")
def bouton_1(channel):
  print ("Bouton 1 (GPIO" + str(channel) + ") appuyé")
boutons = [18,22,23,27]
GPIO.setmode(GPIO.BCM)
GPIO.setup(boutons, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.add_event_detect(18, GPIO.FALLING, callback=bouton_4, bouncetime=250)
GPIO.add_event_detect(22, GPIO.FALLING, callback=bouton_2, bouncetime=250)
GPIO.add_event_detect(23, GPIO.FALLING, callback=bouton_1, bouncetime=250)
GPIO.add_event_detect(27, GPIO.FALLING, callback=bouton_3, bouncetime=250)
print ("J'attends 30secondes...")
time.sleep(30)
GPIO.cleanup()

12. Code pour Bouton PowerOFF :
#!/usr/bin/env python3
# d'après https://fearby.com/article/using-python-to-use-buttons-on-the-pitft-plus-320x240-tft-touch>
# et adapté à pigpio
import pigpio
import os
import time
Bouton1 = 23 # relié à GPIO23
Poweroff = False
def Int_Bouton1(pin, level, tick):
  global Poweroff
  if not (Poweroff):
    #print ("Bouton 1 appuyé", pin, level, tick)
    Poweroff = True
    os.system("sudo poweroff")
pi = pigpio.pi()
if pi.connected:
  pi.set_pull_up_down(Bouton1, pigpio.PUD_UP)
  pi.set_mode(Bouton1, pigpio.INPUT)
  int23 = pi.callback(Bouton1, pigpio.FALLING_EDGE, Int_Bouton1)
  while True:
    time.sleep(1)
else:
  print("Impossible d'accéder à PIGPIO, fin.")
  exit(1)

13. Création du service : sudo nano /etc/systemd/system/shutdown-press-simple.service
[Unit]
Description=Script bouton Poweroff
After=multi-user.target
[Service]
User=dietpi
WorkingDirectory=/home/dietpi
ExecStart=/usr/bin/python3 /home/dietpi/shutdown-press-simple.py
[Install]
WantedBy=multi-user.target

14. sudo systemctl enable shutdown-press-simple.service
15. sudo systemctl start shutdown-press-simple.service
16. sudo systemctl status shutdown-press-simple.service
