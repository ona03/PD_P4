# Procesadores Digitales - Práctica 4

## Objetivo
<div align="justify">
El objetivo de esta práctica consiste en comprender el funcionamiento de WIFI y bluetooth. Para ello generaremos una web utlizando el procesador esp32, usando lenguaje html. También estableceremos una comunicación serie con una aplicacion de móvil pudiendo mantener una conversación a través de esta aplicación. Además, implementaremos un código que permite encender y apagar unos leds mediante connexión bluetooth generando una página con dirección ip.
(decidir si volem posar el codi que escanejava tots els dispositius bluetooth)

## Generación de una página web

En primer lugar, para realizar una correcta connexión, deberemos incluir la libreria de `<WIFI.h>` que nos servirá para connectarnos sin problemas a la red wifi; y la de `<WebServer.h>` que nos proporcionará métodos de configuración del servidor y no tener problemas la implementación de la connexión, además de la de `<Arduino.h>`.

```
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>
```
Después de eso, debemos incluir la red wifi en la cual queremos trabajar. Para ello, creamos dos cadenas de carácteres, una para el nombre de la red (SSID: Service Set Identifier) y otra para la contraseña. 

```
const char* ssid = "*********"
const char* password = "**********"
```
A continuación, creamos un objeto de WebServer para poder acceder a las funciones de la librería en cuestión. Utilizamos como parámetro de entrada el puerto en el que va a escuchar el servidor al que estemos conectados. El valor por defecto es de 80 ya que es el puerto que usa HTTP, por lo que esto permite connectarnos al servidor sin tener que especificar el puerto en la URL.
```
WebServer server(80);
```
Como último paso antes de empezar el `setup()`, declaramos la variable que almacenará el contenido de nuestra página web en lenguaje html. Ésta será un `string`.

```
String HTML = "<!DOCTYPE html>\
  <html>\
  <body>\
  <h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
  </body>\
  </html>";
```
Una vez eso hecho, nuestro programa intentará establecer connexión en la red wifi con la función `WIFI.begin(ssid,password)`, con los valores de nuestra red como parámetros de entrada. Justo después, se detendrá en un bucle hasta que no se connecte, comprobando cada 1 segundo si la connexión ha sido exitosa o no con `WL_CONNECTED`. 
```
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print(".");
}
```
En cuanto se haya connectado, nos avisará por el terminal que se habrá establecido connexión exitosa e imprimirá la IP del servidor web generado con `Serial.println(WiFi.localIP())`. El siguiente paso es determinar qué código debe ejecutarse cuando se accede a una URL específica. Con `server.on("/", handle_root)`, indicamos que cuando un servidor recibe una solicitud HTTP en la URL `('/')`, llamará a la función `handle_root`. Con esa condición declarada, ya podemos inicializar el web server con `server.begin()` y lo hacemos saber al usuario por el terminal. 
```
Serial.println("WiFi connected successfully");
Serial.print("Got IP: ");
Serial.println(WiFi.localIP()); 
server.on("/", handle_root);
server.begin();
Serial.println("HTTP server started");
```
Si nos centramos en la función `handle_root()` que llamamos en el `setup()`, consiste en enviar el código `200` de estado HTTP, que se corresponde con respuesta `OK`, especificamos el tipo en `"text/html"` y finalmente le pasamos la variable con el contenido deseado (fichero html).
```
void handle_root() {
  server.send(200, "text/html", HTML);
}
```
Para finalizar, dentro del `loop()` tan solo recibimos las peticiones del usuario de direcciones HTTP.
```
void loop() {
  server.handleClient();
}
```
Por lo que juntándolo todo, queda el siguiente código:
```
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>

// SSID & Password
const char* ssid = "NetworkName";
const char* password = "*********";

WebServer server(80);
void handle_root();

String HTML = "<!DOCTYPE html>\
  <html>\
  <body>\
  <h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
  </body>\
  </html>";

// Handle root url (/)

void setup() {
  Serial.begin(115200);
  Serial.println("Try Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected successfully");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP()); 
  server.on("/", handle_root);
  server.begin();
  Serial.println("HTTP server started");
  delay(100);
}

void loop() {
  server.handleClient();
}

void handle_root() {
  server.send(200, "text/html", HTML);
}
```
Con una salida:
```
Try Connecting to 'NetworkName'
..........
WiFi connected successfully
Got IP: 
HTTP server started
```
Para ver el fichero html creado, tan solo hay que sustituir el contenido de la variable de HTML en el trozo de código en el que hemos escrito la página web de prueba. Es decir, escrivimos `String HTML = "fitxer_html.MD.html";`
en vez de:
```
String HTML = "<!DOCTYPE html>\
  <html>\
  <body>\
  <h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
  </body>\
  </html>";
```