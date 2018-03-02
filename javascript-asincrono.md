# JavaScript asincrono

## Sommario

Nelle due principali implementazioni di JavaScript, i browser e
[Node.js](https://nodejs.org), le operazioni di input / output (I/O) sono
offerte attraverso API asincrone. Per molti sviluppatori il modo di lavorare che
ne consegue risulta poco intuitivo e/o poco pratico, ma in realtà ha i suoi
vantaggi e può essere reso decisamente più agevole attraverso l'uso di appositi
pattern e di sintassi dedicata.

In questa guida introdurrò brevemente le motivazioni dietro l'uso di API
asincrone in JavaScript, il modo classico di utilizzarle ed alcune "tecniche
avanzate".

## Indice

* [Background](#background)
* [Callback](#callback)
* [Promesse](#promesse)
* [Generator](#generator)
* [Async await](#async-await)

## Background

Prima di poter lavorare efficacemente con API asincrone è ovviamente necessario
capire cosa sono e a che servono.

Un'API asincrona è essenzialmente un meccanismo per richiedere operazioni e
specificare come reagire al loro completamento, lasciando però il thread
chiamante libero di proseguire mentre tali operazioni vengono svolte in
parallelo. Le API asincrone possono essere quindi implementate mediante thread
ausiliari. Nel caso di operazioni di I/O (che poi sono quelle per le quali le
API asincrone vengono impiegate più spesso) esistono anche meccanismi più
efficienti che richiedono però un supporto da parte del sistema operativo
all'I/O non bloccante.

Ma a cosa serve avere API asincrone nel browser e in Node.js? Per capirlo
occorre partire dal fatto che il browser e Node.js sono essenzialmente dei
gestori di eventi, come ad esempio il click su un bottone (browser) o la
ricezione di una richiesta HTTP (Node.js). La gestione di questi eventi avviene
eseguendo il codice JavaScript specificato dalla pagina (o dal programma nel
caso di Node.js). Un aspetto cruciale è che **la gestione degli eventi viene
sempre effettuata utilizzando solo un thread dunque gli eventi vengono gestiti
uno per volta**. In quest'ottica le API asincrone servono ad avviare operazioni
anche molto lunghe senza impattare sulla durata della gestione di un evento
corrente e, quindi, senza ritardare troppo la gestione del prossimo evento.
Senza API asincrone il browser non potrebbe rimanere al passo con le azioni
dell'utente e Node.js non potrebbe gestire richieste concorrenti.

Per certi versi le API asincrone sono dunque il prezzo da pagare per l'uso di un
solo thread per la gestione di eventi. D'altro canto **l'utilizzo di un solo
thread comporta degli importanti benefici**: gestire un solo evento per volta
vuol dire che **la gestione di un evento non interferirà mai con quella di un
altro**. Inoltre, **nel caso di Node.js, non dover istanziare un thread per ogni
richiesta permette di una maggiore scalabilità ed una minore latenza**.

## Callback

Come anticipato, browser e Node.js funzionano ad eventi. Esistono meccanismi di
sottoscrizione che permettono di definire sotto forma di funzione, detta
"callback", il codice da eseguire per effettuare la gestione di un dato evento.
Ad esempio in una pagina web la definizione di come debba essere gestito il
click su un certo bottone può essere effettuata come segue:

```
bottone.addEventListener('click', function () {
  console.log('Pulsante premuto!');
});
```

Poiché il completamento di un operazione asincrona è considerato un evento (cioè
il browser e Node.js lo trattano come tale, ovvero eseguendo codice JavaScript
in risposta ad esso), anche le API asincrone sono basate sull'uso di callback:

```
fs.readFile('/home/marco/Desktop/esempio.txt', 'utf8', function () {
  console.log('Operazione completata!');
});
```

In questo esempio la funzione `readFile` del modulo `fs` di Node.js, che
permette di leggere un file, prende una callback come ultimo parametro (che per
altro è una convenzione piuttosto comune a fra le API asincrone JavaScript).

La callback passata ad un'API asincrona può avere a sua volta dei parametri che
rappresentano l'esito dell'operazione:

```
fs.readFile('/home/marco/Desktop/esempio.txt', 'utf8', function (errore, dati) {
  if (errore) {
    console.log('Ops: si è verificato un errore!');
  } else {
    console.log(dati);
  }
});
```

Convenzionalmete il primo parametro è quello che rappresenta l'eventuale errore
che ha fatto fallire l'operazione (se è fallita) mentre i parametri successivi
rappresentano gli eventuali valori restituiti dall'operazione in caso di
successo.

Nel lavorare con API asincrone bisogna sempre ricordare che JavaScript gestisce
un solo evento per volta e che perciò le callback non saranno mai eseguite prima
del termine della gestone dell'evento corrente. Ad esempio, supponiamo che
durante la gestione di un evento venga eseguito il seguente codice:

```
var errore = null;
var dati = null;
fs.readFile('/home/marco/Desktop/esempio.txt', 'utf8', function (e, d) {
  errore = e;
  dati = d;
});
if (errore) {
  console.log('Ops: si è verificato un errore!');
} else {
  console.log(dati);
}
```

Tale codice è errato e mostrerà sempre il messaggio di errore, anche se la
lettura del file avrà successo. Il motivo è semplice: la parte `if-else` che
determina la selezione del messaggio corretto, nonostante appaia per ultima nel
codice, viene eseguita prima della callback passata a `readFile`. Il codice
all'interno di tale callback, infatti, viene sì definito durante la gestione
dell'evento corrente, ma verrà eseguito durante la gestione di un altro evento,
ovvero il completamento della lettura del file.

Il codice che gestisce l'esito di un'operazione asincrona, quindi, va sempre
scritto tutto all'interno della rispettiva callback. Nel caso esso stesso
contenga chiamate asincrone si avrà quindi una callback nella callback e in
generale una sequenza di operazioni asincrone può assumere un aspetto simile al
seguente:

```
operazioneUno(function () {
  operazioneDue(function () {
    operazioneTre(function () {
      operazioneQuattro(function () {
        operazioneCinque(function () {
          operazioneSei(function () {
            operazioneSette(function () {
              // ecc.
            });
          });
        });
      });
    });
  });
});
```

Nella comunità JavaScript questo genere di struttura è ironicamente chiamata
"Pyramid of Doom" ed è evidente come possa diventare rapidamente difficile da
gestire in termini di indentazione e di scope. Quando poi si passa da una
semplice sequenza prefissata a flussi più complessi scrivere codice manutenibile
in questo modo diventa un'impresa, perciò sono emersi pattern specifici per la
semplificazione del codice asincrono e fra questi il pattern delle promesse è il
più popolare.

## Promesse

L'idea alla base delle promesse è relativamente semplice. Al momento di invocare
un'API asincrona, invece di passare una callback, ci si aspetta semmai di
ricevere un oggetto, chiamato appunto "promessa", dotato di un metodo `next` che
permette di specificare due distinte callback: una per gestire il successo
dell'operazione e l'altra per gestirne il fallimento.

Ad esempio un'operazione per la lettura di un file potrebbe avvenire in modo
simile al seguente:

```
var promessa = leggiFile('/home/marco/Desktop/esempio.txt', 'utf8');
promessa.then(function (dati) {
  console.log(dati);
}, function (errore) {
  console.log('Si è verificato un errore!');
});
```

Rispetto all'approccio "tradizionale", sono già evidenti due miglioramenti:

* la richiesta della promessa e la definizione delle callback sono indipendenti
* così come sono indipendenti la gestione del successo e quella del fallimento

Le promesse, poi, permettono di risolvere la "Pyramid of Doom" attraverso un
meccanismo di concatenazione basato su singola callback e in cui ogni callback
restituisce la promessa associata all'operazione asincrona successiva:

```
operazioneUno().then(function () {
  return operazioneDue();
}).then(function () {
  return operazioneTre();
}).then(function () {
  return operazioneQuattro();
}).then(function () {
  return operazioneCinque();
}).then(function () {
  return operazioneSei();
}).then(function () {
  return operazioneSette();
}); // ecc.
```

Se le callback si limitano a restituire una promessa, è anche possibile
utilizzare la seguente forma, ancora più concisa:

```
operazioneUno().
  then(operazioneDue).
  then(operazioneTre).
  then(operazioneQuattro).
  then(operazioneCinque).
  then(operazioneSei).
  then(operazioneSette); // ecc.
```

Il risultato della concatenazione di promesse è a sua volta una promessa
rappresentante l'intera sequenza e permette di gestire in maniera centralizzata
il fallimento di una qualunque delle operazioni nella sequenza:

```
operazioneUno().
  then(operazioneDue).
  then(operazioneTre).
  then(function () {
    console.log('Le tre operazioni hanno avuto successo');
  }, function () {
    console.log('Una delle tre operazioni è fallita');
  });
```

Una trattazione più esaustiva delle promesse richiederebbe una guida a parte, ma
gli esempi mostrati sono già sufficienti a muovere i primi passi con le
promesse. A [questo indirizzo](https://promisesaplus.com/) sono poi disponibili
delle specifiche complete che possono essere usate per capire meglio tutti gli
effettivi requisiti del pattern. Come si potrà facilmente verificare, si tratta
di un pattern in realtà piuttosto complesso, perciò invece che implementarlo
autonomamente gli sviluppatori JavaScript utilizzano librerie di supporto che
permettono di creare promesse conformi alle specifiche. Nella versione
ECMAScript 6 di JavaScript le funzioni offerte da queste librerie sono persino
diventate parte delle API standard delle linguaggio e dunque non è necessaria
alcuna libreria. Ecco quindi un esempio di come trasformare un'API asincrona
tradizionale in una basata su promesse utilizzando ECMAScript 6:

```
function apiConPromessa(x) {
  return new Promise(function (resolve, reject) {
    apiAsincronaTradizionale(x, function (errore, y) {
      if (errore) return reject(errore);
      resolve(y);
    });
  };
}
```

## Generator

Oltre al supporto alle promesse, ECMAScript 6 introduce i generator, ovvero
funzioni che possono essere sospese e riprese (anche più volte) prima di
terminare la propria esecuzione. Come vedremo nella
[prossima sezione](#async-await), i generator permettono di gestire il codice
asincrono in nuovi modi. In questa sezione, però, cerchiamo prima di capire il
loro funzionamento generale.

Come detto, un generator può sospendere la propria esecuzione. In particolare
ECMAScript 6 introduce a tal proposito la parola chiave `yield`:

```
function* esempioDiGenerator() {
  yield;
}
```

**Nota:** Per dichiarare una generator occorre aggiungere un asterisco
alla parola chiave `function`.

Uno dei modi in cui i generator si distinguono dalle normali funzioni è il fatto
che non è sufficiente invocarli per avviarne l'esecuzione. Piuttosto quando una
generator viene invocato, esso restituisce immediatamente un oggetto chiamato
"iterator" con un metodo `next` che permette finalmente di avviare l'esecuzione
del generator:

```
function* esempioDiGenerator() {
  console.log('Esecuzione iniziata');
  yield;
}
var iteratore = esempioDiGenerator();
console.log('Esecuzione non ancora iniziata');
iteratore.next(); // avvia l'esecuzione
```

Una volta avviata, l'esecuzione prosegue fino a quando non viene incontrata la
parola chiave `yield`:

```
function* esempioDiGenerator() {
  console.log('Questo messaggio sarà mostrato');
  yield;
  console.log('Questo no');
}
var iteratore = esempioDiGenerator();
iteratore.next();
```

Chiamando nuovamente `next` sarà possibile riprendere l'eseuzione del generator:

```
function* esempioDiGenerator() {
  console.log('Questo messaggio sarà mostrato');
  yield;
  console.log('E anche questo');
}
var iteratore = esempioDiGenerator();
iteratore.next();
iteratore.next();
```

Il metodo `next` e `yield` permettono quindi di alternare l'esecuzione della
funzione chiamante e quella del generator:

```
function* esempioDiGenerator() {
  console.log('b');
  yield;
  console.log('d');
  yield;
  console.log('f');
  yield;
  // ecc.
}
var iteratore = esempioDiGenerator();
console.log('a');
iteratore.next();
console.log('c');
iteratore.next();
console.log('e');
iteratore.next();
console.log('g');
// ecc.
```

Inoltre `next` e `yield` permettono di scambiare valori fra le due funzioni. In
particolare, `next` restituisce ogggetti basati sui valori inviati da `yield`:

```
function* esempioDiGenerator() {
  yield 1;
  yield 2;
  yield 3;
}
var iteratore = esempioDiGenerator();
var n = iteratore.next();
console.log(n); // { value: 1, done: false }
n = iteratore.next();
console.log(n); // { value: 2, done: false }
n = iteratore.next();
console.log(n); // { value: 3, done: false }
n = iteratore.next();
console.log(n); // { value: undefined, done: true }
```
**Nota:** La proprietà `done` permette di sapere se l'esecuzione del generator
è terminata.

A sua volta `yield` fornisce accesso agli eventuali valori passati tramite
`next`:

```
function* esempioDiGenerator() {
  var x;
  x = yield;
  console.log(x);
  x = yield;
  console.log(x);
  x = yield;
  console.log(x);
}
var iteratore = esempioDiGenerator();
iteratore.next();
iteratore.next(1);
iteratore.next(2);
iteratore.next(3);
```

**Nota:** La prima invocazione di `next` non passa alcun valore. In altre
parole, il primo `yield` riceve il valore passato dal secondo `next`, il secondo
`yield` riceve dal terzo `next`, e così via.

L'iterator permette anche di provocare un'eccezione all'interno del
corrispondente generator:

```
function* esempioDiGenerator() {
  try {
    yield;
  } catch (errore) {
    console.log(errore);
  }
}
var iteratore = esempioDiGenerator();
iteratore.next();
iteratore.throw('finto errore');
```

## Async await

Ok: cosa c'entrano i generator con le API asincrone?

Per capirlo osserviamo anzitutto che una normale funzione JavaScript viene
interamente eseguita nel corso della gestione di un singolo evento, mentre
l'esecuzione di un generator può iniziare durante la gestione di un evento,
essere sospesa e poi riprendere durante la gestione di un altro evento: è
sufficiente che le callback che gestiscono i due eventi abbiano entrambe accesso
all'iterator che controlla il generator. Ad esempio, dati due bottoni su una
pagina web, il secondo bottone può far riprendere l'esecuzione di un generator
avviato dal primo bottone:

```
var iteratore;
b1.addEventListener('click', function () {
  iteratore = (function* () {
    console.log('esecuzione avviata da b1...');
    yield;
    console.log('...e ripresa da b2');
  })();
  iteratore.next();
});
b2.addEventListener('click', function () {
  iteratore.next();
});
```

I generator possono quindi essere utilizzati per scrivere codice in grado di
"attendere" il risultato di un'operazione asincrona: l'attesa inizia
all'interno del generator facendo lo `yield` di una promessa e termina quando
da una delle callback della promessa viene chiamato `next` o `throw`. Esempio:

```
function* example() {
  try {
    var result = yield makeMeAPromise();
    console.log(result);
  } catch (error) {
    console.log(error);
  }
}

var iterator = example();
var promise = iterator.next();
promise.then(function (result) {
  iterator.next(result);
}, function (error) {
  iterator.throw(error);
});
```

The code in the previous example may be refactored into this:

```
function magicFunction(aGenerator) {
  var iterator = aGenerator();
  var promise = iterator.next();
  promise.then(function (result) {
    iterator.next(result);
  }, function (error) {
    iterator.throw(error);
  });
}

magicFunction(function* () {
  try {
    var result = yield makeMeAPromise();
    console.log(result);
  } catch (error) {
    console.log(error);
  }
});
```

`magicFunction` can be extended in order to support waiting for multiple
promises:

```
magicFunction(function* () {
  try {
    var x = yield promiseOne();
    var y = yield promiseTwo();
    var z = yield promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
});
```

Now, if you look at the genarator's body you'll notice that it's starting to
look like plain old synchronous code. From the genarator's perspective there's
no difference between asynchronous values waited with `yield` and values
obtained through a synchronous function. In other words, you don't need anymore
callbacks or complex promise compositions: you just write "regular" code except
that you put `yield` before promises.

Of course this works only if you have a suitable `magicFunction`. You can find a
sample implementation [here](https://www.promisejs.org/generators/) and of
course you can make your own. In ECMAScript 2017 you won't need to do that: the
generator based pattern seen in this section is natively supported with
dedicated syntax. Specifically, you can declare `async` functions and use the
`await` keyword inside them:

```
async function example() {
  try {
    var x = await promiseOne();
    var y = await promiseTwo();
    var z = await promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
}
```

This will be internally translated into something like:

```
async(function* () {
  try {
    var x = yield promiseOne();
    var y = yield promiseTwo();
    var z = yield promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
});
```

Where `async` is a function similar to the `magicFunction` seen in the previous
examples.

This conclude our asynchronous JavaScript overview ;)
