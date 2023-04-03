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
    - Az alap paraméterek:
        - T = 1.0
        - k = 5
        - TvalosIdo = T/k
        - Frequency 3
        - fill = 50

    - Lámpa idők, melyek kiteszik a tOsszido-t.
        - tOsszido = tPiros + tPirosSarga + tZold + tSarga
            - tPiros = 10
            - tPirosSarga = 3
            - tZold = 10
            - tSarga = 3
4. Az algoritmus megvalósítása.
    - Létrehoztunk egy végtelen ciklust.
        - Az első if elágazásban, összehasonlítottuk a t-t az tOsszido-vel, hogyha azonos a t változó a tOsszido-vel akkor a t-t lenulázzuk.
        - Következő elágazásban, ha t egyenlő volt 0-val:
            - Akkor a JarmuLampa piroson, még a GyalogLampa zölden világított.
                - JarmuLampa: Piros (True,False,False)
                - GyalogLampa: Zöld (False,True)
        - A harmadik elágazásban, ha t egyenlő tPiros-sal akkor:
            - JarmuLampa : piros-sárgán világít, míg a GyalogLampa : zold-en villog.
                - JarmuLampa: Piros-Sarga (True, True, False).
            - Elértünk a p változóhoz
5. Tesztelés.





# 1. A teszt 

Megvalósítás:
```py
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


```
# A program működése
    1-6. sorig inizaliáljuk a programot
    9-13. sorig kiosztjuk a pineket táblázat alapján
    a változókat beleraktuk a pinek tömbjébe
    - 19. sortól a 20. sorig pinek kimenetnek definiálása
        -forciklusal definiáltuk a pinek kimenetként
## Függvények létrehozása:
     -létrehoztuk a JarmuLampa függvényt aminek a paraméterei a pinek szinei(piros, sárga, zöld)
     -létrehoztuk a GyalogLampa függvényt aminek a paraméterei a pinek szinei(piros, zöld)
## Teszt függvény:
     -létrehoztuk az onOff változót amibe beleraktuk True-t
     -létrehoztuk egy forciklust aminek segítségével n-szer fel-le kapcsoljuk a ledeket, majd lekapcsolja a ledeket
## A progrsm paraméterei:
    - a T változó az idő ,ami 1
     - a k változó  a gyorsításí faktor ,ami 5
     - a TvalosIdo a változo ,ami a T és a k hányadosa
     - a frequency a frekvencia ,ami 3
     - a fill kitöltés ,ami 50
## A gyalogos zöld villogási frekvenciája:
     - az f változó ami valósidő és a T kétsueresének a hányadosa
## A lámpák égési ideje:
     - tpiros és a tZold 10 másodpercig még a tSargaPiros és tSarga 3 másodpercig ég
## Összidö:
     -Létrehoztuk a tOssziod változót ami a tPiros, tPirosSarga, tZold, tSarga összege
## A program indulási pontja:
     - étre hoztunk egy try-t ami ismétli az algoritmust
         - a try-ba meghívtuk a test függvényt aminek paraméterként odaadtunk 10-t vagys 10-szer ismétli meg test függvényünket
## Az algoritmus elkezdése:
    - létrehoztunk ehy While ciklust amine egy True-t adtunk oda 
        -megvizsgáltuk az tOsszido-t , ha eléri az összidőt akkor újra indul a ciklus
        - a cikluson belül meghívtuk a JarmuLampa és a GyalogLampa függényeket
        - az hogy melyik lámpa égjen a True és a False paraméterektöl függ pl:a jármű lámpa piros akkor az  JarmuLampa(True,False,False) még a gyalogos lámpa zöld  GyalogLampa(False,True)
        - a ciklus végén valós órajel ideig várakozunk 
## Az egész lezárása:
    except KeyboardInterrupt:
    print("Pinek lekapcsolva")
    GPIO.cleanup()