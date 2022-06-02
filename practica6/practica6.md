# Pràctica 6A: Busos de comunicació II (SPI, Lector SD)
L'objectiu d'aquesta pràctica consisteix en entendre el funcionament dels busos SPI i saber com implementar-los en els nostres projectes.

Els busos SPI conecten el modul principal, anomenat "master" (en el nostre cas la ESP32) amb els diferents components que volquem implementar, anomenats "slaves". Aquesta és una tecnologia de comunicació sincrona ja que tots els components comparteixen una senyal de rellotge.

 Aquests busos utilitzen un seguit cables per conectar el "master" amb els diferents "slaves" que volguem conectar:

  * SS / SDA / CS (prén diferents noms): "Slave Select" per aquest canal el master envia un senyal al slave amb el que vol establir la conexió. Cada slave té un SS propi.
  * RST: és el canal pel qual el master envia el senyal de que la conexió s'ha finalitzat. Cada slave té un RST propi.
  * CLK: és canal pel qual s'envia la senyal de rellotge que s'utilitza per poder sincronitzar les comunicacions. Aquest canal està conectat en serie amb tots els components, master i slaves.
  * MOSI (Master Out Slave In): és el canal pel qual el master envia la informació a tots els slaves, però solament es llegida per el slave que s'ha seleccionat previament amb el SS. Aquest canal està conectat amb tots els components, master i slaves.
  * MISO (Master In Slave Out): és el canal pel qual el slave envia la informació a tots els components però unicament es llegit pel master. Aquest canal està conectat amb tots els components, master i slaves.
___
## CODI DE LA PRÀCTICA
```
#include <Arduino.h>
#include <SPI.h>
#include <SD.h>

File myFile;

void setup()
{
  Serial.begin(9600);
  Serial.print("Iniciando SD ...");
  if (!SD.begin(4)) {
    Serial.println("No se pudo inicializar");
    return;
  }
  Serial.println("inicializacion exitosa");

  myFile = SD.open("pf32.txt");
  if (myFile) {
    Serial.println("pf32.txt:");
    while (myFile.available()) {
    	Serial.write(myFile.read());
    }
    myFile.close(); 
  } else {
    Serial.println("Error al abrir el archivo");
  }
}

void loop(){}
```  
No cal definir els pins que utilitzarem pel bus, ja que la ESP32 té assignats uns pins per defecte pel bus SPI. De fet, la ESP32 pot soportar fins a dos busos SPI. Cadascun dels quals amb pins propis per defecte, tot i que hi ha opcions per definir els canals SS i RST. 

(A la segona part d'aquesta practica es defineixen els pins SS i RST al crear l'objecte del lector RFId, i es pot utilitzar qualsevol pin que estigui disponible. Cal assegurar-se primer de quins pins de la ESP32 es poden utilitzar) 
___
## FUNCIONAMENT
El codi d'aquesta pràctica implementa un lector de targetes SD que es conecta mitjançant un bus SPI. Per tal de fer-ho, s'utilitzen les llibreries:
* "SPI.h": per la gestió bus 
* "SD.h": pel lector de targetes.

El programa simplement establirà la conexió amb la targeta i carregarà un fitxer que mostrarà pel terminal. Tot això fent un seguiment del procés a través del terminal.
___
### Declaració d'objectes
```
File myFile;
```
Declarem un objecte "File" (propi de la llibreria "SD.h") que serà l'arxiu on es carregarà el fitxer de la SD amb el que volguem treballar.
___
### Setup
```
void setup()
{
  Serial.begin(9600);
  Serial.print("Iniciando SD ...");
  if (!SD.begin(4)) {
    Serial.println("No se pudo inicializar");
    return;
  }
  Serial.println("inicializacion exitosa");
 
  myFile = SD.open("pf32.txt");//abrimos  el archivo 
  if (myFile) {
    Serial.println("pf32.txt:");
    while (myFile.available()) {
    	Serial.write(myFile.read());
    }
    myFile.close(); //cerramos el archivo
  } else {
    Serial.println("Error al abrir el archivo");
  }
}
```
Primerament es mostra pel terminal: "Iniciando SD...". 

A continuació trobem un condicional que revisa si s'ha realitzat correctament la conexió amb el lector SD. En cas de que la conexió no s'hagi aconseguit es mostrarà el missatge "No se pudo inicializar" i aplicarà un "return" que terminarà el procés: 
```
  Serial.print("Iniciando SD ...");
  if (!SD.begin(4)) {
    Serial.println("No se pudo inicializar");
    return;
  }
```
Per contra, si no hi ha hagut cap problema, el procés no és terminarà i mostrarà pel terminal "inicialización exitosa" (això voldrà dir, entre altres coses, que hem conectat correctament el bus SPI):
```
  Serial.println("inicializacion exitosa");
```
Un cop establerta la conexió, carreguem el fitxer que desitgem al objecte "myFile" creat anteriorment. En aquest cas es tracta d'un fitxer de text anomenat "pf32.txt". En cas de que es trobi dins d'una o algunes carpetes, caldrà escriure la direcció del fitxer:
```
myFile = SD.open("pf32.txt");
```
Seguidament s'executa un altre condicional que revisa que s'ha pogut carregar el fitxer correctament. En cas afirmatiu es mostrarà el contingut a traves de la terminal i, un cop acabat, tancarà el fitxer:
```
  if (myFile) {
    Serial.println("pf32.txt:");
    while (myFile.available()) {
    	Serial.write(myFile.read());
    }
    myFile.close(); 
  }
```
Si, per contra, s'ha produit un error al obrir el fitxer, aquest no es podrà llegir i ens apareixerà un missatge d'error pel terminal:
```
  else {
    Serial.println("Error al abrir el archivo");
  }
```