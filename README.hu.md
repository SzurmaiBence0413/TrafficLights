# Feladat lépései
0. A projekt fájl: Projektek2023/TrafficLights/TrafficLights.py
1. A teszt 
    - JarmuLampa(piros,sarga,zold)
        - A paramétereknek megfelelően, kigyújtja a JarmuLampa ledeit.
    - GyalogLampa(piros,zold)
        - A paramétereknek megfelelően, kigyújtja a JarmuLampa ledeit.
    - test(n)
        - n-szer megvillogtatja a ledeket.

2. Az algoritmus tanulmányozása és megértése.
***
3. Az alap paraméter elkészítése.
***
4. Az algoritmus megvalósítása.

5. Tesztelés.

Megvalósítás:
```py
import RPi.GPIO as GPIO
import time

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.cleanup()

pJarmuPiros = 21
pJarmuSarga = 20
pJarmuZold = 16
pGyalogPiros = 26
pGyalogZold = 19

pinek = [pJarmuPiros, pJarmuSarga, pJarmuZold, pGyalogPiros, pGyalogZold]

for pin in pinek:
    GPIO.setup(pin, GPIO.OUT)

def JarmuLampa(piros,sarga,zold):
    GPIO.output(pJarmuPiros, piros)
    GPIO.output(pJarmuSarga, sarga)
    GPIO.output(pJarmuZold, zold)
        
def GyalogLampa(piros,zold):
    GPIO.output(pGyalogPiros, piros)
    GPIO.output(pGyalogZold, zold)

def test(n):
    print("Teszt...")
    onOff = True
    for i in range(n):
        JarmuLampa(onOff, onOff, onOff)
        GyalogLampa(onOff, onOff)
        onOff = not onOff
        time.sleep(0.5)
    onOff = False
    JarmuLampa(onOff, onOff, onOff)
    GyalogLampa(onOff, onOff)
    print("Teszt vége")
    
test(5)

try:
    while True:
        time.sleep(1)
        print("Hello")
            
           
except KeyboardInterrupt:
    print("Pinek lekapcsolva")
    GPIO.cleanup()

```

