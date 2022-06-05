# IOT-proyect
proyecto de IOT control de led, temperatura,presencia y luz



/*
Diplomado IoT Marzo 2022
Práctica 21:
En esta práctica la tarjeta se suscribe a un tópico llamado LED, del 
cual se obtiene la información necesaria para realizar el encendido 
o apagado del led incorporado de la tarjeta. Este mensaje lo publica 
un cliente adicional en el Broker.
*/

#include <WiFi.h> 
#include <PubSubClient.h>
  

// Credenciales MQTT
const char *mqtt_server = "34.196.176.135"; 
const int   mqtt_port   = 1883;
const char *mqtt_user   = "OSCAR_MQTT";
const char *mqtt_pass   = "OSCAR_MQTT";

// Tópicos
const char *Raiz_Sus =    "Raiz_OSCAR";
const char *Raiz_Led_Sus= "Raiz_OSCAR/Habitacion1/Led";
//Raiz_OSCAR

//Raiz_OSCAR/Habitacion1/Led
//Raiz_OSCAR/Habitacion1/Temperatura
//Raiz_OSCAR/Habitacion1/ControlAM
//Raiz_OSCAR/Habitacion1/ventilador

//Raiz_OSCAR/Habitacion2/Led
//Raiz_OSCAR/Habitacion2/ControlAM
//Raiz_OSCAR/Habitacion2/Presencia

//Raiz_OSCAR/Pasillo/Led
//Raiz_OSCAR/Pasillo/Intensidad



// Credenciales de WiFi
const char* ssid     = "Totalplay-DAA0";
const char* password = "DAA07D04bKD7NT9s";

bool Led_Status = 0;

// Variables Globales
WiFiClient espClient;
PubSubClient client(espClient);
char msg[25];
long count=0;
int contconexion = 0;

// Funciones
void callback(char* topic, byte* payload, unsigned int length);
void reconnect();
void setup_wifi();

void setup() {
  Serial.begin(115200);
  pinMode(4, OUTPUT); 
  digitalWrite(4, LOW);
  // Configuración conexión WIFI
  setup_wifi();
  // Configuración conexión MQTT
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  // Se pregunta si el cliente está conectado
  if (!client.connected()) {
    // Si no está conectado se realiza la reconexión
    reconnect();
  }

  if (client.connected()){

  
   
    count++;
    delay(100);
  }
  client.loop();

    if(Led_Status==1)
    {
      digitalWrite(4, HIGH);
      //Led_Status=0;
    }
    else
    {
      digitalWrite(4, LOW);
    }
  
}

//Conexión WiFi
void setup_wifi(){
  delay(10);
  delay(10);
  // Conexión WIFI
  Serial.println();
  Serial.print("Conectando a ssid: ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED and contconexion <50) { 
    ++contconexion;
    delay(500);
    Serial.print(".");
  }
  if (contconexion <50) {
      Serial.println("");
      Serial.println("WiFi conectado");
      Serial.println(WiFi.localIP());
  }
  else { 
      Serial.println("");
      Serial.println("Error de conexion");
  }
}

// Conexión MQTT
void reconnect() {
  // Si no hay conexión se entra dentro del ciclo
  while (!client.connected()) {
    Serial.println("Intentando conexión Mqtt...");
    // Se crea un cliente ID
    String clientId = "DIoT_Cliente";
    clientId += String(random(0xffff), HEX);
    // Se intenta realizar l aconexión conectar
    if (client.connect(clientId.c_str(),mqtt_user,mqtt_pass)) {
      // Si hay conexión se imprime el mensaje
      Serial.println("Conectado con el Broker!");
      // Nos suscribimos al tópico Raíz/Piso3
      if(client.subscribe(Raiz_Sus)){
        Serial.print("Suscripcion --");
        Serial.println(Raiz_Sus);
        Serial.println(" -- ok");
      }else{
        Serial.println("fallo Suscripciión a --");
        Serial.println(Raiz_Sus);
      }
      // Nos suscribimos al tópico Raíz/Piso3/Sala
      if(client.subscribe(Raiz_Led_Sus)){
        Serial.print("Suscripcion --");
        Serial.println(Raiz_Led_Sus);
        Serial.println(" -- ok");
      }else{
        Serial.println("fallo Suscripciión a --");
        Serial.println(Raiz_Led_Sus);
      }
    } else {
      // Si no hay conexión se imprime el mensaje, el estado de conexión
      // y se intenta nuevamente dentro de 5 segundo
      Serial.print("falló :( con error -> ");
      Serial.print(client.state());
      Serial.println(" Intentamos de nuevo en 5 segundos");
      delay(5000);
    }
  }
}

// Función para cuando se recibe un mensaje por parte del broker de un tópico al
// cual se está subscrito
void callback(char* topic, byte* payload, unsigned int length){
  String incoming = "";
  Serial.print("Mensaje recibido desde -> ");
  Serial.print(topic);
  Serial.println("");
  for (int i = 0; i < length; i++) {
    incoming += (char)payload[i];
  }
  incoming.trim();
  Serial.println("Mensaje -> " + incoming);


  if (strcmp(topic,Raiz_Led_Sus)==0)
  {
    int NUMERO = incoming.toInt();
    if(NUMERO==1)
    {
      Led_Status=1;
    }
    else
    {
      Led_Status=0;
    }
    
  }
}
