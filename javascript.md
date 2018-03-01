# JavaScript

## Sommario

JavaScript è probabilmente la migliore scelta possibile per chi è in cerca di un
linguaggio di programmazione da studiare: è semplice iniziare (basta un browser)
ed è estremamente utile da imparare in quanto permette di sviluppare
applicazioni web, applicazioni desktop ([Node.js](https://nodejs.org),
[Electron](https://electronjs.org/)) e applicazioni mobile
([Ionic](https://ionicframework.com/),
[React Native](https://facebook.github.io/react-native/)) senza dover utilizzare
altri linguaggi di programmazione.

In questa guida vengono spiegate brevemente le basi di JavaScript, assumendo che
il lettore abbia comunque esperienza di sviluppo in altri linguaggi e preferendo
esempi a spiegazioni esageratamente dettagliate.

**Nota:** Si consiglia di seguire la guida provando gli esempi. È sufficiente
[aprire la console del proprio browser](https://www.google.it/search?q=aprire+console+browser)
e copiare una a una le istruzioni, osservando come reagisce l'interprete
JavaScript collegato alla console.

## Indice

* [Variabili](#variabili)
* [Array](#array)
* [Selezione](#selezione)
* [Iterazione](#selezione)
* [Funzioni](#funzioni)
* [Oggetti](#oggetti)
* [Prototipi](#prototipi)

## Variabili

Si dichiarano con la parola chiave `var` e non è necessario (e nemmeno
possibile) specificarne il tipo:

```
var x;
```

I valori assegnabili alle variabili, però, hanno sempre un tipo. Quelli più
semplici sono numeri, stringhe e booleani:

```
var n = 7;
var s = 'hello';
var b = true;

typeof n; // 'number'
typeof s; // 'string'
typeof b; // 'boolean'
```

**Nota:** Non viene fatta distinzione fra numeri interi e numeri con la virgola.

**Nota:** Così come in molti altri linguaggi, i commenti possono essere inseriti
con un doppio slash (`//`) o delimitandoli con `/*` e `*/`.

Come in altri linguaggi, in JavaScript esiste il valore speciale `null`, che si
usa per rappresentare valori non validi. C'è poi un valore speciale `undefined`
che viene automaticamente assegnato alle variabili non inizializzate:

```
var x;
console.log(x); // undefined
```

**Nota:** `console.log` è una funzione per stampare a schermo il valore di
un'espressione.

## Array

JavaScript ha una sintassi estremamente semplice per la creazione di array:

```
var numeri = [ 1, 2, 3 ];
```

A differenza di altri linguaggi, JavaScript permette di memorizzare valori di
tipi diversi nello stesso array:

```
var caos = [ 1, 'ciao', false, null ];
```

La sintassi per l'accesso in lettura e in scrittura ai singoli elementi
dell'array è la stessa vista in molti altri linguaggi:

```
var paperi = [ 'qui', 'quo', 'qua' ];
console.log(paperi[0]); // 'qui'
console.log(paperi[1]); // 'quo'
console.log(paperi[2]); // 'qua'
paperi[0] = 'paperino';
console.log(paperi); // [ 'paperino', 'quo', 'qua' ]
```

Gli array JavaScript hanno tipo `object`, cioè sono considerati
[oggetti](#oggetti) e come tali hanno delle "proprietà". Fra queste, la
proprietà `length` permette di ottenere la lunghezza dell'array:

```
[ 'a', 'b', 'c' ].length; // 3
```

Gli array possono essere inoltre modificati utilizzando alcuni metodi appositi,
es.:

```
var lettere = [ 'a', 'b', 'c' ];
lettere.push('d');
console.log(lettere); // [ 'a', 'b', 'c', 'd' ]
lettere = lettere.concat([ 'e', 'f', 'g' ]);
console.log(lettere); // [ 'a', 'b', 'c', 'd', 'e', 'f', 'g' ];
lettere.pop();
console.log(lettere); // [ 'a', 'b', 'c', 'd', 'e', 'f' ];
...
```

**Nota:** Per una panoramica completa sui metodi degli array si rimanda ad una
bella ricerca Google ;)

## Selezione

Sintassi simile a quella degli altri linugaggi ispirati al C:

```
if (x > 5) {
  console.log('x è maggiore di 5');
} else {
  console.log('x è minore di o uguale a 5');
}
```

Nelle espressioni booleane utilizzate per controllare il flusso di esecuzione
è bene ricordare che in JavaScript esistono test di equivalenza (`==`) e test di
uguaglianza (`===`):

```
console.log('5' == 5); // true (equivalenti...)
console.log('5' === 5); // false (...ma non uguali)
```

**Nota:** Analogamente, esistono gli operatori `!=` e `!==`.

**Nota:** I test di equivalenza (o di non equivalenza) vanno usati con
particolare cautela. Il rischio è quello di "portare in giro" nel codice valori
che si pensa essere di un tipo mentre invece sono di un altro.

In JavaScript `null`, `undefined` e le stringhe vuote sono considerati
equivalenti a `false`, perciò è possibile scrivere codice di questo genere:

```
if (x) {
  console.log('x non ha un valore');
} else {
  console.log('x vale: ' + x);
}
```

## Iterazione

Anche in questo caso la sintassi è quella tradizionale di molti altri linguaggi:

```
// Contiamo da 1 a 10
for (var i = 0; i < 10; i++) {
  console.log(i + 1);
}
```

## Funzioni

Così come in altri linuaggi (es. C) in JavaScript una funzione è una sequenza di
istruzioni parametrizzata e richiamabile attraverso il suo nome:

```
function somma(a, b) {
  return a + b;
}
var x = somma(7, 3);
console.log(x); // 10
```

**Nota:** Così come le variabili, anche i parametri di funzione non hanno tipo.

A differenza di quello che accade in molti altri linguaggi, in JavaScript le
funzioni sono considerate alla stregua di altri valori e possono essere
assegnate a variabile:

```
var unaFunzione = function (x) {
  console.log(x);
};
unaFunzione('Ciao'); // 'Ciao'
```

Questo esempio mostra anche un'altra caratteristica delle funzioni JavaScript:
possono essere "anonime". Naturalmente la dichiarazione di funzioni anonime ha
senso quando la funzione viene immediatamente assegnata a variabile oppure
quando viene invocata immediatamente:

```
(function (x) {
  console.log(x);
})('Ciao'); // 'Ciao'
```

Ovviamente per chi non ha esperienza di sviluppo JavaScript l'invocazione
immediata può in effetti apparire inutile e si potrebbe pensare di scrivere
semplicemente la corrispondente sequenza di istruzioni senza utilizzare una
funzione. L'esempio corrispondente, cioè, potrebbe essere riscritto
semplicemente così:

```
var x = 'Ciao';
console.log(x); // 'Ciao'
```

In realtà l'invocazione immediata di funzioni ha comunque senso in quanto
permette di definire variabili locali ad esse, senza "inquinare" lo scope
corrente:

```
function esempio() {
  var n = 7;
  console.log(n);
}
esempio(); // 7
console.log(n); // Errore (n qui non esiste)
```

Di scope parlerò meglio nella [sezione apposita](#scope).

Riguardo alle funzioni, è importante sapere che in JavaScript è possibile
passare ad una funzione un numero di argomenti diverso da quello previsto:
quelli mancanti saranno `undefined`, mentre quelli in eccesso verranno
semplicemente ignorati. Esempio:

```
function esempio(x1, x2) {
  if (x2 === undefined) {
    console.log(x1);
  } else {
    console.log([ x1, x2 ]);
  }
}
esempio(7); // 7
esempio(7, 42); // [ 7, 42 ]
esempio(7, 42, 666); // [ 7, 42 ]
```

È anche posssibile usare la parola chiave `arguments`, che in una funzione
rappresenta sotto forma di array la sequenza di argomenti con cui è stata
invocata:

```
function esempio() {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
  console.log(arguments.length);
}
esempio('a', 'b'); // 'a', 'b', undefined, 2
```

Ciò consente di implementare molto facilmente funzioni con un numero variabile
di parametri.

## Scope

In generale lo scope delle variabili JavaScript corrisponde alla funzione in cui
vengono dichiarate più le eventuali altre funzioni in essa annidate:

```
function funzioneEsterna() {
  var n = 7;
  function funzioneAnnidata() {
    // qui la variabile n è ancora accessibile
    function funzione ancoraPiuAnnidata() {
      // anche qui la variabile n è ancora accessibile
    }
  }
}
```

In altre parole, ogni funzione "cattura" il contesto di esecuzione in cui viene
dichiarata e si usa il termine "closure" per fare riferimento alla coppia
costituita appunto dalla funzione dichiarata e dal contesto di esecuzione in cui
avviene la dichiarazione.

Le closure permettono di preservare il contesto di esecuzione di una funzione
anche **dopo** che la sua esecuzione è ormai terminata:

```
function esempio(x) {
  return function () {
    return x;
  }
}
var f = esempio(7);
f(); // 7
```

In questo esempio la funzione `f` "tiene in vita" il contesto di esecuzione
della funzione `esempio` anche dopo che l'esecuzione di quest'ultima è
terminato.

Questo genere di comportamento può essere sfruttato in diversi modi, ma è
importante anche tenere presente che può portare a spreco di memoria.

È inoltre importante sapere che le variabili JavaScript sono soggette a
"hoisting" (innalzamento), cioè non importa in quale punto di una funzione
vengano dichiarate: l'interprete JavaScript si comporterà sempre come se la
dichiarazione fosse avvenuta all'inizio della funzione. Per intenderci, codice
di questo tipo:

```
function esempio() {
  console.log('ciao');
  var n = 7;
}
```

Viene interpretato come se fosse stato scritto in questo modo:

```
function esempio() {
  var n;
  console.log('ciao');
  n = 7;
}
```

Questo comportamento può causare a volte dei bug molto sottili e quindi molti
sviluppatori JavaScript cercando di dichiarare sempre tutte le variabili in
testa alle funzioni.

## Oggetti

Un oggetto JavaScript è essenzialmente un insieme di coppie nome-valore dette
"proprietà":

```
var marco = {
  nome: 'Marco',
  eta: 30
};
console.log(marco.nome); // 'Marco'
console.log(marco.eta); // 30
```

In questo esempio abbiamo un oggetto con due proprietà chiamate `nome` ed `eta`
con valori `'Marco'` e `30` e l'operatore `.` è stato utilizzato per accedere a
tali valore dati i corrispondenti nomi. In alternativa, è possibile usare la
seguente sintassi:

```
console.log(marco['nome']); // 'Marco'
console.log(marco['eta']); // 30
```

Questa sintassi ha 2 vantaggi. Il primo è che permette di passare il nome della
proprietà attraverso una variabile:

```
var qualeProprieta = 'nome';
console.log(marco[qualeProprieta]); // 'Marco'
```

L'altro è che permette di utilizzare nomi altrimenti vietati quali nomi
contenenti spazi e altri caratteri speciali o nomi corrispondenti a parole
chiave del linguaggio:

```
var oggettoStrano = {
  'proprietà con un nome strano': 42
};
console.log(oggettoStrano['proprietà con un nome strano']); // 42
```

Dato che in JavaScript le funzioni sono considerate valori, le proprietà di un
oggetto possono anche essere utilizzate come metodi:

```
var marco = {
  diQualcosa: function (cosa) {
    console.log(cosa);
  }
};
marco.diQualcosa('ciao'); // 'ciao'
```

Così come in molti altri linguaggi, la parola chiave `this` può essere
utilizzata in un metodo per accedere lo stato dell'oggetto sul quale il metodo
viene invocato:

```
var marco = {
  nome: 'Marco',
  presentati: function () {
    console.log('Ciao sono ' + this.nome);
  }
};
marco.presentati(); // 'Ciao sono Marco'
```

Una volta istanziato, un oggetto può essere modificato a piacere, aggiungendo,
eliminando o modificando le sue proprietà, metodi inclusi:

```
var marco = {}; // partiamo da un oggetto vuoto
marco.nome = 'Marco'; // Aggiungiamo una proprietà
// Aggiungiamo un metodo
marco.presentati = function () {
  console.log('Ciao sono ' + this.nome);
};
// Aggiungiamo una proprietà, poi la modifichiamo, poi la eliminiamo
marco.eta = 30;
marco.eta = 31;
delete marco.eta;
```

In JavaScript è molto semplice iterare sulle proprietà di un oggetto:

```
var marco = { nome: 'Marco', eta: 30 };
for (var nomeProprieta in marco) {
  console.log(marco[nomeProprieta]);
} // 'Marco', 30
```

## Prototipi

Le funzioni JavaScript possono essere utilizzate come costruttori di oggetti:

```
function Persona(nome) {
  this.nome = nome;
  this.presentati = function () {
    console.log('Ciao sono ' + this.nome);
  };
}
var marco = new Persona('Marco');
marco.presentati(); // 'Ciao sono Marco'
```

Il vantaggio di questa tecnica, rispetto all'uso della sintassi mostrata nella
sezione precedente, consiste nel fatto che tutti gli oggetti istanziati tramite
lo stesso costruttore possono essere successivamente estesi operando su un unico
oggetto, detto "prototipo", associato a tale costruttore:

```
function Persona(nome) {
  this.nome = nome;
}
var marco = new Persona('Marco');
var fabio = new Persona('Fabio');
Persona.prototype.presentati = function () {
  console.log('Ciao sono ' + this.nome);
};
marco.presentati(); // 'Ciao sono Marco'
fabio.presentati(); // 'Ciao sono Fabio'
```

In questo esempio i due oggetti `marco` e `fabio` hanno ottenuto automaticamente
un nuovo metodo dopo essere stati istanziati, semplicemente aggiungendo tale
metodo al loro prototipo.

Gli oggetti JavaScript sono liberi di sovrascrivere le proprietà ereditate:

```
function Persona(nome) {
  this.nome = nome;
  this.presentati = function () {
    console.log('Ciao sono ' + this.nome);
  };
}
var marco = new Persona('Marco');
var fabio = new Persona('Fabio');
fabio.presentati = function () {
  console.log('Mi chiamo ' + this.nome);
};
marco.presentati(); // 'Ciao sono Marco'
fabio.presentati(); // 'Mi chiamo Fabio'
```

Un altro vantaggio dell'uso di costruttori è la possibilità di effettuare test
tramite operatore `instanceof`:

```
function Persona(nome) { ... }
var marco = new Persona('Marco');
marco instanceof Persona; // true
var cloneDiMarco = { nome: 'Marco' };
cloneDiMarco instanceof Persona; // false
```

Tramite costruttori e prototipi è possibile riprodurre in JavaScript dei
meccanismi di ereditarietà simili a quelli di altri linguaggi:

```
function Persona() {}
function Sviluppatore() {}
Sviluppatore.prototype = new Persona();
var marco = new Sviluppatore();
marco instanceof Sviluppatore; // true
marco instanceof Persona; // true
```
