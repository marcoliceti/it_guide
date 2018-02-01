# ECMAScript 6

## Sommario

Lo scopo di questa guida è quello di presentare in maniera chiara e breve alcune
delle funzionalità più importanti introdotte con ECMAScript 6. In particolare il
focus è sulle funzionalità di uso più comune e non saranno invece trattate
quelle comunque importanti ma più pertinenti allo sviluppo di librerie e
framework.

Questa guida è inoltre rivolta a lettori che abbiano già esperienza di sviluppo
JavaScript. La parte sulle classi presuppone inoltre una conoscenza basilare
della programmazione orientata agli oggetti basata su classi (costruttori,
ereditarietà, overriding, metodi statici, ecc.). La parte sulle _promesse_,
infine, presuppone appunto la conoscenza di tale pattern.

La maggioranza delle funzionalità di ECMAScript 6 possono già essere provate
sulla maggioranza dei browser (o su [Node](https://nodejs.org/)) in ultima
versione. In alternativa è possibile utilizzare [Babel](https://babeljs.io/).

## Indice

* [Block scope](#block-scope)
* [Template strings](#template-strings)
* [Arrow functions](#arrow-functions)
* [Symbols](#symbols)
* [Iterables](#iterables)
* [Generators](#generators)
* [Arrow functions](#arrow-functions)
* [Classi](#classi)
* [Promesse](#promesse)
* [Miglioramenti agli object literal](#miglioramenti-agli-object-literal)
* [Default per i parametri di funzione](#default-per-i-parametri-di-funzione)
* [Arrow functions](#arrow-functions)
* [Spread operator](#spread-operator)
* [Destructuring](#destructuring)
* [Insiemi e mappe](#insiemi-e-mappe)
* [Moduli](#moduli)

## Block scope

Oltre che con `var` è ora possibile dichiarare variabili con `let` e `const`. In
entrambi i casi si otterrà una variabile visibile solo nel blocco di
dichiarazione. Nel caso di `const`, inoltre:

* è obbligatorio inizializzare subito la variabile
* non sarà possibile cambiarne il valore in seguito

Esempi:

```
let x = 7;
x = 8; // OK

const y; // Errore (non inizializzata)

const z = 9; // OK
z = 10; // Errore (già inizializzata)

if (false) {
  let h = 11;
  const j = 12;
} else {
  console.log(h); // Errore (dichiarata in un altro blocco)
  console.log(j); // Errore (dichiarata in un altro blocco)
}
```

Un'ulteriore interessante caratteristica delle variabili `let` e `const` e che
sono immuni
all'_[hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting=)_.

È bene infine ricordare che assegnare ad una variabile dichiarata con `const` il
riferimento ad un oggetto non è affatto sufficiente a rendere tale oggetto
immutabile:

```
const marco = { nome: 'Marco' };
marco.nome = 'Mario';
```

## Template strings

È possibile definire stringhe utilizzando il carattere \` come delimitatore:

```
const nome = `Marco`;
```

Stringhe definite in questo modo possono essere distribuite su più righe senza
dover utilizzare continuamente il carattere speciale `\n`:

```
const messaggio = `Tanto
va la gatta
al lardo, che
ci lascia lo
zampino`;
```

La peculiarità più importante, però, è la possibilità di _interpolare_ dei
valori all'interno della stringa:

```
const nome = 'Marco';
const eta = 30;
const messaggio = `Ciao, mi chiamo ${nome} e ho ${eta} anni`;
console.log(messaggio); 'Ciao, mi chiamo Marco e ho 30 anni'
```

## Arrow functions

Si tratta di una nuova sintassi semplificata per la definizione di funzioni
anonime. Tale sintassi è in realtà disponibile in diverse varianti:

```
[ 1, 2, 3 ].map(x => 2 * x); // [ 2, 4, 6 ]
[ 1, 2, 3 ].map((x, i) => i + ': ' + x); // [ '0: 1', '1: 2', '2: 3' ]
[ 1, 2, 3 ].map((x, i) => {
  return i + ': ' + x;
}); // [ '0: 1', '1: 2', '2: 3' ]
...
```

In pratica:

* con più parametri, occorre racchiuderli fra parentesi tonde
* con zero parametri, si usano delle tonde vuote
* nel caso di istruzioni su più righe occorre usare le graffe per delimitare il
corpo della funzione
* nel caso di funzioni composte da una sola istruzione, il `return` è implicito

Da notare inoltre che le funzioni definite in questo modo _catturano_ `this` dal
contesto in cui vengono definite, come se fosse una qualunque altra variabile e
non occorre il classico `var self = this;`.

**Nota:** Lo stesso vale per la variabile speciale `arguments`.

## Symbols

La nuova funzione globale `Symbol` permette di creare valori:

* con garanzia di univocità (`Symbol() !== Symbol()`)
* con tipo `symbol` (che entra a far parte dell'insieme dei tipi primitivi di
JavaScript)
* utilizzabili come chiavi di proprietà
* eventualmente corredati da una stringa descrittiva (`Symbol('pippo')`), che
però non impatta sull'univocità (`Symbol('pippo') !== Symbol('pippo')`)

Lo scopo è quello di evitare sovrascritture involontarie delle proprietà degli
oggetti: se usi una stringa rischi di scegliere un nome già utilizzato, mentre
con i simboli questo non potrà mai succedere.

Esempio:

```
const nome = Symbol('nome');
const marco = {
  [nome]: 'Marco'
};
marco[nome] = 'Mario'; // OK
marco.nome = 'Mirko'; // OK (ma non è una sovrascrittura, infatti...)
console.log(marco[nome]); // 'Mario'
const nome2 = Symbol('nome');
marco[nome2] = 'Mark'; // OK (ma nemmeno questa è una sovrascrittura...)
console.log(marco[nome]); // 'Mario'
```

## Iterables

È ora possibile iterare su un oggetto sulla base di una logica definita
dall'oggetto stesso. In particolare:

* l'oggetto deve avere una proprietà con chiave `Symbol.iterator`, che è un
simbolo globale predefinito
* il valore di tale proprietà deve essere una funzione
* tale funzione deve a sua volta restituire un oggetto con un metodo `next`
* il metodo `next` deve restituire oggetti con due proprietà `value` e `done`
che permettono di ottenere il prossimo valore dell'iterazione e di sapere se è
l'ultimo

Quando queste condizioni sono rispettate, è possibile usare una variante del
`for` che l'interprete JavaScript implementa utilizzando la funzione `next`
appena descritta.

Esempio:

```
const numeriCasuali = {
  [Symbol.iterator]: function () {
    return {
      next: function () {
        return {
          value: Math.random(),
          done: Math.random() > 0.9
        }
      }
    };
  }
};
for (let x of numeriCasuali) {
  console.log(x);
}
```

Alcune note terminologiche:

* oggetti iterabili in questa maniera vengono detti _iterable_
* _iterator_ è invece il nome dell'oggetto che implementa il metodo `next`

## Generators

Le _generator functions_ (o _generators_) sono funzioni che:

* sono contrassegnate da un `*`
* possono sospendere temporeaneamente la propria esecuzione tramite parola
chiave `yield`
* a ogni sospensione, possono "emettere" un valore affiancandolo alla parola
chiave `yield`, un po' come una normale funzione può restituire un valore
tramite parola chiave `return`
* quando vengono invocate non vengono realmente eseguite ma piuttosto
restituiscono un oggetto iterator
* l'esecuzione effettiva avviene ogni volta che viene invocato il metodo `next`
dell'iterator
* a ogni invocazione di `next`, la proprietà `value` riporta il valore emesso
dall'ultimo utilizzo della parola chiave `yield`

Le generator functions sono il costrutto ideale per la definizione di oggetti
iterabili:

```
const numeriCasuali = {
  [Symbol.iterator]: function* () {
    while (Math.random() < 0.9) yield Math.random();
  }
};
for (let x of numeriCasuali) {
  console.log(x);
}
```

## Classi

ECMAScript 6 introduce la parola chiave `class` per la creazione di prototipi
secondo una sintassi simile a quella per le classi negli altri linguaggi di
programmazione orientati agli oggetti:

```
class Persona {
  constructor(nome) {
    this.nome = nome;
  }
  presentati() {
    console.log(`Ciao sono ${this.nome}`);
  }
}

const marco = new Persona('Marco');
marco.presentati(); // 'Ciao sono Marco'
console.log(marco instanceof Persona); // true
Persona.prototype.diQualcosa = () => console.log('Qualcosa!');
marco.diQualcosa(); // 'Qualcosa!'
```

### Getter e setter

Mentre negli altri linguaggi i getter e i setter sono metodi pubblici per
l'accesso indiretto a membri privati (_incapsulamento_), in JavaScript sono
semplicemente dei metodi che possono essere invocati tramite la sintassi
normalmente utilizzata per l'accesso a proprietà.

Esempio:

```
class Persona {
  get nome() {
    console.log('Accesso in lettura a nome');
    return this._nome;
  }
  set nome(n) {
    console.log('Accesso in scrittura a nome');
    this._nome = n;
  }
}

const marco = new Persona();
marco.nome = 'Marco'; // 'Accesso in scrittura a nome'
console.log(marco.nome); // 'Accesso lettura a nome', 'Marco'
```

**Nota:** Prefissare le variabili d'istanza con un underscore è solo una
convenzione per indicare che non andrebbero accedute direttamente e non ha alcun
effetto sulla loro accessibilità (le proprietà di un oggetto JavaScript sono
sempre accessibili).

### Metodi statici

Esempio:

```
class PersoneHelper {
  static omonimi(p1, p2) {
    return p1.nome === p2.nome;
  }
}

const marco = new Persona('Marco');
const fabio = new Persona('Fabio');
PersoneHelper.omonimi(marco, fabio); // false
```

### Ereditarietà e overriding

Esempio:

```
class Persona {
  constructor(nome) {
    this.nome = nome;
  }
  presentati() {
    console.log(`Ciao sono ${this.nome}`);
  }
}

// Ereditarietà
class Lavoratore extends Persona {
  constructor(nome, mestiere) {
    super(nome);
    this.mestiere = mestiere;
  }
  // Overriding
  presentati() {
    console.log(`Ciao sono ${this.nome} e sono ${this.mestiere}`);
  }
}

const marco = new Lavoratore('Marco', 'sviluppatore');
marco.presentati(); // 'Ciao sono Marco e sono sviluppatore'
console.log(marco instanceof Persona); // true
console.log(marco instanceof Lavoratore); // true
```

## Promesse

Le promesse sono un modo particolarmente agevole di interagire con API
asincrone. L'argomento è abbastanza ampio da meritare di essere trattato
separatamente. Per quanto riguarda questa guida, il punto è che ECMAScript 6
introduce il suppporto nativo alle promesse tramite il costruttore `Promise`.

Esempio:

```
// Creazione di promesse
function miaFunzioneAsincrona() {
  // L'operazione avrà successo o fallirà randomicamente fra 2 secondi
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) resolve();
      else reject();
    }, 2000);
  });
}

// Consumo di promesse
miaFunzioneAsincrona().then(() => {
  console.log('L\'operazione schedulata 2 secondi fa è terminata con successo');
}, () => {
  console.log('L\'operazione schedulata 2 secondi fa è fallita');
});
```

## Miglioramenti agli object literal

In ECMAScript 6 è possibile evitare di ripetere lo stesso identificatore quando
si assegna ad una proprietà il valore di una variabile con lo stesso nome della
proprietà:

```
const nome = 'Marco';
const marco = { nome }; // invece di { nome: nome }
```

È inoltre disponibile una nuova sintassi dedicata ai metodi:

```
const nome = 'Marco';
const marco = {
  nome,
  presentati() {
    console.log('Ciao sono ' + this.nome);
  }
};
```

Infine i nomi delle proprietà possono finalmente essere assegnati dinamicamente:

```
const nome = 'Marco';
const proprieta = 'nome';
const marco = {
  [proprieta]: nome
};
```

## Default per i parametri di funzione

```
function presentati(nome = 'Mario Rossi') {
  console.log('Ciao sono ' + nome);
}
presentati(); 'Ciao sono Mario Rossi';
presentati('Marco'); 'Ciao sono Marco';
```

È possibile avere più parametri con valori di default, così come è possibile
mescolare parametri con e senza default. In questi casi la logica di
assegnazione è più intuitiva di quanto si possa pensare:

```
function sommaConDefault(a, b = 1, c, d = 2) {
  console.log(a, b, c, d);
  return a + b + c + d;
}
sommaConDefault(1); // NaN (1 + 1 + undefined + 2)
sommaConDefault(1, 2); // NaN (1 + 2 + undefined + 2)
sommaConDefault(1, undefined, 1); // 5 (1 + 1 + 1 + 2)
sommaConDefault(1, undefined, 1, 10); // 13 (1 + 1 + 1 + 10)
```

## Spread operator

Lo spread operator `...` consente di trasformare un array in una sequenza di
valori:

Esempio:

```
let x = [ 3, 4 ];
let y = [ 7, 8 ];
console.log([ 1, 2, ...x, 5, 6, ...y ]); // [ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

Può essere applicato anche a stringhe per ottenere sequenze di caratteri:

```
console.log([ ...'ciao' ]); // [ 'c', 'i', 'a', 'o' ]
```

Infine, applicato all'ultimo parametro di una funzione ad N parametri, permette
di farne un array al quale vengono assegnati tutti gli argomenti dall'N-esimo
(incluso) in poi:

```
function esempio(primo, secondo, ...altri) {
  console.log(altri);
}
esempio(1, 2, 3, 4); // [ 3, 4 ]
```

## Destructuring

Funzionalità che permette di estrarre porzioni di un oggetto o di un array:

```
const { a, b } = { a: 'x', b: 'y', c: 'z' };
console.log(a); // 'x'
console.log(b); // 'y'

const [ , a, , , b, ] = [ 4, 8, 15, 16, 23, 42 ];
console.log(a); // 8
console.log(b); // 23
```

È possibile indicare dei valori di default, es.:

```
const [ a, b = 2 ] = [ 1 ];
console.log(a); // 1
console.log(b); // 2
```

Il destructuring può essere utilizzato anche nella firma di una funzione per
estrarre automaticamente porzioni degli argomenti:

```
function presentati({ nome }) {
  console.log(`Ciao sono ${nome}`);
}
presentati({ nome: 'Marco' }); // Ciao sono Marco
```

È possibile estrarre proprietà annidate:

```
const { indirizzo: { citta } } = {
  nome: 'Marco',
  indirizzo: {
    citta: 'Roma'
  }
};
console.log(citta); // 'Roma'
```

È possibile scegliere il nome della variabile che ospiterà il valore estratto:

```
const { indirizzo: { citta: cittaDiMarco } } = {
  nome: 'Marco',
  indirizzo: {
    citta: 'Roma'
  }
};
console.log(cittaDiMarco); // 'Roma'
```

## Insiemi e mappe

Sono stati introdotti i costruttori `Set` e `Map` per la rappresentazione di
insiemi e mappe:

```
const paperi = new Set();
paperi.add('Qui');
paperi.add('Qui');
paperi.add('Quo');
paperi.add('Quo');
paperi.add('Qua');
paperi.add('Qua');
console.log(paperi.size); // 3
for (let papero of paperi) console.log(papero);

const coloriSquadre = new Map();
coloriSquadre.set('inter', 'nerazzurro');
coloriSquadre.set('roma', 'giallorosso');
for (let coppia of coloriSquadre) {
  const squadra = coppia[0];
  const colore = coppia[1];
  // ...
}
```

Esistono delle varianti `WeakSet` e `WeakMap` che permettono rispettivamente di
memorizzare oggetti e di usare oggetti come chiavi ma senza influire sulla
candidatura alla cancellazione di questi oggetti da parte del garbage collector.

## Moduli

In ECMAScript 6 è possibile separare il codice in più moduli, intesi come file
separati in cui tutte le definizioni sono di default private ma possono essere
condivise qualora esplicitamente _esportate_ dal file a cui appartengono ed
_importate_ nel file di destinazione. Allo scopo, sono state introdotte le
parole chiave `import` ed `export`.

Esempio di esportazione:

```
// File 'mio-modulo.js'
export const importantissimoValore = 42;
```

Esempio di importazione:

```
// File 'altro-mio-modulo.js' (stessa cartella)
import { importantissimoValore } from './mio-modulo.js';

console.log(importantissimoValore); // 42
```

Altro esempio di importazione:

```
// File 'altro-mio-modulo.js' (stessa cartella)
import * as mioModulo from './mio-modulo.js';

console.log(mioModulo.importantissimoValore); // 42
```

**Nota:** Questa seconda sintassi è preferibile quando il numero di definizioni
da importare è elevato.

**Nota:** Esiste anche altra sintassi dedicata a importazione ed esportazione di
definizioni, ma trattarle tutte quante servirebbe solo a fare confusione ;)
