# Introduzione

Questa guida è un po' diversa dalle altre.

Molte delle guide che ho letto su git si preoccupano di introdurti ai comandi base e lasciano ai capitoli più avanzati la descrizione del modello di funzionamento interno.

Quello che ho notato, però, è che imparando git partendo dai comandi base, rischi di finire per trovarlo uno strumento vagamente simile a SVN ma provvisto di un set aggiuntivo di comandi esoterici, il cui funzionamento ti resterà sostanzialmente oscuro.

Facci caso: alcuni di quelli che hanno imparato git abbasanza da riuscire ad usarlo quotidianamente ti racconteranno di aver fatto molta fatica a capire cosa sia un `rebase`, o di non cogliere esattamente che uso fare dello `stage`. 

La mia impressione è che, una volta capito il modello interno (che è stupefacentemente semplice!), tutto git appaia improvvisamente lineare e coerente: non c'è davvero alcun motivo per cui il `rebase` dovrebbe essere un argomento misterioso.


Questa guida prova a spiegarti git seguendo un percorso contrario a quello adottato di solito: partirai dalla spiegazione degli internal e finirai per imparare, nello stesso momento, sia comandi base che quelli avanzati, in poco tempo e senza troppi grattacapi.


# Non sono parente di SVN

Per chi arrivi da SVN, git presenta una sola difficoltà: ha molti comandi identici. Ma è una somiglianza superficiale e ingannevole: sotto il cofano git è totalmente differente.

Per questo ti suggerisco di rifuggire sempre dalla tentazione di fare dei paralleli con SVN, perché sarebbero solo fuorvianti. Troverai comandi come `add`, `checkout`, `commit` e `branch` che ti sembrerà di conoscere. Ecco: fai tabula rasa di quel che conosci, perché in git quei comandi significano cose molto molto differenti.

Cercare di capire git usando SVN come modello, a volte, porta semplicemente fuori strada.

Per esempio: ci crederesti che questo repository ha 3 branch?

![Alt text](img/3-branches.png)

Sì: 3 branch, non 2.

Oppure: ci crederesti che git, più che un sistema di versionamento del codice, potrebbe essere meglio descritto come un "*sistema peer-to-peer di database chiave/valore su file system*"?

Per cui: dimentica quello che sai sui branch e sui changeset di SVN, e preparati a concetti completamente nuovi.

Se siamo fortunati, li troverai molto più omogenei e potenti di quelli di SVN.


## Setup

Installa [git](http://git-scm.com/downloads).

Poi configuralo perché ti riconosca

> git config --global user.name "Arialdo Martini"<br/>
> git config --global user.emal arialdomartini@gmail.com`

Se vuoi, installa anche un client grafico. Io ti suggerisco [SmartGit](http://www.syntevo.com/smartgithg/), che è gratuito per progetti OpenSource. Altrimenti appoggiati al tool `gitx` che trovi in bundle insieme all'installazione di git.

Fantastico. Partiamo.


# Gli internal di git
## 3 differenze principali

Iniziamo con tre caratteristiche di git con le quali dovresti familiarizzare.



1. **Non c'è un server**: il repository è locale. La gran parte delle operazioni è locale e non richiede l'accesso alla rete. Anche per questo troverai git  incredibilmente veloce.
2. **Il progetto è indivisibile**: git lavora sempre con l'intero codice sorgente del progetto e non su singole directory o su singoli file; con git non c'è differenza tra committare nella directory principale o in una sotto-directory. Non esiste il concetto di fare il checkout di un file o di una singola directory. Per git il progetto è l'unità indivisibile di lavoro.
3. **git non memorizza i cambiamenti dei file**: git salva sempre i file nella loro interezza. Se in un file di 2 mega modificassi un singolo carattere, git memorizzerebbe per intero la nuova versione del file. Questa è una differenza importante: SVN memorizza le differenze e, all'occorrenza, ricostruisce il file; git memorizza il file e, all'occorrenza, ricostruisce le differenze.


### 4 livelli di nerdosità

Sull'assenza di un server ho un po' mentito: come ti ho già detto e come vedrai più avanti, git è un sistema peer-to-peer, e riesce ad interagire con dei server remoti.
Nonostante questo resta sostanzialmente un sistema locale.

Per capire quanto questo possa avvantaggiarti, prova a vederla così: quando il codice sorgente di un progetto è ospitato su un computer remoto hai 4 modi per editare il codice

1. Lasci tutto il codice sul computer remoto e vi accedi con ssh per editare un singolo file
2. Trovi il modo di ottenere una copia del singolo file per poterci lavorare più comodamente in locale e lasci tutto il resto sul computer remoto
3. Trovi il modo di ottenere una copia locale di un intero albero del file system e lasci il resto della storia dei commit sul computer remoto
4. Ottenieni una copia locale dell'intero repository con tutta la storia del progetto e lavori in locale

Avrai notato due cose.

La prima, che SVN e i sistemi di versionamento ai quali sei probabilmente abituato operano al livello 3.

La seconda, che i 4 sistemi sono elencati in ordine di comodità: quando il materiale è conservato sul sistema remoto, normalmente, il tuo lavoro è più macchinoso, lento e scomodo. SVN ti permette di fare il checkout di un'intera directory proprio perché così ti risulti più comodo passare da un file all'altro senza dover continuamente interagire col server remoto.

Ecco: git è ancora più estremo; preferisce farti avere a disposizione tutto sul tuo computer locale; non solo il singolo checkout, ma l'intera storia del progetto, dal primo all'ultimo commit.

In effetti, qualunque cosa tu voglia fare, git chiede normalmente di ottenere una copia completa di quel che è presente sul server remoto. Ma non preoccuparti troppo: git è più veloce a ottenere l'intera storia del progetto di quanto SVN lo sia ad ottenere un singolo checkout.


### Il modello di storage

Passiamo dalla terza differenza. E preparati a conoscere il vero motivo per cui git sta sostituendo molto velocemente SVN come nuovo standard *de-facto*.

* SVN memorizza la collezione delle varie patch (o diff) applicate nel tempo ai file; all'occorrenza ricostruisce lo stato attuale;
* git memorizza i file così come sono, nella loro interezza; all'occorrenza ne calcola le diff.

Se vuoi evitare tanti grattacapi con git, il miglior suggerimento che tu possa seguire è di trattarlo come un **database chiave/valore**. 

Apri una console e vediamolo nel concreto.

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

Siccome il `blob-storage` è un database chiave valore, git cercherà di calcolare una chiave ed un valore per il file che hai aggiunto. Per il valore git userà il contenuto stesso del file; per la chiave, verrà calcolato lo sha1 del contenuto (se sei curioso, nel caso di un file vuoto vale `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`)

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


Come avrai intuito, un `commit` non è altro che un elemento del database chiave/valore, la cui chiave è uno SHA1, come per tutti gli altri oggetti, e il cui valore è un puntatore al `tree` del progetto, cioè la sua chiave (più un altro po' di informazioni, come il commento e l'autore).<br/>
Non è troppo complicato, dopo tutto, no?

Quindi, il `commit` è l'attuale fotografia del file system.

Adesso fai

> git commit -m "commit A, il mio primo commit"

Stai dicendo a git: 

*memorizza nel repository, cioè nella storia del progetto, il commit che ti ho preparato a colpi di add*

Il tuo repository, visto da SmartGit, adesso ha questo aspetto

![Alt text](img/first-commit.png)

La riga col pallino che vedi sulla sinistra rappresenta l'oggetto `commit`. Nel pannello sulla destra, invece, puoi vedere la chiave del `commit`.

In generale, a meno che non si debba parlare proprio del modello interno come stiamo facendo adesso, non c'è una grande necessità di rappresentare tutta la struttura di `blob` e `tree` che costituisce un `commit`. Difatti, dopo il prossimo paragrafo inizieremo a rappresentare i `commit` come nella figura qui sopra: con un semplice pallino.

Già da adesso, comunque, dovrebbe risultarti più chiaro il fatto che dentro un `commit` ci sia l'intera fotografia del progetto e che, di fatto, un `commit` sia l'unità minima ed indivisibile di lavoro.


## L' `index` o `staging area` 

Sostanzialmente, non c'è molto altro che tu debba sapere del modello di storage di git. Ma prima di passare a vedere i vari comandi, vorrei introdurti ad un altro meccanismo interno: la `staging area` o `index`. L'`index` risulta sempre misterioso a chi arrivi da SVN: vale la pena parlarne perché quando si sa come funzionano il `blob storage` e l'`index`, git passa da sembrare un tool contorto e incomprensibile ad essere un oggetto molto lineare e coerente.


L'`index` è una struttura che fa da cuscinetto tra il file system e il repository. È un piccolo buffer che puoi utilizzare per costruire il prossimo `commit`. 

![Alt text](img/index1.png)

Non è troppo complicato:

 * il `file system` è la directory con i tuoi file.
 * il `repository` è il database locale su file, che conserva i vari `commit`
 * l'`index` è lo spazio che git ti mette a disposizione per creare il tuo prossimo `commit`.

Fisicamente, l'`index` non è molto diverso dal `repository`: entrambi conservano i dati nel `blob storage`, usando le strutture che hai visto prima.

In questo momento, appena dopo aver completato il tuo primo `commit`, l'`index` conserva una copia del commit e si aspetta che tu lo modifichi.

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

All'esecuzione di `git add` git ripete quel che aveva già fatto prima: analizza il contenuto di `libs/foo.txt`, vede che c'è un contenuto che non ha mai registrato e quindi aggiunge al `blob storage` un nuovo `blob` col nuovo contenuto del file; contestualmente, aggiorna il `tree` `libs` perché il file `foo.txt` punti al suo nuovo contenuto

![Alt tex1](img/index3.png)

Prosegui aggiungendo un nuovo file `doh.html` alla root del progetto

>  echo "happy happy joy joy" > doh.html<br/>
>  git add doh.html

Come prima: git aggiunge un nuovo `blob` object col contenuto del file e, contestualmente, aggiunge nel `tree` "/" un nuovo puntatore chiamato `doh.html` che punta al nuovo `blob` object

![Alt tex1](img/index4.png)

Il contenitore di tutta questa struttura è un oggetto `commit` che git tiene parcheggiato nella `staging area`.
Questa struttura rappresenta esattamente la nuova situazione sul file system.

Siccome però a noi interessa che git conservi anche la storia del nostro file system, ci sarà bisogno di memorizzare da qualche parte il fatto che questa nuova situazione (lo stato attuale dell'`index`) sia figlia della precedente situazione (il precedente `commit`).

In effetti, git aggiunge automaticamente al `commit` posteggiato nella `staging area` un puntatore al `commit` dal quale si proviene

![Alt tex1](img/index-and-first-commit.png)

La freccia rappresenta il fatto che l'`index` è figlio del `commit A`. È un semplice puntatore. Nessuna sopresa, se ci pensi; git, dopo tutto, utilizza il solito, medesimo, semplicissimo modello ovunque: un database chiave/valore per conservare il dato, e una chiave come puntatore tra un elemento e l'altro.


Ok. Adesso committa


>  git commit -m "Commit B, Il mio secondo commit"


Con l'operazione di commit si dice a git "*Ok, prendi l'attuale `index` e falla diventare il tuo nuovo commit. Poi restituiscimi l'`index` così che possa fare una nuova modifica*"


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

Il comando `checkout` prende il commit indicato e lo copia nel file system e nella `staging area`.

![Alt tex1](img/index-add-commit-checkout.png)

Già: ma qual è la chiave del `commit A`?
Lo scopriamo col comando `git log` che mostra tutto quello che abbiamo fatto fin'ora

> git log --oneline<br/>
>**2a17c43** Commit B, Il mio secondo commit<br/>
>**56674fb** commit A, il mio primo commit<br/>

Occhio! Siccome nel commit vengono memorizzati anche la data e l'autore, le tue chiavi risulteranno diverse dalle mie.

La chiave del `commit A` è `56674fb`. Uhm, un po' scomodo come sistema.<br/>
Comunque: torniamo indietro al passato, al momento del commit A

> ls<br/>
> **doh.html&nbsp;&nbsp;&nbsp;&nbsp;libs&nbsp;&nbsp;&nbsp;&nbsp;templates**<br/>
> git checkout 56674fb<br/>
> ls<br/>
> **libs&nbsp;&nbsp;&nbsp;&nbsp;templates**<br/>

Effettivamente, a parte un misterioso e prolisso messaggio di con cui git si lamenta di essere in `'detached HEAD' state` (poi chiariremo questo punto), il file system è tornato allo stato del primo commit e, infatti, il file `doh.html` è scomparso.



##Obiettivo 2: divergere

Usando una convenzione grafica molto comune nella letteratura su git, potremmo rappresentare la situazione attuale del tuo repository con

> **A**---B

Cioè: ci sono due `commit`, `A` e `B`. Il `commit B` è figlio di `A` (il tempo scorre verso destra). Il `commit` in grassetto indica il punto dove ti trovi attualmente.


Che succederebbe se adesso facessi qualche modifica e committassi?<br/>
Accadrebbe che il nuovo `commit C` che andresti a generare sarebbe figlio di `A` (perché è da lì che parti), ma la linea di svilupppo proseguirebbe divergendo dalla linea `A---B`.

Cioè, si creerebbe questa situazione

>  A---B<br/>
>  &nbsp;&nbsp;\ <br/>
>  &nbsp;&nbsp;&nbsp;**C**

Proviamolo davvero:

>  echo "ei fu siccome immobile" > README.md<br/>
>  git add README.md<br/>
>  git commit -m "Ecco il commit C"<br/>

![Alt tex1](img/repo1.png)


Hai ottenuto una diramazione, senza il meccanismo della copia utilizzato da SVN: il modello a chiave/valore e puntatori di git rende molto economico rappresentare una linea di sviluppo che diverge.

Due osservazioni importanti.
 
La prima per ribadire il concetto che git non ha mai memorizzato i "diff" tra i file: `A`, `B` e `C` sono snapshot dell'intero progetto. È molto importante ricordarselo, perché ti aiuterà a capire che tutte le considerazioni che sei sempre stato abituato a fare con SVN qui non valgono.

La seconda è un po' sorprendente: le due linee di sviluppo divergenti che hai appena visto non sono `branch`. In git i rami sono dei puntatori dotati di nome, o delle etichette. Te ne parlerò nel prossimo paragrafo, ma abituati già a ripeterti: in git i `branch` non sono rami di sviluppo.


# Obiettivo 3: creare un branch

Con il comando `checkout` hai imparato a spostarti da un `commit` all'altro

![Alt tex1](img/repo1.png)

Basta conoscere la chiave di ogni `commit`
> git log --oneline --all<br/>
>**deaddd3** Ecco il commit C<br/>
>**2a17c43** Commit B, Il mio secondo commit<br/>
>**56674fb** commit A, il mio primo commit<br/>
><br/>
>git checkout **56674fb** # vai al `commit A`<br/>
>git checkout **2a17c43** # vai al `commit B`<br/>
>git checkout **300c737** # vai al `commit C`<br/>

Sì, però, bisogna ammetterlo: gestire i `commit` `A`, `B` e `C` dovendoli chiamare `56674fb`, `2a17c43` e `deaddd3` è di una scomodità unica.

git risolve il problema facendo quello che farebbe ogni programmatore di buon senso: dal momento che quei numeri sono dei puntatori ad oggetti, git permette di salvarli in delle variabili.

git crea di default alcune di queste variabili. Per esempio, in questo momento la variabile `master` contiene il puntatore a `B`.

Lo puoi verificare chiaramente dalla rappresentazione grafica:

![Alt tex1](img/repo2.png)


Vedi che c'è un'etichetta chiamata `master` proprio in corrispondenza del `commit B`? Ecco: quell'etichetta ti permette di andare sul quel `commit` scrivendo:

> git checkout master

Ora attento, perché siamo di nuovo in una di quelle occasioni dove la conoscenza di SVN fornisce solo dei grattacapi: queste etichette in git si chiamano `branch`. Ripetiti mille volte: un `branch` in git non è un ramo, è un'etichetta, un puntatore ad un commit, una variabile valorizzata alla chiave di un `commit`. Tanti comportamenti di git che appaiono assurdi e complicati diventano molto semplici se eviti di pensare ai `branch` di git come ad un equivalente dei branch di SVN.

Dovrebbe iniziare a risultarti chiaro perché molti dicano che "*i branch su git sono molto economici*": per forza! Sono delle semplicissime variabili!

Divertiti ad aggiungerne altre

>git branch dev

![Alt tex1](img/branch-dev.png)

Voilà: hai aggiunto un `branch`.

Nota un'altra cosa: vedi che accanto a `master` SmartGit aggiunge un segnaposto triangolare verde? Quel simbolo indica che in questo momento sei *agganciato* al `branch` `master`, perché il tuo ultimo comando di spostamento è stato `git checkout master`.

Potresti spostarti su `dev` con

>git checkout dev
a
![Alt tex1](img/branch-dev2.png)

Di default git aggiunge sempre una variabile: il puntatore `HEAD`, che punta sempre all'elemento del `repository` sul quale ti trovi. Sostanzialmente, il segnaposto visualizzato da SmartGit indica la posizione di HEAD. Altri editor grafici utilizzano differenti rappresentazioni. `gitk`, per esempio, visualizza in grassetto il `branch` sul quale ti trovi.<br/>
Per sapere su quale `branch` ti trovi, dalla linea di comando, ti basta eseguire

>git branch<br/>
>* dev<br/>
>  master<br/>

L'asterisco suggerisce che `HEAD` adesso stia puntanto a `dev`.

Non dovresti essere troppo sorpreso nel verificare che, nonostante tu abbia cambiato `branch` da `master` a `dev` il tuo `file system` non sia cambiato di una virgola: in effetti, sia `dev` che `master` stanno puntando allo stesso identico `commit`.

Non di meno, ti domanderai probabilmente a cosa mai possa servire passare da un `branch` all'altro, se non sortisce alcun effetto visibile.

Il fatto è che quando esegui il `checkout` di `branch`, in qualche modo ti *agganci* all'etichetta; l'etichetta del `branch`, in altre parole, inizierà a seguirti, `commit` dopo `commit`.

Guarda: adesso sei su `dev`. Apporta una modifica qualsiasi e committa

>touch style.css<br/>
>git add style.css<br/>
>git commit -m "Adesso ho anche il css"<br/>

![Alt tex1](img/branch-dev3.png)

Visto cosa è successo? L'etichetta `dev` si è spostata in avanti e si è agganciata al tuo nuovo `commit`.

Ti domanderai anche perché mai git chiami quelle etichette `branch`. Il motivo è che, anche se le linee di sviluppo che divergono in git non sono `branch`, i `branch` vengono normalmente usati proprio per dar loro un nome.

Guardalo nel concreto. Torna a `master` ed apporta qualche modifica.

>git checkout master<br/>
>touch jquery.js<br/>
>git add angular.js<br/>
>git commit -m "angular.js rocks" 

![Alt tex1](img/angular.png)




* il merge
* il cherrypick

* rebase
* eliminare un commit sbagliato

* p2p
* fetch
* push
