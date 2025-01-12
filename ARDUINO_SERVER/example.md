// #include <Arduino.h>
// #include <Adafruit_NeoPixel.h>

// // Configuración del panel
// #define PIN 0            // Pin conectado al panel (D4 en Wemos D1 Mini)
// #define NUM_LEDS 16      // Número total de LEDs (4x4 matriz)
// #define DELAY_TIME 500   // Tiempo de encendido por LED en milisegundos

// // Inicializar el objeto de NeoPixel
// Adafruit_NeoPixel strip(NUM_LEDS, PIN, NEO_GRB + NEO_KHZ800);

// void setup() {
//   strip.begin();         // Inicializar el strip
//   strip.show();          // Apagar todos los LEDs al inicio
// }

// void loop() {
//   for (int i = 0; i < NUM_LEDS; i++) {
//     strip.clear();       // Apagar todos los LEDs antes de encender uno nuevo
//     strip.setPixelColor(i, strip.Color(5, 255, 0)); // Encender en rojo
//     strip.show();        // Mostrar los cambios
//     delay(DELAY_TIME);   // Esperar el tiempo definido
//   }
// }

#include <Adafruit_NeoPixel.h>
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// Configuración de los LEDs
#define PIN 0          // Pin donde están conectados los LEDs
#define NUMPIXELS 16    // Número de LEDs en la matriz

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

// Configuración WiFi
const char* ssid = "Fibertel WiFi758 2.4GHz";         // Cambia por tu SSID
const char* password = "9735517504"; // Cambia por tu contraseña

ESP8266WebServer server(80); // Servidor web en el puerto 80

void handleRoot() {
  // Página principal para control manual
  String html = "<html><body>"
                "<h1>Control de NeoPixels</h1>"
                "<form action='/setPixel'>"
                "LED: <input type='number' name='id' min='0' max='15'><br>"
                "Color (HEX): <input type='text' name='color' placeholder='#RRGGBB'><br>"
                "<button type='submit'>Enviar</button>"
                "</form>"
                "</body></html>";
  server.send(200, "text/html", html);
}

void handleSetPixel() {
  // Control individual de LEDs a través de parámetros
  if (server.hasArg("id") && server.hasArg("color")) {
    int ledId = server.arg("id").toInt();
    String color = server.arg("color");

    if (ledId >= 0 && ledId < NUMPIXELS) {
      long hexColor = strtol(color.c_str() + 1, NULL, 16); // Convierte HEX a RGB
      byte r = (hexColor >> 16) & 0xFF;
      byte g = (hexColor >> 8) & 0xFF;
      byte b = hexColor & 0xFF;

      strip.setPixelColor(ledId, strip.Color(r, g, b));
      strip.show();
      server.send(200, "text/plain", "LED actualizado");
    } else {
      server.send(400, "text/plain", "ID de LED fuera de rango");
    }
  } else {
    server.send(400, "text/plain", "Parámetros inválidos");
  }
}

void setup() {
  // Configuración inicial
  Serial.begin(115200);
  strip.begin();
  strip.show(); // Apaga todos los LEDs al inicio

  // Conexión WiFi
  WiFi.begin(ssid, password);
  Serial.print("Conectando a WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nConectado a WiFi");
  Serial.println(WiFi.localIP());

  // Configuración del servidor web
  server.on("/", handleRoot);         // Página principal
  server.on("/setPixel", handleSetPixel); // Ruta para actualizar LEDs
  server.begin();
  Serial.println("Servidor iniciado");
}

void loop() {
  // Manejador del servidor web
  server.handleClient();
}
