# ESP8266-PNP-Sensor-counter

![alt text](https://github.com/MarcinanBarbarzynca/ESP8266-PNP-Sensor-counter/blob/main/Schematic_ESP8266%20%2B%20PNP%20sensor%200.4_2021-02-17(1).png)

## Co to jest?
To licznik elementów które pojawią się w pobliżu czujnika zbliżeniowego/laserowego i tym podobnych podłączonych pod wejście płyty. 
ESP8266 łączy się z siecią wifi i udostępnia dane do serwera przy użyciu HTML POST. 

Link do edytora. 2x
https://oshwlab.com/polaski/esp8266-pnp-sensor-0-4/  V4
https://oshwlab.com/polaski/esp8266-pnp-proximity-sensor_copy_copy_copy_copy V2


To pliki użyte do uruchomienia sterownika ESP8266 który odpowiada za liczenie elementów które opuszczają taśmę produkcyjną. 

1. Łączy się z siecią wifi przedsiębiorstwa
2. Zlicza zadziałania czujnika
3. Wrzuca dane do serwera poprzez HTML POST

## Algorytm działania:
1. Zainicjalizuj przerwanie na pinie do którego jest podłączony transoptor (nóżka D8 i D7 w zależności 
*Zliczaj kiedy czujnik zmienia stan na niski (lowering).
2. Zainicjalizuj połączenie wifi (łącz do ustalonej sieci wifi)
3. Nadaj klucz do skryptu key.php na serwerze i oczekuj na odpowiedź
4. Jeżeli klucz jest prawidłowy to nadaj licznik za pomocą HTML GET
5. Nadaj licznik za pomocą HTML GET. 

## Aby odtworzyć utracony kod wystarczy połączyć przykłady:
1. ESP8266 HTML Client
2. ESP8266 interrupts handling
3. ESP8266 WiFi client

## Prawidłowa funkcja działania:
Co 10 sekund mryga niebieski led na sterowniku. --> Jest to sygnalizacja o poprawnym wysłaniu danych do serwera.
Za każdym razem 

## Słownictwo:
Płyta głowna --> Płyta z cyną robiona ręcznie

Płyta mikrokontrolera; ESP8266; sterownika --> Płyta brandowana czarna profesjonalna. 

## Dotychczasowe usterki:
1. Uszkodzenie transoptora 
* Transoptor posiada 4 nóżki. Pierwsza para odpowiada ma diodę led a druga fototranzystor. Jeżeli uszkodzeniu uległ tranzystor to sygnał ujemny można uzyskać poprzez zwarcie tych nóżek z tranzystorem. Sprawdź multimetrem czy element jest sprawny (woltomierz wskazuje 19V przed ledem transoptora i 0V za ledem transoptora. Po załączeniu czujnika powinien pokazywać 0,7V na obu nóżkach --> jest to spadek napięcia na diodzie). Następnie sprawdź wyjście transoptora. Pojawia się na tych nóżkach 3,3V i 0V. (Po załączeniu na obu pojawia się 0,3V, jest to spadek napięcia na sprawnym tranzystorze.) 
2. Uszkodzenie mikrokontrolera
* Na złączu VIN i GND jest zwarcie. Oznacza to że usmarzył się mikrokontroler. Płytka jest do wyrzucenia(nie da się naprawić mikrokontrolera). Nalerzy sprawdzić napięcie wyjściowe na przetwornicy i wyregulować je do 5V. Ewentualnie wymienić przetwornicę. Następnie wgraj program przygotowany na dany sterownik ESP8266. Do tego potrzebujesz ID sterownika (jest ono napisane w środku puszki). Następnie określ adres serewera i uzupełnij te dane w kodzie programu który wgrywasz przez Arduino IDE. Program jest tu w tym repozysorium na Githubie. 
3. Nie mryga sterownik co 10sekund. 
* Sterownik nie może połączyć się z serwerem lub ma opuźnienia w nadaniu danych.
* Po reakji czujnika nie pojawia się reakcja na płycie głównej (jeżeli są 2 ledy dodatkowe). --> Rozwalił się transoptor. 
* Mimo zliczania nie pojawia się więcej niż 0 na serwerze --> Oznacza to że płyta główna nie przepuszcza sygnału z czujnika do sterownika.
* Przemyj płytkę alkoholem aby wyeliminować zwarcia od kurzu i brudu. 
4. Sterownik nalicza za dużo elementów wyjściowych. 
* Wstawiłem debounce do kodu w funkcji przerwania. Zapoznaj się z szybkością urządzenia a potem wprowadź obliczoną wartość czasu debouncu
* Dla urządzenia o wydajności maxymalnej 60 szt na minutę można dać x<1000ms debouncu. Pomyśl jaki debounce będzie optymalny i wstaw nowy. 
5. Sterownik świeci lampką sygnalizującą zadziałanie czujnika PNP cały czas, nie reaguje na inne bodźce
* Po włączeniu zasilania (wymianie bezpiecznika/uruchomieniu ponownym maszyny) może dojść do zawieszania. Aby go odwiesić wystrczy zresetować sterownik. Efekt teni nie występuje za często.
* Wprowadzę jeszcze watchdoga który resetuje samoczynnie ten efekt... jednak jest to niebezpieczne, bo może kasować rejestry podręczne. 
  
Jakby coś zawsze możecie napisać 10 mejli na p_ir@o2.pl. Im więcej mejli tym większe prawdopodobieństwo że zauważę. 
  
