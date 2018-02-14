# TypeScript

## Sommario

TypeScript è un linguaggio progettato da Microsoft per fornire agli sviluppatori
JavaScript controlli automatici (e suggerimenti automatici) durante la scrittura
del codice e senza richiederne l'esecuzione (analisi statica). Tali feature si
basano soprattutto sulla dichiarazione dei tipi delle variabili e sulla
definizione di classi di oggetti.

TypeScript ha inoltre l'obiettivo di anticipare il supporto alle feature
definite nelle revisioni più recenti di JavaScript.

In questa guida presenterò brevemente TypeScript (versione 2) dando per noti i
concetti base della programmazione orientatata agli oggetti basata su classi
(interfacce, incapsulamento, erediterietà, polimorfismo, overriding, membri
statici). Prima di proseguire, può essere utile leggere anche la guida che ho
scritto su [ECMAScript 6](http://marcoliceti.xyz/it/guide/ecma-script-6.html),
visto che tali feature non saranno rispiegate in questa guida.

Il codice TypeScript va compilato (o meglio:
["transpilato"](https://en.wikipedia.org/wiki/Source-to-source_compiler)) in
codice JavaScript, perciò gli esempi in questa guida possono essere eseguiti su
browser (o su [Node](https://nodejs.org)) a patto di effettuarne appunto la
compilazione. A tale scopo è possibile utilizzare il [compilatore ufficiale di
Microsoft](https://www.typescriptlang.org). In alternativa è possibile usare
[questo compilatore online](https://www.typescriptlang.org/play/index.html)
ufficiale. Per un utilizzo più serio, è invece consigliabile [Visual Studio
Code](https://code.visualstudio.com/) o altro IDE.

## Indice

* [Tipi semplici e dichiarazione di variabili](#tipi-semplici-e-dichiarazione-di-variabili)
* [Array e tuple](#array-e-tuple)
* [Funzioni](#funzioni)
* [Interfacce](#interfacce)
* [Type assertion](#type-assertion)
* [Classi](#classi)
* [Implementazione di interfacce](#implementazione-di-interfacce)
* [Incapsulamento](#incapsulamento)
* [Membri statici](#membri-statici)
* [Ereditarietà e polimorfirmo](#ereditarieta-e-polimorfismo)
* [Overriding](#overriding)
* [Classi astratte](#classi-astratte)
* [Generics](#generics)

## Tipi semplici e dichiarazione di variabili

I tipi semplici di uso più comune sono:

* boolean
* number
* string
* void (può assumere solo i valori `null` e `undefined`)
* any (può assumere qualunque valore)

Esempi:

```
let n: number = 7; // OK
let s: string = 'ciao'; // OK
let b: boolean = true; // OK
let x: void = null; // OK
let y: any = 'va bene qualunque valore'; // OK

n = '7'; // Errore (string assegnata a number)
s = 42; // Errore (number assegnato a string)
b = 'true'; // Errore (string assegnata a boolean)
x = 'ciao'; // Errore (assegnato valore non null e non undefined)
```

Per le variabili inizializzate a tempo di dichiarazione è possibile omettere il
tipo, che verrà inferito dal compilatore:

```
let n = 7; (inferito tipo number)
n = 'ciao'; // Errore (string assegnata a number)
```

**Nota:** Per le variabili non inizializzate, in assenza di dichiarazione
esplicita, viene assegnato tipo `any`.

## Array e tuple

Una sintassi possibile è quella utilizzata nel seguente esempio:

```
let n: number[];
```

In alternativa:

```
let n: Array<number>;
```

In entrambi i casi, ovviamente, l'array potrà contenere solo elementi del tipo
specificato:

```
let n: number[];
n = [ 1, 2, 3 ]; // OK
n = [ '1', '2', '3' ]; // Errore
```

Se si desidera memorizzare valori di tipi disomogenei (come possibile in
JavaScript tradizionale) si possono utilizzare le tuple:

```
let voto: [ string, number ];
voto = [ 'informatica', 10 ]; // OK
voto = [ 'informatica', '10' ]; // Errore
```

Anche per gli array è supportata l'inferenza dei tipi:

```
let n = [ 1, 2, 3 ]; // tipo inferito: number[]
```

## Funzioni

È possibile specificare il tipo dei parametri e del valore di ritorno:

```
function somma(a: number, b: number): number {
  return a + b;
}
somma(2, 2); // 4
somma('2', '2'); // Errore
```

In molti casi risulta comodo e corretto omettere il tipo di ritorno, accettando
implicitamente quello inferito da TypeScript:

```
function somma(a: number, b: number) {
  return a + b; // tipo inferito: number
}
let s: string = somma(2, 2); // Errore (assegnato number a string)
```

## Interfacce

Definiscono tipi di oggetto in termini di proprietà previste:

```
interface Persona {
  nome: string;
  eta: number;
}
```

Le proprietà possono anche essere metodi e in tal caso la sintassi è simile a
quella con cui si dichiarano le funzioni:

```
interface Calcolatrice {
  somma(a: number, b: number): number;
  prodotto(a: number, b: number): number;
  ...
}
```

Le proprietà dichiarate in un'interfaccia sono intese come obbligatorie:

```
interface Persona {
  nome: string;
  eta: number;
}
const marco: Persona = { nome: 'Marco', eta: 30 }; // OK
const anonimo: Persona = { eta: 33 }; // Errore (manca il nome)
```

È però possibile marcarle come opzionali:

```
interface Persona {
  nome?: string;
  eta: number;
}
const marco: Persona = { nome: 'Marco', eta: 30 }; // OK
const anonimo: Persona = { eta: 33 }; // OK
```

Non è possibile utilizzare proprietà diverse da quelle previste:

```
interface Persona {
  nome: string;
  eta: number;
}
let marco: Persona = { nome: 'Marco', eta: 30 }; // OK
marco = { nome: 'Marco', eta: 30, professione: 'sviluppatore' }; // Errore
```

## Type assertion

Permettono di trattare una variabile come se avesse un certo tipo, diverso da
quello effettivo.

Esempio:

```
interface Persona {
  nome: string;
}

interface Lavoratore {
  nome: string;
  mestiere: string;
}

const marco: Persona = { nome: 'Marco' };
marco.mestiere = 'Sviluppatore'; // Errore
(marco as Lavoratore).mestiere = 'Sviluppatore'; // OK
```

Esiste anche una sintassi alternativa:

```
(<Lavoratore>marco).mestiere = 'Sviluppatore';
```

## Classi

È possibile definire classi e dichiarare le loro variabili d'istanza utilizzando
la stessa sintassi valida per le interfacce:

```
class Persona {
  nome: string;
}
const marco = new Persona();
marco.nome = 'Marco'; // OK
marco.mestiere = 'Sviluppatore'; // Errore
```

È possibile definire un costruttore personalizzato:

```
class Persona {
  nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }
}
const marco = new Persona('Marco');
console.log(marco.nome); // 'Marco'
```

È possibile definire uno o più metodi d'istanza:

```
class Persona {
  nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }

  creaSaluto(): string {
    return `Ciao sono ${this.nome}`;
  }
}

const marco = new Persona('Marco');
const saluto = marco.creaSaluto();
console.log(saluto); // 'Ciao sono Marco'
```

Una volta compilato il codice TypeScript, una classe viene tradotta in una
funzione da usare come costruttore. Ad esempio la seguente classe:

```
class Persona {
  nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }
}
const marco = new Persona('Marco');
```

...viene tradotta in qualcosa di simile a:

```
function Persona(nome) {
  this.nome = nome;
}
var marco = new Persona('Marco');
```

## Implementazione di interfacce

La classe implementante deve contenere le proprietà dichiarate nell'interfaccia:

```
interface AventeNome {
  nome: string;
}

interface Chiaccherone {
  diQualcosa(cosa: string);
}

class Persona implements AventeNome, Chiaccherone {
  nome: string;

  diQualcosa(cosa: string) {
    console.log(qualcosa);
  }
}
```

## Incapsulamento

```
class Persona {
  private nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }
}

const marco = new Persona('Marco');
console.log(marco.nome); // Errore (variabile privata)
```

Oltre a variabili d'istanza `private` è possibile dichiarare variabili d'istanza
`protected`, il che permette di accederle dall'interno di una
[sottoclasse](#ereditarieta).

Anche i metodi possono essere oggetto di incapsulamento:

```
class Esempio {
  private metodoPrivato() {
    console.log('OK');
  }

  metodoPubblico() {
    this.metodoPrivato();
  }
}

const esempio = new Esempio();
esempio.metodoPrivato(); // Errore (metodo privato)
esempio.metodoPubblico(); // 'OK'
```

Sono infine supportate anche variabili d'istanza `readonly`, che possono essere
oggetto di assegnazione solo all'interno del costruttore:

```
class Persona {
  readonly nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }

  cambiaNome(nuovoNome: string) {
    this.nome = nuovoNome; // Errore
  }
}

const marco = new Persona('Marco');
console.log(marco.nome); // 'Marco';
marco.nome = 'Mario'; // Errore
```

## Membri statici

Sono proprietà che invece di essere associate alle istanze della classe sono
associate alla classe stessa. A livello di codice JavaScript risultante, i
membri statici vengono assegnati alla funzione ottenuta a seguito della
transpilazione.

Per esempio il seguente codice TypeScript:

```
class Contatore {
  static n: number;

  static inizializza() {
    Contatore.n = 0;
  }
}
```

...viene tradotto in codice JavaScript simile al seguente:

```
function Contatore() {}
Contatore.inizializza = function () {
  Contatore.n = 0;
};
```

## Ereditarietà e polimorfismo

```
class Persona {
  protected nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }

  getNome() {
    return this.nome;
  }
}

class Lavoratore extends Persona {
  protected mestiere: string;

  constructor(nome: string, mestiere: string) {
    super(nome);
    this.mestiere = mestiere;
  }

  getNomeEMestiere() {
    return `${this.nome} ${this.mestiere}`;
  }
}

const marco: Persona = new Lavoratore('Marco', 'Sviluppatore');
marco.getNome(); // 'Marco'
marco.getNomeEMestiere(); // Errore
(marco as Lavoratore).getNomeEMestiere(); // 'Marco Sviluppatore'
```

## Overriding

```
class Persona {
  protected nome: string;

  constructor(nome: string) {
    this.nome = nome;
  }

  presentati() {
    return `Ciao sono ${this.nome}`;
  }
}

class Lavoratore extends Persona {
  protected mestiere: string;

  constructor(nome: string, mestiere: string) {
    super(nome);
    this.mestiere = mestiere;
  }

  presentati() {
    return `Ciao sono ${this.nome} e sono ${this.mestiere}`;
  }
}

const marcoPersona = new Persona('Marco');
const marcoLavoratore = new Lavoratore('Marco', 'Sviluppatore');
marcoPersona.presentati(); // 'Ciao sono Marco'
marcoLavoratore.presentati(); // 'Ciao sono Marco e sono Sviluppatore'
```

## Classi astratte

```
abstract class FiguraPiana {
  abstract area(): number;
}

class Quadrato extends FiguraPiana {
  lato: number;

  constructor(lato: number) {
    super();
    this.lato = lato;
  }

  area() {
    return this.lato * this.lato;
  }
}

const quadrato: FiguraPiana = new Quadrato(8);
quadrato.area(); // 64
```

## Generics

Sono supportate funzioni generiche:

```
function identita<T>(x: T): T {
  return x;
}

identita<number>(7); // 7
identita<number>('7'); // Errore
identita<string>('7'); // '7'
identita('7'); // '7' (tipo inferito)
```

...e classi generiche:

```
class CalcolatoreIdentita<T> {
  calcola(x: T): T {
    return x;
  }
}

const identitaNumeri = new CalcolatoreIdentita<number>();
identitaNumeri.calcola(7); // 7
identitaNumeri.calcola('7'); // Errore

const identitaStringhe = new CalcolatoreIdentita<string>();
identitaStringhe.calcola(7); // Errore
identitaStringhe.calcola('7'); // '7'
```
