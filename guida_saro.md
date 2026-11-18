# 🤖 Ciao! Benvenuto nel mondo di SARO!

**SARO** è il tuo robot personale che puoi programmare usando comandi facilissimi in italiano. Niente formule complicate: scrivi le azioni in ordine e guarda SARO prender vita!

---

## ⚡ Come iniziare un programma

Ogni volta che scrivi un programma per SARO, usa questa struttura magica all'inizio:

```cpp
#include <Saro.h>

inizia() {
  // Scrivi qui sotto quello che deve fare SARO!
}
```

*Vuoi che il robot ripete le azioni all'infinito? Sostituisci `inizia()` con `ripeti()`!*

---

## 🎮 Tutti i comandi che puoi usare

Tutti i comandi funzionano sia da soli (con un tempo predefinito) sia inserendo un numero tra parentesi (i **secondi**) se vuoi far durare il movimento di più o di meno!

### 🚗 Muovere le ruote
| Cosa vuoi fare? | Come si scrive? | Esempio con i secondi |
| :--- | :--- | :--- |
| **Andare avanti** | `avanti();` | `avanti(2);` (per 2 secondi) |
| **Andare indietro** | `indietro();` | `indietro(1.5);` |
| **Fare un piccolo passo** | `passo();` | `passo(0.8);` |
| **Girare a destra** | `destra();` | `destra(1);` |
| **Girare a sinistra** | `sinistra();` | `sinistra(0.5);` |
| **Fermarsi** | `ferma();` o `stop();` | — |

---

### 🦾 Muovere il braccio e la pinza
| Cosa vuoi fare? | Come si scrive? | Esempio con i secondi |
| :--- | :--- | :--- |
| **Alzare il braccio** | `su();` | `su(2);` |
| **Abbassare il braccio** | `giu();` | `giu(1.5);` |
| **Chiudere la pinza** (prendi) | `prendi();` | `prendi(1.2);` |
| **Aprire la pinza** (lascia) | `lascia();` | `lascia(1.2);` |

---

### ⏱️ Altri comandi utili
* **Aspettare un po':** `aspetta(2);` (aspetta 2 secondi prima di passare al comando successivo).

---

## 🚀 Il tuo primo programma: La Missione!

Ecco un esempio completo di programma in cui SARO si muove, abbassa il braccio, prende un oggetto, lo solleva e va avanti:

```cpp
#include <Saro.h>

inizia() {
  aspetta(2);      // Aspetta 2 secondi prima di partire
  
  lascia();        // Apre la pinza
  giu();           // Abbassa il braccio
  
  avanti(2);       // Va avanti per 2 secondi verso l'oggetto
  prendi(1.5);     // Chiude la pinza per afferrarlo
  
  su(1.5);         // Solleva il braccio
  avanti(3);       // Porta l'oggetto in avanti
  
  ferma();         // Fine della missione!
}
```

---

## 🛠️ Collegamenti (per i grandi / tutor)
Se devi controllare i cavi del robot, ricorda lo schema corretto:
* **Braccio (Servo 8135MG):** Pin **D3**
* **Pinza (Motore su Driver L298N):** Pin **D9 e D10**
* **Ruote (Driver L298N):** Pin **D2, D4, D5, D6**
* **Alimentazione:** Usa sempre una batteria dedicata per i motori e collega tutte le masse (GND) insieme!

Buon divertimento con la programmazione di SARO! 🚀