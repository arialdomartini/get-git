# Introduzione

Questa guida è un po' diversa dalle altre.

Molti dei testi che ho letto su git si preoccupano di introdurti ai comandi base e lasciano ai capitoli più avanzati la descrizione del modello di funzionamento interno, oppure la saltano del tutto.

Quello che ho notato, però, è che imparando git partendo dai comandi base rischi di finire per usarlo come uno strumento vagamente simile a SVN ma provvisto di un set aggiuntivo di comandi esoterici, il cui funzionamento ti resterà sostanzialmente oscuro.

Facci caso: alcuni di quelli che hanno imparato git abbasanza da riuscire ad usarlo quotidianamente ti racconteranno di aver fatto molta fatica a capire cosa sia un `rebase` o di non cogliere esattamente che uso fare dell'`index`. 

La mia impressione è che, una volta capito il modello interno (che è sorprendentemente semplice!), tutto git appaia improvvisamente lineare e coerente: non c'è davvero alcun motivo per cui il `rebase` dovrebbe essere un argomento misterioso.

Questa guida prova a spiegarti git seguendo un percorso contrario a quello adottato di solito: partirai dalla spiegazione degli internal e finirai per imparare, nello stesso momento, sia comandi base che quelli avanzati, in poco tempo e senza troppi grattacapi.

Non imparerai, però, tutti i comandi. Piuttosto che mostrarti tutte le opzioni disponibili, questa guida punterà a farti comprendere i concetti e il modello sottostante e a darti gli strumenti per essere autonomo quando vorrai approfondire un argomento sulle man page o vorrai fare qualcosa di fuori dall'ordinario con il tuo `repository`.

Un'ultima nota: questa guida è organizzata come un lungo tutorial. Se ti armi di terminale ed esegui ognuno dei comandi, tipograficamente riportati così

>ls

potrai riprodurre esattamente sul tuo computer ognuno degli esempi della guida.

# Non sono parente di SVN

Per chi arrivi da SVN, git presenta una sola difficoltà: ha molti comandi identici. Ma è una somiglianza superficiale e ingannevole: sotto il cofano git è totalmente differente.

Per questo ti suggerisco di rifuggire sempre dalla tentazione di fare dei paralleli con SVN, perché sarebbero solo fuorvianti. Troverai comandi come `add`, `checkout`, `commit` e `branch` che ti sembrerà di conoscere. Ecco: fai *tabula rasa* di quel che conosci, perché in git quei comandi significano cose molto molto differenti.

Tentare di capire git usando SVN come modello, a volte, porta semplicemente fuori strada.<br/>
Per esempio: ci crederesti che questo repository ha 3 branch?

![Alt text](img/3-branches.png)

Sì: 3 branch, non 2.

Oppure: ci crederesti che git, più che un sistema di versionamento del codice, potrebbe essere meglio descritto come un "*sistema peer-to-peer di database chiave/valore su file system*"?

Per cui: dimentica quello che sai sui branch e sui changeset di SVN, e preparati a concetti completamente nuovi.

Sono persuaso che li troverai molto più omogenei e potenti di quelli di SVN. Devi solo predisporti ad un piccolo salto culturale.


## Setup

Installa [git](http://git-scm.com/downloads).

Poi configuralo perché ti riconosca

> git config --global user.name "Arialdo Martini"<br/>
> git config --global user.emal arialdomartini@gmail.com`

Se sei su Windows puoi eseguire quei comandi in `git bash`, un terminale predisposto a `git`. Su Linux e Mac OS X puoi usare il tuo terminal preferito.

Se vuoi, installa anche un client grafico. Io ti suggerisco [SmartGit](http://www.syntevo.com/smartgithg/), che è gratuito per progetti OpenSource. Altrimenti appoggiati al tool `gitx` che trovi in bundle insieme all'installazione di git.

Fantastico. Partiamo.


# Gli internal di git
## 3 differenze principali

Iniziamo con tre caratteristiche di git con le quali dovresti familiarizzare.



1. **Non c'è un server**: il repository è locale. La gran parte delle operazioni è locale e non richiede l'accesso alla rete. Anche per questo troverai git  incredibilmente veloce.
2. **Il progetto è indivisibile**: git lavora sempre con l'intero codice sorgente del progetto e non su singole directory o su singoli file; con git non c'è differenza tra committare nella directory principale o in una sotto-directory. Non esiste il concetto di `checkout` di un singolo file o di una singola directory. Per git il progetto è l'unità indivisibile di lavoro.
3. **git non memorizza i cambiamenti dei file**: git salva sempre i file nella loro interezza. Se in un file di 2 mega modificassi un singolo carattere, git memorizzerebbe per intero la nuova versione del file. Questa è una differenza importante: SVN memorizza le differenze e, all'occorrenza, ricostruisce il file; git memorizza il file e, all'occorrenza, ricostruisce le differenze.


### 4 livelli di nerdosità

Sull'assenza di un server ho un po' mentito: come ti ho già detto e come vedrai più avanti, git è un sistema peer-to-peer, e riesce ad interagire con dei server remoti.
Nonostante questo resta sostanzialmente un sistema locale.

Per capire quanto questo possa avvantaggiarti, prova a vederla così: quando il codice sorgente di un progetto è ospitato su un computer remoto hai 4 modi per editare il codice

1. Lasci tutto il codice sul computer remoto e vi **accedi con ssh per editare un singolo file**
2. Trovi il modo di ottenere **una copia del singolo file** per poterci lavorare più comodamente in locale e lasci tutto il resto sul computer remoto
3. Trovi il modo di ottenere **una copia locale di un intero albero del file system** e lasci il resto della storia dei commit sul computer remoto
4. Ottenieni una **copia locale dell'intero repository** con tutta la storia del progetto e lavori in locale

Avrai notato due cose.

La prima, che SVN e i sistemi di versionamento ai quali sei probabilmente abituato operano al livello 3.

La seconda, che i 4 sistemi sono elencati in ordine di comodità: in linea di massima, quando il materiale è conservato sul sistema remoto il tuo lavoro è più macchinoso, lento e scomodo. SVN ti permette di fare il checkout di un'intera directory proprio perché così ti risulti più comodo passare da un file all'altro senza dover continuamente interagire col server remoto.

Ecco: git è ancora più estremo; preferisce farti avere a disposizione tutto sul tuo computer locale; non solo il singolo checkout, ma l'intera storia del progetto, dal primo all'ultimo commit.

In effetti, qualunque cosa tu voglia fare, git chiede normalmente di ottenere una copia completa di quel che è presente sul server remoto. Ma non preoccuparti troppo: git è più veloce a ottenere l'intera storia del progetto di quanto SVN lo sia ad ottenere un singolo checkout.


### Il modello di storage

Passiamo dalla terza differenza. E preparati a conoscere il vero motivo per cui git sta sostituendo molto velocemente SVN come nuovo standard *de-facto*.

* SVN memorizza la collezione delle varie patch (o diff) applicate nel tempo ai file; all'occorrenza ricostruisce lo stato attuale;
* git memorizza i file così come sono, nella loro interezza; all'occorrenza ne calcola le diff.

Se vuoi evitare tanti grattacapi con git, il miglior suggerimento che tu possa seguire è di trattarlo come un **database chiave/valore**. 

Passa al terminal e guarda nel concreto.

Mettiti nella condizione di avere 2 file vuoti sul file system: 

> mkdir progetto<br/>
> cd progetto<br/>
> mkdir libs<br/>
> touch libs/foo.txt<br/>
> mkdir templates<br/>
> touch templates/bar.txt<br/>

    /
    ├──libs
    |     └──foo.txt
    |
    ├──templates
            └──bar.txt


Decidiamo di gestire il progetto con git

> git init

Aggiungi il primo file a git

> git add libs/foo.txt

Con questo comando, git ispeziona il contenuto del file (è vuoto!) e lo memorizza nel suo database chiave/valore, chiamato `blob storage` e conservato su file system nella directory nascosta `.git`. 

Siccome il `blob-storage` è un database chiave valore, git cercherà di calcolare una chiave ed un valore per il file che hai aggiunto. Per il valore git userà il contenuto stesso del file; per la chiave, calcolerà lo sha1 del contenuto (se sei curioso, nel caso di un file vuoto vale `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`)

Per cui, nel `blob storage` git salverà un oggetto `blob`, univocamente identificabile dalla sua chiave (che, in assenza di ambiguità, vale la pena di abbreviare)

![Alt text](img/blob.png)

Adesso aggiungi il secondo file

> git add templates/bar.txt

Ora, siccome `libs/foo.txt` e `templates/bar.txt` hanno lo stesso identico contenuto (sono entrambi vuoti!), nel `blob storage` entrambi verranno conservati in un unico oggetto:

![Alt text](img/blob.png)


Come vedi, nel `blob storage` git ha memorizzato solo il contenuto del file, non il suo nome né la sua posizione.

Naturalmente, però, a noi il nome dei file e la loro posizione interessano eccome.
Per questo, nel `blob storage`, git memorizza anche altri oggetti, chiamati `tree` che servono proprio a memorizzare il contenuto delle varie directory e i nomi dei file.

Nel nostro caso, avremo 3 `tree` 

![Alt text](img/tree.png)

Come ogni altro oggetto, anche i `tree` sono memorizzati come chiave/valore.

Tutte queste strutture vengono raccolte dentro un contenitore, chiamato `commit`.

 
![Alt text](img/commit.png)


Come avrai intuito, un `commit` non è altro che un elemento del database chiave/valore, la cui chiave è uno SHA1, come per tutti gli altri oggetti, e il cui valore è un puntatore al `tree` del progetto, cioè la sua chiave (più un altro po' di informazioni, come la data di creazione, il commento e l'autore).<br/>
Non è troppo complicato, dopo tutto, no?

Quindi, il `commit` è l'attuale fotografia del file system.

Adesso fai

> git commit -m "commit A, il mio primo commit"

Stai dicendo a git: 

*memorizza nel repository, cioè nella storia del progetto, il commit che ti ho preparato a colpi di add*

Il tuo `repository`, visto da SmartGit, adesso ha questo aspetto

![Alt text](img/first-commit.png)

La riga col pallino che vedi sulla sinistra rappresenta l'oggetto `commit`. Nel pannello sulla destra, invece, puoi vedere la chiave del `commit`.

In generale, a meno che non si debba parlare proprio del modello interno come stiamo facendo adesso, non c'è una grande necessità di rappresentare tutta la struttura di `blob` e `tree` che costituisce un `commit`. Difatti, dopo il prossimo paragrafo inizieremo a rappresentare i `commit` come nella figura qui sopra: con un semplice pallino.

Già da adesso, comunque, dovrebbe risultarti più chiaro il fatto che dentro un `commit` ci sia l'intera fotografia del progetto e che, di fatto, un `commit` sia l'unità minima ed indivisibile di lavoro.


## L' `index` o `staging area` 

Sostanzialmente, non c'è molto altro che tu debba sapere del modello di storage di git. Ma prima di passare a vedere i vari comandi, vorrei introdurti ad un altro meccanismo interno: la `staging area` o `index`. L'`index` risulta sempre misterioso a chi arrivi da SVN: vale la pena parlarne perché quando saprai come funzionano il `blob storage` e l'`index`, git non ti sembrerà più contorto e incomprensibile; piuttosto, ne coglierai la coerenza e lo troverai estremamente prevedibile.

L'`index` è una struttura che fa da cuscinetto tra il file system e il repository. È un piccolo buffer che puoi utilizzare per costruire il prossimo `commit`. 

![Alt text](img/index1.png)

Non è troppo complicato:

 * il `file system` è la directory con i tuoi file.
 * il `repository` è il database locale su file, che conserva i vari `commit`
 * l'`index` è lo spazio che git ti mette a disposizione per creare il tuo prossimo `commit` prima di registrarlo definitivamente.

Fisicamente, l'`index` non è molto diverso dal `repository`: entrambi conservano i dati nel `blob storage`, usando le strutture che hai visto prima.

In questo momento, appena dopo aver completato il tuo primo `commit`, l'`index` conserva una copia del tuo ultimo `commit` e si aspetta che tu lo modifichi.

![Alt tex1](img/index2.png)


Sul file system hai

    /
    ├──libs
    |     └──foo.txt
    |
    ├──templates
            └──bar.txt


Proviamo a fare delle modifiche al file `foo.txt`

>  echo "nel mezzo del cammin" >> libs/foo.txt 

e aggiorna l'`index` con

> git add libs/foo.txt

All'esecuzione di `git add` git ripete quel che aveva già fatto prima: analizza il contenuto di `libs/foo.txt`, vede che c'è un contenuto che non ha mai registrato e quindi aggiunge al `blob storage` un nuovo `blob` col nuovo contenuto del file; contestualmente, aggiorna il `tree` `libs` perché il puntatore chiamato `foo.txt` indirizzi il suo nuovo contenuto

![Alt tex1](img/index3.png)

Prosegui aggiungendo un nuovo file `doh.html` alla root del progetto

>  echo "happy happy joy joy" > doh.html<br/>
>  git add doh.html

Come prima: git aggiunge un nuovo `blob` object col contenuto del file e, contestualmente, aggiunge nel `tree` "/" un nuovo puntatore chiamato `doh.html` che punta al nuovo `blob` object

![Alt tex1](img/index4.png)

Il contenitore di tutta questa struttura è sempre un oggetto `commit`; git lo tiene parcheggiato nella `staging area` in attesa che tu lo spedisca al `repository`.
Questa struttura rappresenta esattamente la nuova situazione sul file system: è nuovamente una fotografia dell'intero progetto, ed include anche il file `bar.txt`, nonostante tu non lo abbia modificato. Per inciso: non dovresti preoccuparti per il consumo di spazio perché, come vedi, per memorizzare `bar.txt` git sta riutilizzando l'oggetto `blob` creato nel `commit` precedente, per evitare duplicazioni.

Bene. Abbiamo quindi una nuova fotografia del progetto.<br/>
A noi interessa, però, che git conservi anche la storia del nostro file system, per cui ci sarà bisogno di memorizzare da qualche parte il fatto che questa nuova situazione (lo stato attuale dell'`index`) sia figlia della precedente situazione (il precedente `commit`).

In effetti, git aggiunge automaticamente al `commit` parcheggiato nella `staging area` un puntatore al `commit` di provenienza

![Alt tex1](img/index-and-first-commit.png)

La freccia rappresenta il fatto che l'`index` è figlio del `commit A`. È un semplice puntatore. Nessuna sopresa, se ci pensi; git, dopo tutto, utilizza il solito, medesimo, semplicissimo modello ovunque: un database chiave/valore per conservare il dato, e una chiave come puntatore tra un elemento e l'altro.


Ok. Adesso committa


>  git commit -m "Commit B, Il mio secondo commit"


Con l'operazione di commit si dice a git "*Ok, prendi l'attuale `index` e fallo diventare il tuo nuovo `commit`. Poi restituiscimi l'`index` così che possa fare una nuova modifica*"


Dopo il `commit` nel database di git avrai

![Alt tex1](img/index-and-second-commit.png)

Una breve osservazione: spesso le interfacce grafiche di git omettono di visualizzare l'`index`. `gitk`, per esempio, la visualizza solo se ci sono modifiche da committare. Il tuo repository in `gitk` adesso viene visualizzato così

![Alt tex1](img/gitk.png)

Guarda tu stesso. Lancia

>  gitk


Ricapitolando:

1. git memorizza sempre i file nella loro interezza
2. il `commit` è uno dei tanti oggetti conservati dentro il database chiave/valore di git. È un contenitore di tanti puntatori ad altri oggetti del database: i `tree` che rappresentano directory con nomi di file che a loro volta puntano ad altri `tree` (sottodirectory) o a dei `blob` (il contenuto dei file)
3. ogni oggetto `commit` ha un puntatore al `commit` padre da cui deriva
4. l'`index` è uno spazio di appoggio nel quale puoi costruire, a colpi di `git add`, il nuovo `commit` 
5. con `git add` aggiungi un file all'`index`; con `git commit` registri l'attuale `index` facendolo diventare il nuovo `commit`.  ![Alt tex1](img/index-add-commit.png)  



Bene: adesso hai tutta la teoria per capire i concetti più astrusi di git come il `rebase`, il `cherrypick`, l'`octopus-merge`, l'`interactive rebase`, il `revert` e il `reset`.

Passiamo al pratico.



# I comandi di git

## Obiettivo 1: tornare indietro nel tempo

Dunque, se in git tutto è conservato in un database chiave/valore, probabilmente ci saà modo per referenziare un qualunque oggetto del database usando la sua chiave.

In effetti è proprio così.<br/>

Adesso proviamo a tornare indietro nel tempo, al `commit A`, utilizzando il comando `git checkout`.

Il comando `checkout` prende il `commit` indicato e lo copia nel `file system` e nella `staging area`.

![Alt tex1](img/index-add-commit-checkout.png)

Già: ma qual è la chiave del `commit A`?
Lo scopriamo con un client grafico o col comando `git log` che mostra tutto quello che abbiamo fatto fin'ora

> git log --oneline<br/>
>**2a17c43** Commit B, Il mio secondo commit<br/>
>**56674fb** commit A, il mio primo commit<br/>

Occhio! Siccome nel commit vengono memorizzati anche la data e l'autore, le tue chiavi risulteranno diverse dalle mie.

Sul mio `repository` la chiave del `commit A` è `56674fb`.<br/>
Bene: torniamo indietro al passato, al momento del commit A

> ls<br/>
> doh.html&nbsp;&nbsp;&nbsp;&nbsp;libs&nbsp;&nbsp;&nbsp;&nbsp;templates<br/>
> **git checkout 56674fb**<br/>
> ls<br/>
> libs&nbsp;&nbsp;&nbsp;&nbsp;templates<br/>

Effettivamente, a parte un misterioso e prolisso messaggio di con cui git si lamenta di essere in `'detached HEAD' state` (poi chiariremo questo punto), il file system è tornato allo stato del primo commit e, infatti, il file `doh.html` è scomparso.



##Obiettivo 2: divergere

Usando una convenzione grafica molto comune nella letteratura su git, potremmo rappresentare la situazione attuale del tuo repository con

> **A**---B

Cioè: ci sono due `commit`, `A` e `B`. Il `commit B` è figlio di `A` (il tempo scorre verso destra). Il `commit` in grassetto indica il punto in cui ti trovi attualmente.


Che succederebbe se adesso facessi qualche modifica e committassi?<br/>
Accadrebbe che il nuovo `commit C` che andresti a generare sarebbe figlio di `A` (perché è da lì che parti), ma la linea di svilupppo proseguirebbe divergendo dalla linea `A---B`.

Cioè, si creerebbe questa situazione

>  A---B<br/>
>  &nbsp;&nbsp;\ <br/>
>  &nbsp;&nbsp;&nbsp;**C**

Provalo davvero:

>  echo "ei fu siccome immobile" > README.md<br/>
>  git add README.md<br/>
>  git commit -m "Ecco il commit C"<br/>

![Alt tex1](img/repo1.png)


Hai ottenuto una diramazione, senza il meccanismo della copia utilizzato da SVN: il modello a chiave/valore e puntatori di git rende molto economico rappresentare una linea di sviluppo che diverge.

Due osservazioni importanti.
 
La prima per ribadire il concetto che git non ha mai memorizzato i "diff" tra i file: `A`, `B` e `C` sono snapshot dell'intero progetto. È molto importante ricordarselo, perché ti aiuterà a capire che tutte le considerazioni che sei sempre stato abituato a fare con SVN qui non valgono.

La seconda è un po' sorprendente: le due linee di sviluppo divergenti che hai appena visto non sono `branch`. In git i rami sono dei puntatori dotati di nome, o delle etichette. Te ne parlerò nel prossimo paragrafo, ma abituati già a ripeterti: in git i `branch` non sono rami di sviluppo.


## Obiettivo 3: creare un branch

Con il comando `checkout` hai imparato a spostarti da un `commit` all'altro



Basta conoscere la chiave di ogni `commit`
> git log --oneline --all<br/>
>**deaddd3** Ecco il commit C<br/>
>**2a17c43** Commit B, Il mio secondo commit<br/>
>**56674fb** commit A, il mio primo commit<br/>

![Alt tex1](img/repo1.png)

>git checkout **56674fb** # vai al `commit A`<br/>
>git checkout **2a17c43** # vai al `commit B`<br/>
>git checkout **deaddd3** # vai al `commit C`<br/>

Sì, però, bisogna ammetterlo: gestire i `commit` `A`, `B` e `C` dovendoli chiamare `56674fb`, `2a17c43` e `deaddd3` è di una scomodità unica.

git risolve il problema facendo quello che farebbe ogni programmatore di buon senso: dal momento che quei numeri sono dei puntatori ad oggetti, git permette di salvarli in delle variabili. Assegnare un valore ad una variabile è semplice:

>git branch bob 56674fb

![Alt tex1](img/bob.png)

Vedi l'etichetta `bob` proprio in corrispondenza del `commit B`? Sta ad indicare che l'etichetta `bob` punta a quel `commit`.

Quando crei un'etichetta, se non specifichi un valore, git userà la chiave del `commit` sul quale ti trovi al momento

>git checkout 300c737<br/>
>git branch piccio 

![Alt tex1](img/piccio.png)

L'eliminazine di una variabile è ugualmente banale:

>git branch -d teddy<br/>
>git branch -d piccio


Avrai notato che di default git crea alcune di queste variabili. Per esempio, nelle figure sopra appariva anche la variabile `master`, puntata su `B`.


![Alt tex1](img/repo2.png)


L'etichetta `master` ti permette di andare sul quel `commit` scrivendo:

> git checkout master

Ora attento, perché siamo di nuovo in una di quelle occasioni dove la conoscenza di SVN fornisce solo dei grattacapi: queste etichette in git si chiamano `branch`. Ripetiti mille volte: un `branch` in git non è un ramo, è un'etichetta, un puntatore ad un `commit`, una variabile che contiene la chiave di un `commit`. Tanti comportamenti di git che appaiono assurdi e complicati diventano molto semplici se eviti di pensare ai `branch` di git come ad un equivalente dei branch di SVN.

Dovrebbe iniziare a risultarti chiaro perché molti dicano che "*i branch su git sono molto economici*": per forza! Sono delle semplicissime variabili!

Divertiti ad aggiungerne altre

>git branch dev

![Alt tex1](img/branch-dev.png)

Voilà: hai aggiunto un `branch`.

Nota un'altra cosa: vedi che accanto a `master` SmartGit aggiunge un segnaposto triangolare verde? Quel simbolo indica che in questo momento sei *agganciato* al `branch` `master`, perché il tuo ultimo comando di spostamento è stato `git checkout master`.

Potresti spostarti su `dev` con

>git checkout dev

![Alt tex1](img/branch-dev2.png)

Il segnaposto si è spostato su `dev`.

Di default git aggiunge sempre anche un'altra variabile: il puntatore `HEAD`, che punta sempre all'elemento del `repository` sul quale ti trovi. Sostanzialmente, il segnaposto visualizzato da SmartGit indica la posizione di HEAD. Altri editor grafici utilizzano differenti rappresentazioni. `gitk`, per esempio, visualizza in grassetto il `branch` sul quale ti trovi.<br/>
Per sapere su quale `branch` ti trovi, dalla linea di comando, ti basta eseguire

>git branch<br/>
>* dev<br/>
>  master<br/>

L'asterisco suggerisce che `HEAD` adesso stia puntanto a `dev`.

Non dovresti essere troppo sorpreso nel verificare che, nonostante tu abbia cambiato `branch` da `master` a `dev` il tuo `file system` non sia cambiato di una virgola: in effetti, sia `dev` che `master` stanno puntando allo stesso identico `commit`.

Non di meno, ti domanderai probabilmente a cosa mai possa servire passare da un `branch` all'altro, se non sortisce alcun effetto sul progetto.

Il fatto è che quando esegui il `checkout` di un `branch`, in qualche modo ti *agganci* all'etichetta; l'etichetta del `branch`, in altre parole, inizierà a seguirti, `commit` dopo `commit`.

Guarda: adesso sei su `dev`. Apporta una modifica qualsiasi e committa

>touch style.css<br/>
>git add style.css<br/>
>git commit -m "Adesso ho anche il css"<br/>

![Alt tex1](img/branch-dev3.png)

Visto cosa è successo? L'etichetta `dev` si è spostata in avanti e si è agganciata al tuo nuovo `commit`.

Ti domanderai anche perché mai git chiami quelle etichette `branch`. Il motivo è che, anche se le linee di sviluppo che divergono in git non sono `branch`, i `branch` vengono normalmente usati proprio per dar loro un nome.

Guardalo nel concreto. Torna a `master` ed apporta qualche modifica.

>git checkout master<br/>
>touch angular.js<br/>
>git add angular.js<br/>
>git commit -m "angular.js rocks" 

![Alt tex1](img/angular.png)

Come c'era da aspettarselo, l'etichetta `master` è avanzata di un posto, per puntare al tuo nuovo `commit`.

Adesso c'è una certa equivalenza tra le linee di sviluppo e i `branch`. Nonostante questo, ti conviene sempre tenere mentalmente separati i due concetti, perché ti faciliterà molto la gestione della storia del tuo progetto

Per esempio: non c'è dubbio che il `commit` col commento "*angular.js rocks*" sia contenuto nel `branch master`, giusto?<br/>
Che dire però di `A` e di `B`? A quale `branch` appartengono?

Occhio, perché questo è un altro dei concetti che procurano dei mal di testa agli utenti di SVN, e perfino a quelli di Mercurial. 

In effetti, per rispondere a questo interrogativo gli utenti di git si pongono una domanda differente: 

"*il `commit A` è raggiungibile da `master`?*"

Cioè: percorrendo a ritroso la storia dei `commit` partendo da `master`, si passa da `A`?<br/>
Se la risposta è *sì* si può afferamere che `master` contenga le modifiche introdotte da `A`.

Una cosa che i fan di Mercurial e di SVN potrebbero trovare disorientante è che, siccome il `commit A` è raggiungibile anche da `dev`, appartiene *sia* a `master` che a `dev`.

Pensaci su. Se tratti i `branch` come puntatori a `commit` dovrebbe sembrarti tutto molto lineare.


# Obiettivo 4: fare i giocolieri con i `commit`

Come hai visto, git riesce a conservare la storia delle modifiche dei file senza mai salvarne le differenze.<br/>
All'inizio della guida ti avevo anticipato il comportamento diametralmente opposto di SVN e di git, su questo punto:

* SVN memorizza le diff e, all'occorrenza, ricostruisce lo stato attuale;
* git memorizza lo stato attuale e, all'occorrenza, calcola le diff.

Per cui, quando nel `repository` fai riferimento al `commit` `dev`, intendi "*l'intero progetto, così come è stato fotografato al momento di quel commit*".


![Alt tex1](img/angular.png)


Se la stessa situazione fosse su SVN diresti che il commit `dev` "*contiene tutte le modifiche apportate ai file, partendo dal commit immediatamente precedente*".  

Per git, calcolare le modifiche apportate ai file da un `commit` all'altro non è poi difficile. Per esempio, puoi ricavarle con

> git diff dev master

Con `git diff FROM TO` chiedi a git "*qual è l'elenco delle modifiche ai file che devo applicare a `FROM` perché il progetto diventi identico a quello di `TO`*"? 


Con un po' di immaginazione puoi pensare che le linee tra i commit rappresentino le modifiche che tu hai apportato ai file e alle directory per ottenere un `commit`.<br/>
Per esempio, qui in rosso ho evidenziato la linea che rappresenta quel che hai fatto quando sei partito da `B` e hai creato il commit `dev`.

![Alt tex1](img/angular-highlighted.png)

Se rammenti, avevi fatto 

>touch style.css<br/>
>git add style.css<br/>
>git commit -m "Adesso ho anche il css"<br/>


Quindi, potresti dire che quella linea rossa rappresenti l'aggiunta del file `style.css`.


Bene. Tieni a mente questo modello. Adesso ti mostrerò uno dei comandi più folli e versatili di git: `cherry-pick`.

`cherry-pick` applica i cambiamenti introdotti da un commit sopra un altro commit.

Vediamolo subito con un esempio.

> git checkout dev<br/>
> git checkout -b experiment&nbsp;&nbsp;&nbsp;# esegue sia branch che checkout<br/>
> touch experiment<br/>
> git add experiment<br/>
> git commit -m "un commit con un esperimento"<br/>

![Alt tex1](img/cherry-pick-1.png)

Ci interesserebbe applicare l'esperimento anche al ramo `master`

> git checkout master<br/>
> git cherry-pick experiment

![Alt tex1](img/cherry-pick-2.png)

git ha applicato il cambiamento introdotto dal commit `dev` (la linea evidenziata in rosso xxx) al commit `master`, e poi ha fatto un `commit`. `cherry-pick` "coglie" il `commit` che gli indichi e lo applica sul `commit` dove ti trovi.

Se guardi sul `file system`, infatti, ti accorgi che git ha aggiunto il file `style.css` xxxx

Inizi a intuire le giocolerie che potrai fare con questo strumento?<br/>
Voglio darti qualche spunto.

### Correggere un bug a metà di un ramo

Proviamo a creare una linea di sviluppo con 3 `commit`


> git checkout -b feature&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# scorciatoria per fare branch + checkout<br/>
> touch feature<br/>
> git add feature<br/>
> git commit -m "feature"<br/>
><br/>
> touch orribile-baco<br/>
> git add orribile-baco<br/>
> git commit -m "orrore e raccapriccio"<br/>
><br/>
> touch altra-feature<br/>
> git add altra-feature<br/>
> git commit -m "altra feature"<br/>


![Alt tex1](img/bug-1.png)

Oh, no! il secondo `commit` è stato un errore madornale!  Ah, se solo si potesse riscrivere la storia ed evitarlo!

L'idea è di tornare indietro nel tempo, su `master`, ed usare `cherry-pick` per riapplicarvi le modifiche, avendo cura però di non applicare le modifiche introdotte da `orrore e raccapriccio`. Questo è uno dei casi in cui ti sarà necessario conoscere i valori delle chiavi:

>git log master..feature --oneline<br/>
>**8f41bb8** altra feature<br/>
>**ec0e615** orrore e raccapriccio<br/>
>**b5041f3** feature<br/>


Inizia a tonare indietro nel tempo. Riposizionati su `master`

>git checkout master<br/>

e sposta il `feature` indietro, nella posizione dove lo avevi creato prima di fare i `commit`

>git branch --force feature 

![Alt tex1](img/bug-2.png)

Non ti resta che prenderti, con `cherry-pick` i soli `commit` che ti interessano

>git checkout feature<br/>
>git cherry-pick b5041f3&nbsp;&nbsp;&nbsp;# prendo "feature"

![Alt tex1](img/bug-3.png)

>git cherry-pick 8f41bb8&nbsp;&nbsp;&nbsp;# prendo "altra feature"

![Alt tex1](img/bug-4.png)

Et voilà. Hai ricostruiro il ramo di sviluppo saltando il `commit` sbagliato. Resta un ramo orfano, cioè, senza alcun `branch`: verrà cancellato prima o poi dal garbage collector di git. I rami orfani di solito non vengono mostrati dagli editor grafici per cui, a cose normali, dovresti vedere questa situazione:

Partivi da 

![Alt tex1](img/bug-1.png)

e finisci con

![Alt tex1](img/bug-5.png)

Urca! L'impressione è che git abbia riscritto la storia eliminando un `commit` a metà di un ramo, vero?<br/>
 
Infatti, molti raccontano che git sia capace di riscrivere la storia e che questo suo comportamento sia estremamente pericoloso. Ecco: tu hai visto che non è così; git è estremamente conservativo e quando ti permette di manipolare i `commit` non fa altro che agire *in append*, costruendo *nuovi* rami. 


### Spostare un ramo di sviluppo

Voglio farti vedere un'altra magia del `cherry-pick`, per dissipare il velo di mistero di cui è inspiegabilmente circondato il comando `rebase`.

Riprendi il tuo `repository`. Mettiamo che ti vada di proseguire a sviluppare i tuoi css, per cui farai un nuovo commit su `dev`

![Alt tex1](img/rebase-1.png)

>git checkout dev<br/>
>echo "a { color:red; }" >> style.css<br/>
>git commit -am "i link sono rossi"
>&nbsp;&nbsp;&nbsp;# commit -am è uno shortcut per git add + commit

![Alt tex1](img/rebase-2.png)

Ottimo. I tuoi css sono perfetti. Peccato che il ramo `dev` sia rimasto un po' indietro rispetto a `master`. Del resto, cosa potevi farci? `master` è andato avanti e `dev` è rimasto lì dove lo avevi creato.

Certo, se si potesse spostare il ramo `dev` *sopra* `master`…

Non ti torna in mente `cherry-pick`? È un caso come quello precedente: solo che invece di viaggiare nel passato devi avere un po' di fantasia e immaginare di viaggiare nel futuro. Devi riportare i `commit` di `dev` (scritti nel passato) sull'ultimo commit di `master`, che relativamente a `dev` è il futuro.

Cioè: riscrivi la storia facendo come se i commit di `dev` siano stati scritti *dopo* i `commit` di `master`. 

Ok. Si tratta di spostare 2 `commit`. A colpi di `cherry-pick` sposterai i due commit del ramo blue sopra `master`.

Il risultato sarà questo:

![Alt tex1](img/rebase-3.png)

Confrontalo con la situazione di partenza

![Alt tex1](img/rebase-2.png)

Vedi cosa è successo? Il ramo `dev`, è stato staccato ed è stato impiantato sopra master.

Uno shortcut per evitare di spostare un `commit` alla volta da un ramo all'altro è il comando `git-rebase`.

Provalo. Sul tuo `repository`

![Alt tex1](img/rebase-2.png)

esegui

> git rebase master

dice a git: "*sposta il ramo corrente sulla nuova base: *`master`".<br/>
Sotto sotto, git non fa altro che eseguire una serie di `cherry-pick`: prende tutti i `commit` di `dev` che `master` ancora non ha e ce li applica in ordine.

Il risultato sarà

![Alt tex1](img/rebase-3.png)

Vedi? È del tutto equivalente a spostare uno per uno i `commit` con `cherry-pick`.

Riesci ad immaginare a cosa potrebbe servire un tool simile?<br/>
Guarda, provo a descriverti una situazione molto comune.

Stacchi un ramo da `dev` e inizi a lavorarci

>git checkout -b sviluppo<br/>
>touch file1 && git add file1 && git commit -m "avanzamento 1"<br/>
>touch file2 && git add file2 && git commit -m "avanzamento 2"<br/>
>touch file3 && git add file3 && git commit -m "avanzamento 3"<br/>

![Alt tex1](img/rebase-4.png)

Peccato che, come accade nel mondo reale, i tuoi colleghi nel frattempo abbiano fatto avanzare il ramo `dev` con i loro `commit`

>git checkout dev<br/>
>touch dev1 && git add dev1 && git commit -m "developer 1"<br/>
>touch dev2 && git add dev2 && git commit -m "developer 2"<br/>

![Alt tex1](img/rebase-5.png)

Riconosci questa situazione? È sostanzialmente inevitabile, a causa della natura fortemente non lineare del processo di sviluppo<br/>
`rebase` ti permette di renderla nuovamente lineare. Con

>git checkout sviluppo<br/>
>git rebase dev

chiedi a git "*riapplica tutto il lavoro che ho fatto nel mio ramo come se lo avessi staccato dall'ultimo commit di sviluppo, ma non costringermi a spostare i commit uno per uno con cherry-pick*"

Il risulato è

![Alt tex1](img/rebase-6.png)

Vedi? Gli ultimi 3 `commit` introducono le stesse identiche modifiche che avevi apportato tu nel tuo ramo, ma tutto appare come se tu avessi staccato il ramo dall'ultima versione di `dev`. Di nuovo: apparentemente hai riscritto la storia.

Via via che prenderai la mano con git scoprirai di poter usare `cherry-pick` (ed altri comandi, che spesso sono una sorta di combinazioni di comandi di più basso livello) per manipolare i tuoi `commit` e ottenere risultati che sono letteralmente impossibili con altri sistemi di versionamento: invertire l'ordine di una serie di `commit`, spezzare in due rami separati una singola linea di sviluppo, scambiare `commit` tra un ramo e l'altro, aggiungere un `commit` con un bugfix a metà di un ramo, spezzare un `commit` in due e così via.

Questa versatilità non dovrebbe poi stupirti troppo: alla fine git non è altro che un database chiave/valore e i suoi comandi non sono altro che delle macro per creare oggetti e applicare l'aritmetica dei puntatori.<br/>
Per cui, tutto quel che può venirti in mente di fare con oggetti e puntatori, tendenzialmente, puoi farlo con git.<br/>
Fico, no?

## Obiettivo 5: unire due rami

Passiamo ad un'operazione che farai spessissimo.<br/>
Confronta le ultime due immagini.<br>

![Alt tex1](img/rebase-5-6.png)

Nella prima si vede chiaramente come `sviluppo` non contenga i due contributi `developer 1` e `developer 2` dei tuoi colleghi. Quei due `commit` non sono *raggiungibili* dal tuo ramo. Cioè: percorrendo a ritroso la storia a partire dal tuo ramo `sviluppo` non incontrerai quei due `commit`.

Guarda adesso la seconda immagine, cioè la storia che hai ottenuto dopo il `rebase`: adesso i due `commit` sono *raggiungibili* da `sviluppo`. Beh, avevi fatto `rebase` appositamente per allinearti con il lavoro dei tuoi colleghi quindi, giustamente, git ha fatto in modo che il tuo ramo contenesse anche i loro contributi.

`rebase` e `cherry-pick` non sono i soli strumenti con i quali puoi *integrare* nel tuo ramo il contenuto di altri rami. Anzi: uno degli strumenti che utilizzerai più spesso è `merge`

`merge` funziona come te lo aspetti. Ci sono solo 3 particolarità sulle quali credo valga la pena di soffermarsi. La prima è che il `merge` di git funziona spaventosamente bene. Merito del modello di storage di git: durante i merge git non deve stare ad impazzire, come SVN, per capire se una diff sia già stata applicata o no, perché parte dal confronto di fotografie del progetto. Ma non entriamo nel dettaglio: goditi la potenza di `git merge` e dimentica tutte le difficoltà che hai sempre incontrato con SVN.

Le altre due particolarità sono il `fast-forward` e l'`octopus merge`.

Ma preferisco mostrarteli subito con degli esempi

Stacca un ramo da `dev` e aggiungi un paio di `commit`

>git checkout -b bugfix dev

Nota: qui ho usato una forma super concisa equivalente ai comandi:

> git checkout dev<br/>
> git branch bugfix<br/>
> git checkout bugfix<br/> 

Prosegui aggiungendo i due `commit`

>touch fix1 && git add fix1 && git commit -m "bugfixing  1"<br/>
>touch fix2 && git add fix2 && git commit -m "bugfixing  2"<br/>

![Alt tex1](img/merge-1.png)

Dopo di che, annunci ai tuoi colleghi di aver completato il bugfixing e inviti tutti a integrare il tuo lavoro nel loro.

Per integrare il bugfix in `sviluppo` un tuo collega potrebbe fare

>git checkout sviluppo<br/>
>git merge bugfix

![Alt tex1](img/merge-2.png)

Semplice, non trovi?<br/>
Con `git merge bugfix` hai chiesto a git: "*procurami un `commit` che contenga tutto quello che c'è nel mio `branch` corrente e aggiungici tutte le modifiche introdotte dal ramo `bugfix`*".

Prima di eseguire il merge, git guarda nel suo `blob storage` e cerca se per caso esista già un `commit` contenente entrambi i rami. Dal momento che non lo trova, git lo crea, fonde i due file system e poi assegna come genitori del nuovo `commit` entrambi i `commit` di provenienza.<br/>
In effetti, il risultato è un nuovo `commit` che ha due genitori. Nota anche che l'etichetta del tuo ramo, `sviluppo` si è spostata sul nuovo `commit`. Non dovrebbe essere una sopresa: il `branch` corrente è pensato per seguirti, `commit` dopo `commit`.

### `fast-forward`

Se ti torna questo ragionamento, non avrai difficoltà a capire il `fast-forward`. Mettiti alla prova e vediamo. Prova a rispondere a questa domanda: cosa accadrebbe se ti spostassi sul ramo `dev` e chiedessi un `merge` col ramo `sviluppo` con `git merge sviluppo`?

Per risponderti, ripeti il ragionamento sopra: stai chiedendo a git "*procurami un `commit` che contenga sia il mio ramo corrente `dev` che il ramo `sviluppo`*". git consulterebbe i `commit` nel suo database per asicurarsi che un `commit` con queste caratteristiche sia già presente.

E lo troverebbe!<br/>
Guarda il `commit` puntato proprio dal ramo `sviluppo`: senza dubbio contiene `sviluppo` (per definizione!); e, siccome percorrendo la storia verso il basso da `sviluppo` è possibile raggiungere `dev`, non c'è nemmeno dubbio che `sviluppo` contenga già le modifiche introdotte da `dev`.<br/>
Ti torna?

Quindi, git non ha motivo per creare un nuovo `commit` e si limiterà a spostarvi sopra la tua etichetta corrente.

Prova:

>git checkout dev<br/>
>git merge sviluppo

 
![Alt tex1](img/fast-forward.png)

Prova a confrontare la storia prima e dopo il merge

![Alt tex1](img/fast-forward-2.png)


Vedi cosa è accaduto? Che l'etichetta `dev` è stata *spinta in avanti*.

Ecco: hai visto un caso di `fast-forward`. Tieni a mente questo comportamento: di tanto in tanto capita di averne a che fare, soprattutto quando vuoi evitare che avvenga. Per esempio: il `merge` che hai appena fatto, e che è risultato in un `fast-forward`, ha creato una storia nella quale risulta un po' difficile capire *quando* il ramo `dev` sia stato staccato. Non si vede nemmeno bene quando il `merge` sia stato effettuato, perché manca un `commit` con un commento tipo `merge branch 'dev' into sviluppo`.

`fast-forward` è un argomento cruciale nell'interazione con altri `repository`. Ne parleremo nel paragrafo su `push`.<br/>
Per adesso tieni a mente solo il concetto: 

* un branch può essere mergiato ad un commit in fast-forward quando è possibile spostarcelo semplicemente spingengolo in avanti
* il merge non può essere fast-forward quando il tuo `branch` e il `commit` sul quale vorresti portartlo si trovano su linee di sviluppo separate

Un esempio potrebbe aiutarti a fissare il concetto
 
In questo `repository`, un merge di `bugfix` su  `dev` avverrà in `fast-forward`

![Alt tex1](img/fast-forward.png)

In quest'altro caso, un merge di `sviluppo` su `bugfix` non potrà essere in `fast-forward`, e risulterà in un nuovo `commit`

![Alt tex1](img/merge-1.png)


### `octopus merge`

E per chiudere l'argomento vediamo l'`octopus merge`. Ma ci vorranno pochi secondi, perché è una cosa di una semplicità sconcertante.

Guarda un `commit` nato da un `merge`: non è diverso dagli altri `commit` se non per il fatto di avere due genitori invece di uno solo.


![Alt tex1](img/fast-forward.png)

Ecco: su git il numero di genitori di un `commit` non è limitato a due. In altre parole, puoi mergiare tra loro quanti `branch` vuoi, in un colpo solo.

Guarda. Crea 4 `branch` qualsiasi

>git branch uno<br/>
>git branch due<br/>
>git branch tre<br/>
>git branch quattro<br/>
>git checkout uno && touch uno && git add uno && git commit -m "uno"<br/>
>git checkout due && touch due && git add due&& git commit -m "due"<br/>
>git checkout tre && touch tre&& git add tre && git commit -m "tre"<br/>
>git checkout quattro && touch quattro && git add quattro && git commit -m "e quattro"<br/>

![Alt tex1](img/octopus-1.png)

Bene. Hai 4 rami. Adesso chiedi a `dev` di mergiarli tutti, in un colpo solo

>git checkout dev<br/>
>git merge uno due tre quattro<br/>

![Alt tex1](img/octopus-2.png)

Et voilà! Un `merge` di 4 `branch`.

E ora qualcosa di completamente diverso. Vediamo un po' come si comporta git con i server remoti.


## Obiettivo 6: mettere il `repository` in rete

Fino ad ora hai interagito solo con il tuo `repository` locale, ma ti avevo anticipato che git è un sistema *peer-to-peer*.

In generale, questo significa che il tuo `repository` è un nodo che può entrare a far parte di una rete e scambiare informazioni con altri nodi, cioè con altri `repository`.

A parte il tuo `repository` locale, qualsiasi altro `repository`, non importa che si trovi su GitHub, su un server aziendale o semplicemente in un'altra directory del tuo computer, per git, è un `remote`.

Per collegare il tuo `repository` locale ad un `remote` ti basta fornire a git l'indirizzo del `repository` remoto con il comando `git remote` (naturalmente, devi anche disporre dei permessi di lettura o scrittura sul `remote`)

Per rendere le cose semplici, facciamo un esempio concreto senza stare a coinvolgere server esterni e internet; crea un altro `repository` da qualche parte sul tuo stesso computer

>cd ..<br/>
>mkdir repo-remoto<br/>
>cd repo-remoto<br/>
>git init<br/>


In questo caso, dalla directory del tuo progetto il `repository` remoto sarà raggiungibile tramite `../repo-remoto` o col suo path assoluto.<br/>
Più comunemente, però, avrai a che fare con `repository` remoti raggiungibili, a seconda del protocollo utilizzato, con indirizzi come 

* `https://azienda.com/repositories/cool-project2.git`
* `git@github.com:johncarpenter/mytool.git`.

Per esempio, il `repository` di questa guida ha l'indirizzo

* `git@github.com:arialdomartini/get-git.git`.

Capita molto spesso, anche, che l'accesso ai `remote` richiesta un'autenticazione. In questo caso, di solito, si usano una coppia nome utente/password o una chiave ssh.

Torna nel tuo progetto

>cd ../progetto

Bene. Aggiungi all'elenco dei `remote` il `repository` appena creato, indicando a git un nome qualsiasi e l'indirizzo

>git remote add foobar ../repo-remoto

Ottimo. Hai connesso il tuo `repository` ad un altro nodo. Sei ufficialmente in una rete peer-to-peer di `repository`.<br/>
Da questo momento, quando vuoi riferirti a quel `repository` remoto utilizzerai il nome `foobar`.

Il nome è necessario perché, a differenza di SVN che ha il concetto di "*server centrale*" in git puoi essere collegato ad un numero qualsiasi di`repository` remoti contemporaneamente.

Sono due le cose che fondamentalmente puoi fare con un `remote`: allinearsi al suo contenuto o chiedere che sia lui ad allinearsi a te.

Hai a disposizione due comandi: `push` e `fetch`.

Con `push` puoi *spedire* un set di `commit` al `repository` remoto.<br/>
Con `fetch` puoi *riceverli* dal `repository` remoto

Sia `push` che `fetch`, in realtà, permettono al tuo `repository` e al `remote` di scambiarsi delle etichette. Ma per gradi: iniziamo a vedere in concreto cosa accada.


### Spedire un ramo con `push`

Al momento il `remote` che hai chiamato `foobar` è un `repository` completamente vuoto: lo hai appena creato.<br>
Il tuo `repository` locale, invece, contiene molti `commit` e molti `branch`:


![Alt tex1](img/local-1.png)


Prova a chiedere al `repository` remoto di darti i `commit` e i `branch` di cui dispone e che tu non hai. Se non indichi un `branch` specifico il `repository` remoto cercherà di darteli tutti. Nel tuo caso il `remote` è vuoto, quindi non dovrebbe restituirti nulla

>git fetch foobar

Infatti. Non ricevi nulla.<br/>
Prova, invece, a spedire il ramo `experiment`

>git push foobar experiment<br/>
><br/>
>Counting objects: 14, done.<br/>
>Delta compression using up to 4 threads.<br/>
>Compressing objects: 100% (8/8), done.<br/>
>Writing objects: 100% (14/14), 1.07 KiB | 0 bytes/s, done.<br/>
>Total 14 (delta 3), reused 0 (delta 0)<br/>
>To ../repo-remoto<br/>
>** * [new branch]      experiment -> experiment<br/>**

Wow! Qualcosa è sucesso!<br/>
Di tutti i messaggi di risposta, quello più interessante in questo momento è l'ultimo

>** * [new branch]      experiment -> experiment<br/>**

Ti aiuto a interpretare quello che è successo:

* quando hai scritto `git push foobar experiment` git ha preso in considerazione il tuo ramo `experiment` ed ha ricavato l'elenco di tutti i `commit` raggiunbibili da quel ramo (come al solito: sono tutti i `commit` che puoi trovare partendo da `experiment` e seguendo a ritroso nel tempo qualsiasi percorso tu possa percorrere)
* git ha poi contattato il `repository` remoto `foobar` per sapere quali di quei `commit` non fossero presenti remotamente
* dopo di che, ha creato un pacchetto con tutti i `commit` necessari, li ha inviati  ed ha chiesto al `repository` remoto di aggiungerli al proprio database
* il `remote` ha poi posizionato il proprio `branch` `experiment` perché puntasse esattamente lo stesso `commit` puntato sul tuo `repository` locale. Il `remote` non aveva quel `branch`, per cui lo ha creato.

Proviamo adesso a visualizzare il `repository` remoto


![Alt tex1](img/remote-1.png)

Vedi? Il `remote` non è una copia del tuo `repository`: contiene solo il `branch` che gli hai spedito.

Guarda se torna: prova a prendere in ordine i 4 `commit` e verifica che siano davvero tutti e soli i `commit` che avevi in locale sul ramo `experiment`.<br/>
Sì, sono proprio gli stessi.

Anche sul tuo `repository` locale è successo qualcosa. Prova a visualizzarlo

![Alt tex1](img/push-1.png)

Guarda guarda! Sembra sia stato aggiunto un nuovo `branch`, chiamato `foobar/experiment`. E sembra anche si tratti di un `branch` un po' particolare, perché SmartGit si preoccupa di disegnarlo di colore differente.

Prova a cancellare quel `branch`

>git branch -d foobar/experiment<br/>
>error: branch 'foobar/experiment' not found.<br/>

Uhm. Decisamente quel `branch` ha qualcosa di particolare.

Il fatto è che quel `branch` non è sul tuo `repository`: è su `foobar`. git ha aggiunto un `remote branch` per permetterti di tenere traccia del fatto che, su  `foobar` il `branch` `experiment` punta proprio a quel `commit`.

I `remote branch` sono una sorta di reminder che ti permettono di capire dove si trovino i `branch` sui `repository` remoti ai quali sei collegato.

C'è un aspetto molto importante sulla posizione dei `remote branch` a cui dovrai fare l'abitudine: proprio mentre stavi leggendo queste righe un tuo collega potrebbe aver aggiunto qualche `commit` proprio sul suo ramo `expetiment`, e tu non ne sapresti niente, perché il tuo `repository` non è collegato in tempo reale con i suoi `remote`, ma si sincronizza solo quando ci interagisci con gli appositi comandi. Per cui, il `commit` puntato da `foobar/experiment` è da intendersi come l'ultma posizione nota del ramo `experiment` su `foobar`.

### Ricevere aggiornamenti con `fetch`

Guarda: proviamo proprio a simulare il caso in cui un tuo collega stia lavorando sull'altro `repository`. Prova ad aggiungere un `commit` sul `repository remoto` proprio sul ramo `experiment` di cui hai appena fatto `push`

>cd ../repo-remoto<br/>
>touch x<br/>
>git add x<br/>
>git commit -m "un contributo dal tuo collega"<br/>

Ecco il risultato finale su `foobar`

![Alt tex1](img/push-2.png)

Torna pure al tuo `repository` locale e vediamo cos'è cambiato

>cd ../progetto<br/>

![Alt tex1](img/push-1.png)


Infatti. Non è cambiato niente di niente.<br/>
Il tuo `repository` locale continua a dirti che il ramo `experiment` su `foobar` si trova a "*un commit con un esperimento*". E tu sai benissimo che non è vero! `foobar` è andato avanti, e il tuo `repository` non lo sa. 

Tutto questo è coerente con quel che ti ho detto prima: il tuo `repository` non è collegato in tempo reale con i suo `remote`; ci si allinea solo a comando.

Chiedi allora al tuo `repository` di allinearsi con `foobar`. Puoi chiedere un aggiornamento su un singolo ramo o un aggiornamento su tutti i rami. Di solito, si sceglie la seconda strada

>git fetch foobar<br/>
><br/>
>remote: Counting objects: 3, done.<br/>
>remote: Compressing objects: 100% (2/2), done.<br/>
>remote: Total 2 (delta 1), reused 0 (delta 0)<br/>
>Unpacking objects: 100% (2/2), done.<br/>
>From ../repo-remoto<br/>
>&nbsp;&nbsp;&nbsp;&nbsp;**e5bb7c4..c8528bb  experiment -> foobar/experiment**<br/>

Guarda di nuovo il `repository` locale. (Per renderci la vita più semplice, iniziamo a sfruttare un'opzione ci dui la quasi totalità delle interfacce grafiche di git è provvista: la possibilità di visualizzare un singolo ramo e nascondere tutti gli altri, così da semplificare il risultato finale)

![Alt tex1](img/push-3.png)

Guarda attentamente quello che è successo: il tuo ramo `experiment` non si è spostato di una virgola. Se controlli, anche il tuo `file system` non è cambiato di un solo bit. Solo il tuo `repository` locale è stato aggiornato: git ci ha aggiunto un nuovo `commit`, lo stesso aggiunto remotamente; in concomitanza, git ha anche aggiornato la posizione di `foobar/experiment`, per comunicarti che "*dalle ultime informazioni di cui si dispone, l'ultima posizione registrata su `foobar` del ramo `experiment` è questa*".

Questo è il modo in cui, normalmente, git ti permette di sapere che qualcuno ha proseguito sui lavori del branch `experiment`.

Un'altra osservazione importante: `fetch` non è l'equivalente di `svn update`; solo il tuo `repository` locale si è sincronizzato con quello remoto; il tuo `file system` non è cambiato! Questo significa che, in generale, l'operazione di `fetch` è molto sicura: anche dovessi sincronizzarti con un `repository` di dubbia qualità, puoi dormire sonni tranquilli, perché l'operazione non eseguirà mai il `merge` sul tuo codice senza il tuo esplicito intervento.

Se invece tu volessi davvero includere i cambiamenti introdotti remotamente nel *tuo* lavoro, potresti usare il comando `merge`. 

>git merge foobar/experiment

![Alt tex1](img/push-4.png)

Riconosci il tipo di `merge` che ne è risultato? Sì, un `fast-forward`. Interpretalo così: il tuo `merge` è stato un `fast-forward` perché mentre il tuo collega lavorava il ramo non è stato modificato da nessun altro; il tuo collega è stato il solo ad avervi aggiunto contributi e lo sviluppo è stato lineare.

Possiamo estendere il diagramma delle interazioni tra i comandi di git e i suoi ambienti aggiungendo la colonna `remote` e l'azione di `push` e `fetch`   

![Alt tex1](img/push-fetch.png)


### Divergere

Proviamo a complicare la situazione.<br/>
Vorrei mostrarti un caso che ti capiterà continuamente: il caso in cui due sviluppatori stiano lavorando contemporaneamente su un ramo. Di solito accade che, proprio nel momento in cui vorrai spedire al `remote` i tuoi nuovi `commit`, vieni a scoprire che, nel frattempo, qualcuno ha modificato il `branch`. 

Inizia a simulare l'avanzamento dei lavori del tuo collega, aggiungendo un `commit` sul suo `repository`


>cd ../repo-remoto<br/>
>touch avanzamento && git add avanzamento<br/>
>git commit -m "un nuovo commit del tuo collega"<br/>

![Alt tex1](img/collaborating-1.png)

(En passant, nota una cosa: sul `repository` remoto non c'è alcuna indicazione del tuo `repository`; git è un sistema peer-to-peer asimmetrico)

Torna al tuo `repository`

![Alt tex1](img/push-4.png)

Come prima: fintanto che non chiedi esplicitamente un allineamento con `fetch` il tuo `repository` non sa nulla del nuovo `commit`.

Questa, per inciso, è una delle caratteristiche notevoli di git: essere compatibile con la natura fortemente non lineare delle attività di sviluppo.<br/>
Pensaci: quando due sviluppatori lavorano su un solo branch, SVN richiede che ogni `commit` sia preceduto da un `update`; cioè, che per poter registrare una modifica lo sviluppatore debba integrare preventivamente il lavoro dell'altro sviluppatore.<br/>
git, da questo punto di vista, è meno esigente: gli sviluppatori possono divergere localmente, perfino lavorando sullo stesso `branch`; la decisione se e come integrare il loro lavoro può essere intenzionalmente e indefinitamente spostata avanti nel tempo.

In ogni modo: abbraccia la natura fortemente non lineare di git e, ignorando deliberatamente che potrebbero esserci stati avanzamenti sul `repository` remoto, procedi senza indugio con i tuoi nuovi `commit` in locale


>cd ../progetto<br/>
>touch mio-contributo && git add mio-contributo<br/>
>git commit -m "un mio nuovo commit"<br/>


![Alt tex1](img/collaborating-2.png)

Fai nuovamente caso a quel che ti ho appena descritto:

* il tuo `repository` non sa del nuovo `commit` registrato su `foobar` e continua a vedere una situazione non aggiornata
* a partire dal medesimo `commit` "*un contributo dal tuo collega*" tu e l'altro sviluppatore avete registrato due `commit` completamente indipendenti.

Aver lavorato concorrentemente sullo stesso ramo, con due `commit` potenzialmente incompatibili, se ci pensi, è un po' come lavorare concorrentemente sullo stesso file, con modifiche potenzialmente incompatibili: quando si metteranno insieme i due risultati, c'è da aspettarsi che venga segnalato un conflitto.

E infatti è proprio così. Il conflitto nasce nel momento in cui si cercherà di sincronizzare i due `repository`. Per esempio: prova a spedire il tuo ramo su `foobar`


>git push foobar experiment<br/>
><br/>
>To ../repo-remoto<br/>
>** ! [rejected]**        experiment -> experiment (fetch first)<br/>
>**error: failed to push some refs to '../repo-remoto'**<br/>
>hint: Updates were rejected because the remote contains work that you do<br/>
>hint: not have locally. This is usually caused by another repository pushing<br/>
>hint: to the same ref. You may want to first integrate the remote changes<br/>
>hint: (e.g., 'git pull ...') before pushing again.<br/>
>hint: See the 'Note about fast-forwards' in 'git push --help' for details.<br/>


Rejected. Failed. Error.<br/>
Più che evidente che l'operazione non sia andata a buon fine.<br/>
Ed era prevedibile. Con `git push foobar experiment` avevi chiesto a `foobar` di portare a termine due operazioni:

* salvare nei proprio database tutti i `commit` di cui tu disponi e che remotamente ancora non sono presenti
* spostare la propria etichetta `experiment` in modo che puntasse allo stesso `commit` puntato in locale

Ora: per la prima operazione non ci sarebbe stato alcun problema. Ma per la seconda operazione git pone un vincolo aggiuntivo: il `repository` remoto sposterà la propria etichetta solo a patto che l'operazione si possa concludere con un `fast-forward`, cioè, solo a patto che non ci siano da effettuare dei `merge`. Oppure, detta con altre parole: un `remote` accetta `branch` solo se non creano linee di sviluppo divergenti. 

Il `fast-forward` è citato proprio nell'ultima riga del messaggio di errore

>hint: **See the 'Note about fast-forwards'** in 'git push --help' for details.<br/

Nello stesso messaggio git fornisce un suggerimento: ti dice di provare a fare `fetch`.<br/>
Proviamo

>git fetch foobar

![Alt tex1](img/collaborating-3.png)

La situazione dovrebbe essere chiara già a colpo d'occhio.<br/>
Si vede che le due linee di sviluppo stanno divergendo.<br/>
La posizione dei due rami aiuta a capire dove ti trovi in locale e dove si trovi il tuo collega sul `remote` `foobar`.

Resta solo da decidere cosa fare.<br/>
A differenza di SVN, che di fronte a questa situazione avrebbe richiesto necessariamente un merge in locale, git ti lascia 3 possibilità

* **andare avanti ignorando il collega**: puoi ignorare il lavoro del tuo collega e proseguire lungo la tua linea di sviluppo; certo, non potrai spedire il tuo ramo su `foobar`, perché è incompatibile col lavoro del tuo collega (anche se puoi spedire il tuo lavoro assegnando alla tua linea di sviluppo un altro nome creando un nuovo `branch` e facendo il `push` di quello); comunque, il concetto è che non sei costretto ad integrare il lavoro del tuo collega;
* **`merge`**: puoi fondere il tuo lavoro con quello del tuo collega con un `merge`
* **`rebase`**puoi riallinearti al lavoro del tuo collega con un `rebase`

Prova la terza di queste possibilità.<br/>
Anzi, per insistere sulla natura non lineare di git, prova a far precedere alla terza strada la prima. In altre parole, prova a vedere cosa succede se, temporaneamente, ignori il disallineamento col lavoro del tuo collega e continui a sviluppare sulla tua linea.<br/>
È un caso molto comune: sai di dover riallinearti, prima o poi, col lavoro degli altri, ma vuoi prima completare il tuo lavoro. git non ti detta i tempi e non ti obbliga ad anticipare le cose che non vuoi fare subito

>echo modifica >> mio-contributo<br/>
>git commit -am "avanzo lo stesso"&nbsp;&nbsp;&nbsp;# -a è un'abbreviazione a per `git add`

![Alt tex1](img/collaborating-4.png)

Benissimo. Sei andato avanti col tuo lavoro, disallineandoti ancora di più col lavoro del tuo collega.<br/>
Supponiamo tu decida sia arrivato il momento di allinearsi, per poi spedire il tuo lavoro a `foobar`.

Potresti fare un `git merge foobar/experiment` ed ottenere questa situazione

![Alt tex1](img/collaborating-5.png)

Vedi? Adesso `foobar/experiment` potrebbe essere spinto in avanti (con un `fast-forward`) fino a `experiment`. Per cui, a seguire, potresti fare `git push foobar`.

Ma invece di fare un `merge`, fai qualcosa di più raffinato: usa `rebase`.<br/>
Guarda nuovamente la situazione attuale

![Alt tex1](img/collaborating-3.png)

Rispetto ai lavori su `foobar` è come se tu avessi staccato un ramo di sviluppo ma, disgraziatamente, mentre tu facevi le tue modifiche, `foobar` non ti ha aspettato ed è stato modificato.

Bene: se ricordi, `rebase` ti permette di applicare tutte le tue modifiche ad un altro `commit`; potresti applicare il tuo ramo a `foobar/experiment`. È un po' come se potessi staccare di netto il tuo ramo `experiment` per riattaccarlo su un'altra base (`foobar/experiment`)

Prova

>git rebase foobar/experiment

![Alt tex1](img/collaborating-6.png)

Visto? A tutti gli effetti appare come se tu avessi iniziato il tuo lavoro *dopo* la fine dei lavori su `foobar`.<br/>
In altre parole: `rebase` ha apparentemente reso lineare il processo di sviluppo, che era intrinsecamente non lineare, senza costringerti ad allinearti con il lavoro del tuo collega esattamente nei momenti in cui aggiungeva `commit` al proprio `repository`.

Puoi spedire il tuo lavoro a `foobar`: apparirà come tu abbia apportato le tue modifiche a partire dall'ultimo `commit` eseguito su `foobar`.

>git push foobar experiment<br/>
><br/>
>Counting objects: 6, done.<br/>
>Delta compression using up to 4 threads.<br/>
>Compressing objects: 100% (4/4), done.<br/>
>Writing objects: 100% (5/5), 510 bytes | 0 bytes/s, done.<br/>
>Total 5 (delta 2), reused 0 (delta 0)<br/>
>remote: error: **refusing to update checked out branch: refs/heads/experiment**<br/>
>remote: error: By default, updating the current branch in a non-bare repository<br/>
>remote: error: is denied, because it will make the index and work tree >inconsistent<br/><br/>
>remote: error: with what you pushed, and will require 'git reset --hard' to match<br/>
>remote: error: the work tree to HEAD.<br/>
>remote: error:<br/>
>remote: error: You can set 'receive.denyCurrentBranch' configuration variable to<br/>
>remote: error: 'ignore' or 'warn' in the remote repository to allow pushing into<br/>
>remote: error: its current branch; however, this is not recommended unless you<br/>
>remote: error: arranged to update its work tree to match what you pushed in some<br/>
>remote: error: other way.<br/>
>remote: error:<br/>
>remote: error: To squelch this message and still keep the default behaviour, set<br/>
>remote: error: 'receive.denyCurrentBranch' configuration variable to 'refuse'.<br/>
>To ../repo-remoto<br/>
> ! [remote rejected] experiment -> experiment (branch is currently checked out)<br/>
>error: failed to push some refs to '../repo-remoto'<br/>


Mamma mia! Sembra proprio che a git questo `push` non sia piaciuto. Nel lunghissimo messaggio di errore git ti sta dicendo di non poter fare `push` di un `branch` attualmente "*checked out*": il problema non sembra essere nel `push` in sé, ma nel fatto che sull'altro `repository` il tuo collega abbia fatto `checkout experiment`. 


Questo problema potrebbe capitarti di continuo, se non sai come affrontarlo, per cui a breve gli dedicheremo un po' di tempo.<br>
Per adesso, rimedia chiedendo gentilmente al tuo collega di spostarsi su un altro ramo e ripeti il `push`.

Quindi: su `foobar` vedi di spostarti su un altro `branch`

>cd ../repo-remoto<br/>
>git checkout -b parcheggio<br/>

Dopo di che, torna al tuo `repository` locale e ripeti `push`


>cd ../progetto<br/>
>git push foobar experiment<br/>

Ecco il risultato

![Alt tex1](img/collaborating-7.png)

Ripercorriamo graficamente quello che è successo. Parrivi da

![Alt tex1](img/collaborating-4.png)

Poi hai fatto `rebase` ed hai ottenuto

![Alt tex1](img/collaborating-6.png)

Poi hai fatt `push` su `foobar`: la nuova posizione del `remote branch` `foobar/experiment` testimonia l'avanzamento del ramo anche sul `repository` remoto.

![Alt tex1](img/collaborating-7.png)

Contestualmente, il tuo collega su `foobar` ha visto passare il proprio `repository` da 

![Alt tex1](img/collaborating-1.png)

a

![Alt tex1](img/collaborating-8.png)


Ti torna tutto?<br/>
Ecco, guarda attentamente le ultime due immagini, perché è proprio per evitare quello che vedi che git si è lamentato tanto, quando hai fatto `git push foobar experiment`.<br/>


Mettiti nei panni del tuo collega virtuale, che abbiamo immaginato sul `repository` remoto `foobar`.

Il tuo collega se ne sta tranquillo sul suo ramo `experiment` 

![Alt tex1](img/collaborating-1.png)

quando ad un tratto, senza che abbia dato alcun comando a git, il suo `repository` accetta la tua richiesta di `push`, salva nel database locale un paio di nuovi `commit` e sposta il ramo `experiment` (sì, proprio il ramo di cui aveva fatto il `checkout`!) due `commit` in avanti

![Alt tex1](img/collaborating-8.png)

Ammetterai che se questo fosse il comportamento standard di git non vorresti mai trovarti nella posizione del tuo collega virtuale: la perdita di controllo del proprio `repository` e del proprio `file system` sarebbe davvero un prezzo troppo alto da pagare.

Capisci bene che cambiare il ramo del quale si è fatto `checkout` significa, sostanzialmente, fare il `checkout` di un `commit` differente, quindi cambiare sotto i piedi il `file system`. Ovviamente questo è del tutto inaccettabile, ed è per questo che git si è rifiutato di procedere ed ha replicato con un chilometrico messaggio di errore.

Prima hai rimediato alla situazione spostando il tuo collega virtuale su un ramo `parcheggio`, unicamente per poter spedirgli il tuo ramo. 

![Alt tex1](img/collaborating-9.png)

Questo sporco trucco ti ha permesso di fare `push` di `experiment`.

Ma a pensarci bene anche questa è una soluzione che, probabilmente, tu personalmente non accetteresti mai: a parte la scomodità di doversi interrompere solo perché un collega vuole spedirti del suo codice, comunque non vorresti che l'avanzamento dei tuoi rami fosse completamente fuori dal tuo controllo, alla mercé di chiunque. Perché, alla fine, il remo `experiment` si sposterebbe in avanti contro la tua volontà, e lo stesso potrebbe accadere a tutti gli altri rami di cui non hai fatto `checkout`.

È evidente che debba esistere una soluzione radicale a questo problema.

La soluzione è sorprentemente semplice: **non permette ad altri di accedere al tuo `repository`**. 

Potresti trovarla una soluzione un po' sommaria, ma devi riconoscere che non esista sistema più drastico ed efficace.<br/>
Naturalmente, questa è solo metà della storia e forse vale la pena di approfondire  un po' l'argomento.<br/>
Apri bene la mente, perché adesso entrerai nel vivo di un argomento molto affascinante: la natura distribuita di git. Si tratta, verosimilmente, dell'aspetto più comunemente incompreso di git e, quasi certamente di una delle sue caratteristiche più potenti.

## Obiettivo 7: disegna il tuo workflow ideale










# Daily git


* clone
* git rm
* detached head state
* amend
* eliminare l'ultimo commit
* revert del filesystem
* diff di due branch
* diff del file system
* pull
