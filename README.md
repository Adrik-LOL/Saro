# Tutorial SARO v2.6.0 — Arduino UNO

Questa guida descrive l'uso della libreria **Saro-Base v2.6.0** con Arduino UNO, un motore DC per la pinza comandato da L298N e un servo a rotazione continua 360° per il braccio.

> **Nota di sicurezza:** durante le prime prove tieni il robot sollevato o le ruote libere, lascia la pinza senza oggetti e verifica che il braccio non possa urtare persone o componenti. Non alimentare motori o servo dal pin 5V di Arduino se richiedono più corrente di quella disponibile.

## 1. Installazione

1. Scarica `Saro-Base-v2.6.0-UNO-fix.zip`.
2. Nell'IDE Arduino rimuovi o sposta le vecchie copie della libreria `Saro` dalla cartella `libraries`, per evitare che l'IDE carichi una versione precedente.
3. Apri **Sketch → Include Library → Add .ZIP Library…**.
4. Seleziona il file ZIP della versione 2.6.0.
5. Seleziona **Arduino Uno** in **Strumenti → Scheda**.
6. Verifica che la libreria standard **Servo** sia disponibile: la libreria SARO la include.

Se l'IDE segnala che `Saro.h` non esiste, controlla che non ci siano due cartelle annidate `Saro/Saro/` e che la cartella installata contenga direttamente `Saro.h`, `Saro.cpp` e `library.properties`.

## 2. Collegamenti previsti dalla libreria

I pin sono definiti nel file `Saro.h`. Prima di alimentare il robot, verifica che il cablaggio reale corrisponda a questa tabella.

| Funzione | Pin Arduino UNO |
|---|---:|
| Ruota / driver ingresso 1 | D2 |
| Ruota / driver ingresso 2 | D4 |
| Ruota / driver ingresso 3 | D5 |
| Ruota / driver ingresso 4 | D6 |
| Pinza L298N ingresso IN3 | D9 |
| Pinza L298N ingresso IN4 | D10 |
| Segnale servo continuo del braccio | D3 |

La libreria presume che D9 e D10 comandino i due ingressi direzionali del canale L298N usato dalla pinza. L'apertura è impostata come **D9 HIGH e D10 LOW**; la chiusura usa la combinazione opposta.

**Alimentazione e masse**
- Alimenta motori e servo con alimentatori adeguati alle rispettive specifiche.
- Collega il GND dell'alimentazione motori, il GND dell'alimentazione servo e il GND di Arduino in comune.
- Non collegare insieme uscite positive di alimentatori diversi.
- Verifica la tensione ammessa dal servo e dal driver prima di collegarli.

## 3. Struttura di un programma SARO

La libreria fornisce due macro:

- `inizia() { ... }`: il blocco viene eseguito una sola volta, come il contenuto di `setup()`.
- `ripeti() { ... }`: il blocco viene eseguito continuamente, come il contenuto di `loop()`.

Per una sequenza di test da eseguire una volta, usa `inizia()`:

```cpp
#include <Saro.h>

inizia() {
  avanti(1);
  aspetta(1);
  fermaTutto();
}
```

Non inserire una sequenza completa dentro `ripeti()` se non vuoi che ricominci continuamente.

## 4. Comandi disponibili

I tempi sono espressi in **secondi**.

### Ruote

```cpp
avanti(1);       // Avanza per 1 secondo
indietro(1);     // Indietreggia per 1 secondo
destra(0.5);     // Ruota a destra per mezzo secondo
sinistra(0.5);   // Ruota a sinistra per mezzo secondo
passo();         // Avanza per il tempo predefinito (0,8 s)
ferma();         // Arresta le ruote
```

Le direzioni effettive dipendono dal collegamento dei motori e dalle impostazioni `SARO_INVERTI_SINISTRA` e `SARO_INVERTI_DESTRA` in `Saro.h`.

### Pinza

```cpp
apriPinza(1);    // Apre per 1 secondo
chiudiPinza(1);  // Chiude per 1 secondo

lascia(1);       // Alias di apriPinza()
prendi(1);       // Alias di chiudiPinza()

fermaPinza();    // Ferma il motore della pinza
```

Prima di invertire la direzione la libreria porta entrambi gli ingressi a LOW e attende il tempo di sicurezza definito da `SARO_PINZA_TEMPO_INVERSIONE_MS` (predefinito 40 ms).

Il tempo non misura la posizione della pinza: senza finecorsa o sensore, l'apertura effettiva dipende dal motore, dal carico e dalla meccanica. Non lasciare il motore in stallo contro un arresto meccanico.

### Braccio con servo continuo 360°

```cpp
su(1.2);          // Ruota nel verso configurato come "su"
giu(1.2);         // Ruota nel verso configurato come "giu"
fermaBraccio();   // Invia il comando di neutro
```

Un servo a rotazione continua non imposta un angolo: il segnale determina direzione e velocità di rotazione. La posizione raggiunta dipende dal tempo, dal carico e dalla velocità effettiva. La libreria non conosce la posizione del braccio.

### Attese e arresto

```cpp
aspetta(2);       // Attende 2 secondi
ferma();          // Ferma le ruote
fermaPinza();     // Ferma la pinza
fermaBraccio();   // Invia il neutro al servo
fermaTutto();     // Esegue tutti e tre gli arresti
```

I comandi temporizzati sono **bloccanti**: mentre `avanti(1)` o `su(1.2)` è in esecuzione, lo sketch attende il termine del tempo e non esegue le righe successive.

## 5. Esempio: test completo

Questo esempio esegue una sequenza una sola volta. Effettua la prima prova con il robot in condizioni sicure e con tempi brevi.

```cpp
#include <Saro.h>

inizia() {
  aspetta(2);

  // Ruote
  avanti(1);
  indietro(1);
  passo();
  destra(0.5);
  sinistra(0.5);

  // Pinza
  lascia(1);
  prendi(1);

  // Braccio
  su(1.2);
  giu(1.2);

  // Arresto generale
  fermaTutto();
}
```

Se questa sequenza parte più volte, controlla che:
1. non sia stata copiata dentro `ripeti()`;
2. non ci siano altre chiamate alle stesse funzioni nello sketch;
3. non sia installata una vecchia copia della libreria;
4. la scheda non si stia riavviando per un'alimentazione insufficiente o disturbi elettrici.

## 6. Calibrare il neutro del servo 360°

I valori iniziali nella libreria sono:
- neutro/stop: 1500 µs;
- verso `su`: 2000 µs;
- verso `giu`: 1000 µs.

Questi sono valori iniziali, non una calibrazione garantita per ogni servo. Il punto neutro reale può essere leggermente diverso.

La libreria consente di impostare tre impulsi:

```cpp
saro.impostaImpulsiServo(1500, 2000, 1000);
```

L'ordine dei parametri è:

```cpp
saro.impostaImpulsiServo(neutroUs, impulsoSuUs, impulsoGiuUs);
```

I valori devono essere compresi tra 500 e 2500 µs e devono essere distinti. La funzione restituisce `true` se accetta i valori e `false` se li rifiuta.

Per calibrare il neutro:
1. Scollega il carico meccanico dal servo, se possibile.
2. Prova un valore di neutro vicino a 1500 µs.
3. Se il servo continua a ruotare, spegni e modifica il valore di poco.
4. Ripeti fino a trovare il valore in cui il servo resta fermo.
5. Usa quel valore come primo parametro di `impostaImpulsiServo()`.

Non effettuare la calibrazione con il braccio che può urtare persone, cavi o parti del robot.

## 7. Arduino UNO: attenzione a Timer1

La libreria `Servo` usa **Timer1** su Arduino UNO. In questa versione D9 e D10 sono usati come uscite digitali con `digitalWrite()`, non con PWM.

Mentre il servo è gestito dalla libreria `Servo`, evita di usare `analogWrite()` su D9 o D10. Evita anche altre librerie che richiedono Timer1, perché possono entrare in conflitto con la generazione degli impulsi del servo.

## 8. Risoluzione dei problemi

| Sintomo | Controlli da fare |
|---|---|
| La pinza scatta quando cambia direzione | Verifica cablaggio IN3/IN4, alimentazione, GND comune e che il motore non sia bloccato meccanicamente. |
| La pinza gira nel verso sbagliato | Verifica che D9/D10 siano collegati agli ingressi previsti. La configurazione iniziale presume che apertura = D9 HIGH, D10 LOW. |
| Il braccio non si muove | Verifica alimentazione del servo, GND comune, segnale su D3 e integrità dei collegamenti. |
| Il braccio continua a ruotare al comando di stop | Calibra il neutro del servo: 1500 µs potrebbe non essere il valore corretto per il tuo esemplare. |
| Il braccio si muove in un verso inatteso | Verifica quale impulso produce ciascun verso; se necessario scambia i valori `suUs` e `giuUs` nella calibrazione. |
| Le ruote girano al contrario | Controlla i collegamenti e le costanti `SARO_INVERTI_SINISTRA` / `SARO_INVERTI_DESTRA` in `Saro.h`. |
| La scheda si riavvia quando parte un motore | Sospetta cadute di tensione o disturbi: controlla alimentatori, cablaggio, masse e assorbimento. Non alimentare i motori dal 5V di Arduino. |
| La sequenza si ripete | Controlla l'uso di `ripeti()` e le eventuali altre chiamate nello sketch. `inizia()` è previsto per una sequenza singola. |

## 9. Limiti della versione

Questa libreria controlla i movimenti principalmente tramite durata:
- non legge encoder delle ruote;
- non misura l'angolo del braccio;
- non rileva automaticamente quando la pinza è completamente aperta o chiusa;
- non offre un arresto di emergenza hardware.

Per movimenti realmente ripetibili servono sensori, finecorsa o encoder, oltre alla calibrazione. Un arresto software non sostituisce un interruttore fisico di emergenza quando necessario.

## 10. Checklist prima dell'utilizzo

- [ ] È installata una sola copia di Saro, versione 2.6.0.
- [ ] La scheda selezionata è Arduino UNO.
- [ ] I pin del cablaggio corrispondono alla tabella.
- [ ] Le masse sono collegate in comune.
- [ ] Servo e motori hanno alimentazione adeguata.
- [ ] Il neutro del servo è stato verificato.
- [ ] La pinza non viene tenuta in stallo contro un arresto meccanico.
- [ ] La sequenza singola è in `inizia()`, non in `ripeti()`.
- [ ] La prima prova viene eseguita con movimenti brevi e senza carichi pericolosi.

---

**Nota di verifica:** questa guida descrive l'interfaccia e i pin presenti nel pacchetto SARO v2.6.0. La correttezza del cablaggio, la calibrazione del servo e il comportamento fisico del robot devono essere verificati sul dispositivo reale. La libreria non è stata collaudata elettricamente sul robot.