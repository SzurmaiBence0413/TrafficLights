import RPi.GPIO as GPIO
import time

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.cleanup()

#A Pin kiosztás táblázat alapján
pJarmuPiros = 21
pJarmuSarga = 20
pJarmuZold = 16
pGyalogPiros = 26
pGyalogZold = 19

#Pinek tömbjének létrehozása
pinek = [pJarmuPiros, pJarmuSarga, pJarmuZold, pGyalogPiros, pGyalogZold]

#Pinek kimenetnek definiálása ciklussal
for pin in pinek:
    GPIO.setup(pin, GPIO.OUT)

#Függvények
def JarmuLampa(piros,sarga,zold):
    GPIO.output(pJarmuPiros, piros)
    GPIO.output(pJarmuSarga, sarga)
    GPIO.output(pJarmuZold, zold)
        
def GyalogLampa(piros,zold):
    GPIO.output(pGyalogPiros, piros)
    GPIO.output(pGyalogZold, zold)
    
#Teszt függvény
def test(n):
    onOff = True
    for i in range(n):
        JarmuLampa(onOff, onOff, onOff)
        GyalogLampa(onOff, onOff)
        onOff = not onOff
        time.sleep(0.5)
    onOff = False
    JarmuLampa(onOff, onOff, onOff)
    GyalogLampa(onOff, onOff)
    
#A program paraméterei
T = 1.0
k = 5
TvalosIdo = T/k

frequency = 3
fill = 50

#Gyalogos zöld villogási frekvenciája:
f = TvalosIdo/T*2
t = 0

#Lámpa idők: Meddig égnek a lámpák
tPiros = 10
tPirosSarga = 3
tZold = 10
tSarga = 3

#Összidő
tOsszido = tPiros + tPirosSarga + tZold + tSarga

#A program indulási pontja
try:
    #Teszt indítása (A lámpák villanjanak 5-ször)
    test(10)
    print("Traffic Lights, online, kilépéshez nyomd meg a CTRL+C kombinációt!")
    print(f"Gyorsítási érték : {k}")
    while True:
        if t == tOsszido:
            #Ha elértük az összidőt, az idő induljon újra
            t = 0
        
        #Kezdés:
        if t == 0:
            #Jármű lámpa: piros
            JarmuLampa(True,False,False)
             #Gyalogos lámpa: zöld
            GyalogLampa(False,True)
        
        #Vége a pirosnak (jármű)        
        if t == tPiros:
            #Jármű lámpa: piros-sárga
            JarmuLampa(True, True, False)
             #Gyalogos zöld f-el villog
            p = GPIO.PWM(pGyalogZold, frequency)
            p.ChangeFrequency(frequency)
            p.ChangeDutyCycle(fill)
            p.start(fill)
            
        #Vége a piros-sárgának (jármű)
        if t == tPiros + tPirosSarga:
             #Jármű lámpa: zöld
            JarmuLampa(False, False, True)
            p.stop()
            #Gyalogos lámpa piros
            GyalogLampa(True, False)
            
        #Vége a zöldnek (jármű)
        if t == tPiros + tPirosSarga + tZold:
            #Jármű lámpa: sárga
            JarmuLampa(False, True, False)
            #Gyalogos lámpa piros
            GyalogLampa(True, False)
        
        t += 1
         #Valós órajel ideig várakozunk
        time.sleep(TvalosIdo)
            
        
except KeyboardInterrupt:
    print("Pinek lekapcsolva")
    GPIO.cleanup()
