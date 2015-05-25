# fauteuil
fauteuil motorisé STI2D SIN Bernard Palissy

#include <math.h>

/*
  ILS
*/

int sensorPin= 8; // Pin reli?e au capteur magn?tique
int etat = 0; // etat Pin  
int ancien_etat = 0; // ancien etat Pin  
long chrono = 0; // valeur courante du chrono  
long chrono_depart = 0; // valeur de d?part du chrono  
long duree_test = 15000; // test sur 15 secondes  
int nb_chgt = 0; // nb de changement etat Pin
int mesure_vitesse = 0; // mesure vitesse
float perimetre = 0.00;

float KMH = 3; // varaible sur la broche 3 PWM

/*
  Le Moteur
  nom des variables dans la sens de la marche
*/

const int RoueDroite = 7  ;
const int RoueGauche = 10;
int ArriereDroite = 12;
int ArriereGauche = 13;
int select = 18;
/* 
  Zone en fonction de la position du Joystick
*/

int port_v = A0; // joystick mouvement vertical 
int port_h = A1; // joystick mouvement horizontal     
int valeur_v = 0; 
int valeur_h = 0; 
int zone = 0;

/*
  Valeurs pour le calcul de Tension
*/
int LongueurMax = 703;
int BaseX = 541;
int BaseY = 513;
float ValMot = 255.00;
float intense_old = 00.00;
float intense = 0.00;
float valeur = 0.00;
float acceleration = 0.00;
float equilibrage = 0.00;

void setup() 
{ 
 Serial.begin(9600);
 //configuration en sortie de la broche 9 & 10 & 11 & 12 & 13 & 3
 pinMode(RoueDroite, OUTPUT);
 pinMode(RoueGauche, OUTPUT);
 pinMode(ArriereDroite, OUTPUT);
 pinMode(ArriereGauche, OUTPUT);
 pinMode(KMH, OUTPUT);
 pinMode(select, INPUT); 
 pinMode(sensorPin, INPUT);
 chrono_depart = millis();
}

void calcul_zone(int x, int y) // FONCTION DEFINITION DES ZONES
{  
  if (x <= 341) 
  { 
     if (y >= 682){zone = 3;} 
     if (y <= 341){zone = 5;} 
     if ((y < 682) && (y > 341)) {zone = 4;}
  } 
  if (x >= 682) 
  { 
     if (y >= 682){zone = 1;} 
     if (y <= 341){zone = 7;} 
     if ((y < 682) && (y > 341)) {zone = 8;} 
  } 
  if ((x > 341) && (x < 682)) 
  { 
     if (y >= 682){zone = 2;} 
     if (y <= 341){zone = 6;} 
     if ((y < 682) && (y > 341)) {zone = 0;} 
  }
}


float calcul_intense(float x, float y) // FONCTION DE CALCUL D'INTENSITE (%)
{
  float X = x*(100.0/1023.0);
  float Y = y*(100.0/1023.0);
  float XM = 541.0*(100.0/1023.0);
  float YM = 513.0*(100.0/1023.0);
  float racine;
  racine = sqrt( pow((X - XM), 2) + pow((Y - YM), 2));
  float intense;
  intense = racine/60;
  if (intense > 1)
  {
    intense = 1.00;
  }
  if (y >= 1020)
  {
  intense = 1.00;
  }
  else if (y <= 3)
  {
  intense = 1.00;
  }
  if (x >= 1020)
  {
  intense = 1.00;
  }
  else if (x <= 3)
  {
  intense = 1.00;  
  }
  return intense;   
}

void loop() 
{ 
  // Lit l?entr?e analogique A0 
  valeur_v = analogRead(port_v); 
  // lit l?entr?e analogique A1 
  valeur_h = analogRead(port_h); 
  // calcul de zones
  calcul_zone(valeur_h, valeur_v);
  // calcul % de intensit? (de 0 ? 1);
  intense = calcul_intense(valeur_h, valeur_v);
  valeur = 225*intense;  
  Serial.print("\nVALEUR :"); 
  Serial.print(valeur);
  
  // acc?l?ration progressive : ancienne valeur + 5% valeur actuelle
  acceleration = intense_old+0.15*(valeur-intense_old);
  Serial.print("\nAncien");
  Serial.print(intense_old);
  //Definition de l'intervalle de valeurs
  if (acceleration > 150) // Valeur max (pour ne pas aller trop vite)
  {
    acceleration = 150;
  }
  if (acceleration < 100.00) //Valeur min (pour d?marrer directement)
  {
  acceleration = 100.00;
  }
  
  equilibrage=acceleration*0.45+60.5;
  
   /*
    DEBUT TEST VITESSE
*/
perimetre = pow((0.50*3.14), -3);   
etat = digitalRead(sensorPin);     
// est-ce qu'on peut encore continuer le test ?    
  if (millis()- chrono_depart < duree_test) {   
 // est-ce que l'etat de la Pin a chang? ?  
  if (etat != ancien_etat) {   
      nb_chgt = nb_chgt + 1 ;   
    //Serial.println("\n####################\n");   
    //Serial.println  (nb_chgt);  
      ancien_etat = etat;       
     }  
  } else {      
 // On est arrive au bout du test, on affiche le resultat
 // petite ruse : il y a 4 changements d'?tat par tour 
 // avec le type de montage de l'aimant choisi
 // en faisant un test sur 15 secondes le nb de chgt
 // a la meme valeur que le nombre de tours par minute 
  mesure_vitesse = nb_chgt*4;
  Serial.print  ("\n\nVitesse : ");
  Serial.print (mesure_vitesse); 
  Serial.println ("\n tr/min"); 
  mesure_vitesse = mesure_vitesse*perimetre*60;
  Serial.print  ("\n\nVitesse : ");
  Serial.print (mesure_vitesse); 
  Serial.println ("\n km/h"); 
  analogWrite( KMH, mesure_vitesse ); // on envoie la valeur en PWM ? l'automate
   // on repart pour un autre test   
  //delay(2000);
  Serial.println ("En cours de mesure ...");  
  chrono_depart=millis();
  nb_chgt = 0;
    
  }
/*
     FIN TEST VITESSE
   */
   
  Serial.print("\nacceleration :"); 
  Serial.print(acceleration);
  Serial.print("\nX:"); 
  Serial.print(valeur_h); 
  Serial.print(" Y:"); 
  Serial.println(valeur_v); 
  //delay(500);
  switch (zone) 
  { 
    delay(10);
    case 0: 
      Serial.println(" stop");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH);
      analogWrite( RoueDroite, 0);
      analogWrite( RoueGauche, 0); 
      break; 
    case 1: 
      Serial.println(" avance et tourne ? gauche");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH); 
      analogWrite( RoueDroite, equilibrage );
      analogWrite( RoueGauche, acceleration/1.1 );     
      break; 
    case 2: 
      Serial.println(" avance");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH);
      analogWrite( RoueDroite, equilibrage );
      analogWrite( RoueGauche, acceleration );   
      break; 
    case 3: 
      Serial.println("avance et tourne ? droite");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH);
      analogWrite( RoueDroite, equilibrage/1.1 );
      analogWrite( RoueGauche, acceleration );    
      break; 
    case 4: 
      Serial.println(" tourne ? droite ");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH);
      analogWrite( RoueDroite, 0 );
      analogWrite( RoueGauche, acceleration );     
      break; 
    case 5: 
      Serial.println(" recule et tourne ? droite");
      /* CONTACTS A ACTIVER */
      digitalWrite(ArriereDroite, HIGH);
      digitalWrite(ArriereGauche, LOW);
      analogWrite( RoueDroite, acceleration/1.2 );
      analogWrite( RoueGauche, equilibrage/1.1 );  
      break; 
    case 6: 
      Serial.println(" recule");
      /* CONTACTS A ACTIVER */
      digitalWrite(ArriereDroite, HIGH);
      digitalWrite(ArriereGauche, LOW);
      analogWrite( RoueDroite, acceleration/1.1 );
      analogWrite( RoueGauche, equilibrage/1.1 );    
      break; 
    case 7: 
      Serial.println(" recule et tourne ? gauche");
      /* CONTACTS A ACTIVER */
      digitalWrite(ArriereDroite, HIGH);
      digitalWrite(ArriereGauche, LOW);
      analogWrite( RoueDroite, acceleration/1.1 );
      analogWrite( RoueGauche, equilibrage/1.2 );   
      break; 
    case 8:
      Serial.println("tourne ? gauche");
      digitalWrite(ArriereDroite, LOW);
      digitalWrite(ArriereGauche, HIGH);
      analogWrite( RoueDroite, acceleration );
      analogWrite( RoueGauche, 0 );
      break;     
  }
  intense_old = acceleration;
  //delay(250); 
}
