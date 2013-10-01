= Introduzione =

Questa guida è un po' differente.
Molte delle guide che ho letto su git si preoccupano di introdurti ai comandi base e lasciano ai capitoli più avanzati la descrizione degli internal e del modello di funzionamento.
Quello che ho notato, però, è che imparando git partendo dai comandi base, rischi di finire per valutarlo uno strumento vagamente simile a SVN ma provvisto di un set aggiuntivo di comandi esoterici, il cui funzionamento ti resterà sostanzialmente oscuro.
Facci caso: molti di quelli che hanno imparato git abbasanza da riuscire ad usarlo quotidianamente ti racconteranno di aver fatto molta fatica a capire cosa sia un rebase, o di non cogliere esattamente che uso fare dello stage. 
La mia impressione è che, una volta capito il modello interno (che è stupefacentemente semplice!) tutto git appaia improvvisamente lineare e coerente: non c'è davvero alcun motivo per cui il "rebase" dovrebbe essere un argomento misterioso.


Questa guida prova a spiegarti git seguendo un percorso contrario a quello adottato di solito: partirai da una breve spiegazione degli internal e finirai per imparare, nello stesso momento, sia comandi base che quelli avanzati, in poco tempo e senza troppi grattacapi.


== Non sono parente di SVN ==

Per chi arriva da SVN, git presenta una sola difficoltà: ha molti comandi con gli stessi nomi. Ma è una somiglianza superficiale e ingannevole: sotto il cofano git è totalmente differente.

Per questo ti suggerisco di rifuggire sempre dalla tentazione di fare dei paralleli con SVN, perché sarebbero solo fuorvianti. Troverai comandi come add, checkout, commit, branch che ti sembrerà di conoscere. Ecco: fai tabula rasa di quel che conosci, perché in git quei comandi significano cose molto molto differenti.

Per esempio: ci crederesti che questo repository ha 3 branch?

xxx qui figura

Oppure: ci crederesti che git, più che un sistema di versionamento del codice, potrebbe essere meglio descritto come un "sistema peer-to-peer di database chiave/valore su filesystem"?

Per cui: dimentica quello che sai sui branch e sei changeset di SVN, e preparati a concetti completamente nuovi.
Se siamo fortunati, li troverai molto più omogenei e potenti di quelli di SVN.


== Setup ==

Installa git. xxx qui link.
Poi configuralo perché ti riconosca

  git config --global user.name "Arialdo Martini"
  git config --global user.emal arialdomartini@gmail.com

Se vuoi, installa anche un client grafico. Io ti suggerisco SmartGit, che è gratuito per progetti OpenSource. Altrimenti appoggiati al tool "gitx" che trovi in bundle insieme all'installazione di git.

Fantastico. Partiamo.



== 3 differenze principali ==

Iniziamo con tre caratteristiche di git con le quali dovresti familiarizzare.

1. Non c'è un server: il repository è locale. La gran parte delle operazioni è locale e non richiede l'accesso alla rete. Anche per questo troverai che git sia incredibilmente veloce.
2. git lavora sempre con l'intero codice sorgente del progetto e non su singole directory o su singoli file; con git non c'è differenza tra committare nella directory principale o in una subdirectory. Non esiste il concetto di committare o fare il checkout di un file o di una singola directory. Per git il progetto è l'unità indivisibile di lavoro.
3. git non memorizza i cambiamenti dei file: git salva sempre i file nella loro interezza. Se in un file di 2 mega modificassi un singolo carattere, git memorizzerebbe per intero la nuova versione del file. Questa è una differenza importante: SVN memorizza le differenze e, all'occorrenza, ricostruisce il file; git memorizza il file e, all'occorrenza, ricostruisce le differenze.


== Livelli di nerdosità ==

Sull'assenza di un server ho un po' mentito: come ti ho già detto e come vedrai più avanti, git è un sistema peer-to-peer, e riesce ad interagire con dei server remoti.
Nonostante questo resta, sostanzialmente, un sistema locale.

Per capire quanto questo possa avvantaggiarti, prova a vederla così: quando il codice sorgente di un progetto è ospitato su un computer remoto hai 4 modi per editare il codice

1. lasci tutto il codice sul computer remoto e vi accedi con ssh per editare un singolo file
2. trovi il modo di ottenere una copia locale del singolo file per poterci lavorare più comodamente e lasci tutto il resto sul computer remoto
3. trovi il modo di ottenere una copia locale di un intero albero del filesystem e lasci il resto della storia dei commit sul computer remoto
4. ottenieni una copia locale dell'intero repository con tutta la storia del progetto e lavori tutto in locale

Avrai notato due cose.
La prima, che SVN e i sistemi di versionamento ai quali sei probabilmente abituato operano al livello 3.
La seconda, che i 4 sistemi sono elencati in ordine di comodità: quando il materiale è conservato sul sistema remoto, normalmente, il tuo lavoro è più macchinoso e scomodo. SVN ti permette di fare il checkout di un'intera directory proprio perché così ti risulti più comodo passare da un file all'altro senza dover continuamente interagire col server remoto.
Ecco: git è ancora più estremo; preferisce farti avere tutto a disposizione, sul tuo computer locale; non solo il singolo checkout, ma l'intera storia del progetto, dal primo all'ultimo commit.

Qualunque cosa tu voglia fare, git chiede normalmente di ottenere una copia completa di quel che è presente sul server remoto. Ma non preoccuparti: git è più veloce a ottenere l'intera storia del progetto di quanto SVN lo sia ad ottenere un singolo checkout.


= Modello di storage =

Passiamo dalla terza differenza. 
SVN memorizza i file e la collezione delle varie patch (o diff) applicate nel tempo. 
git invece memorizza i file così come sono, nella loro interezza. All'occorrenza, come vedrai, calcolerà le diff.

Se vuoi evitare tanti grattacapi con git, il miglior suggerimento che tu possa seguire è di trattarlo come un database chiave-valore. 

Mettiamo di avere 2 file vuori sul filesystem 

  mkdir progetto
  cd progetto
  mkdir libs
  touch libs/foo.txt
  mkdir templates
  touch templates/bar.txt

    /
    ├──libs
    |     └──foo.txt
    |
    ├──templates
            └──bar.txt


Decidiamo di gestire il progetto con git

  git init

Aggiungi il primo file a git

  git add libs/foo.txt

Con questo comando, git ispeziona il contenuto del file e lo memorizza nel suo database chiave-valore, che è conservato su filesystem nella directory nascosta .git. Il database si chiama "blob storage"; git associa al file un identificativo unico, la chiave (banalmente: è lo sha1 del file). Il  valore è il contenuto (vuoto) del file. Nel blob storage ci sarà un oggetto Blob, univocamente identificabile dalla chiave, che rappresenta il contenuto del tuo file.

  xxx qui disegno

Adesso aggiungi il secondo file

  git add templates/bar.txt

Ora, siccome libs/foo.txt e templates/bar.txt hanno lo stesso contenuto (sono entrambi vuoti!), nel blob storage entrambi verranno conservati in un unico oggetto, identificato dal suo sha1: xxx

  xxx disegno

Nel Blob git memorizza solo il contenuto del file, non il suo nome né la sua posizione.
Naturalmente, però, a noi il nome dei file e la loro posizione interessano eccome.
Per questo, nel blob storage, git memorizza anche altri oggetti, chiamati "tree" che servono proprio a memorizzare il contenuto delle varie directory e i nomi dei file.

Nel nostro caso, avremo 3 tree 

  xxx qui disegno

Come ogni altro oggetto, anche i tree sono memorizzati come chiave/valore.

Tutte queste strutture vengono raccolte dentro un contenitore, chiamato "commit".

Come avrai intuito, un "commit" è un altro oggetto chiave-valore, la cui chiave è uno SHA1, come per tutti gli altri oggetti, e il cui valore è la collezione dei puntatori ai tree contenuti, cioè l'elenco delle loro chiavi.
Non è troppo complicato, dopo tutto, no?

Quindi, il commit è l'attuale fotografia dello stato del filesystem.

Adesso fai

  git commit -m "commit A, il mio primo commit"

Stai dicendo a git: "memorizza nel repository, cioè nella storia del progetto, il commit che ti ho preparato a colpi di add"
Il tuo repository adesso ha questo aspetto

  xxx qui disegno


== Index o stage == 

Sostanzialmente, non c'è molto altro che tu debba sapere del modello di storage di git e tra pochissimo passiamo a vedere i vari comandi. Ma prima vorrei introdurti ad un altro meccanismo interno: l'"index" o lo "stage". Lo stage risulta sempre misterioso a chi arrivi da SVN: vale la pena parlarne perché quando si sa come funziona il blob storage e lo stage, git passa da sembrare un tool contorto e incomprensibile ad essere un oggetto molto lineare e coerente.


Lo stage è una struttura che fa da cuscinetto tra il filesystem e il repository. È un piccolo buffer che puoi utilizzare per costruire il prossimo commit. 

   xxx qui disegno
    filesystem   |   stage   |   repository

Non è troppo complicato:

 - il file system è la directory con i tuoi file.
 - il repository è il database locale su file, che conserva la storia del filesystem
 - lo stage è lo spazio che git ti mette a disposizione per creare il tuo prossimo commit.

Fisicamente, lo stage è equivalente al repository: entrambi conservano i dati nel blob storage, usando le strutture che hai visto prima.
In questo momento, adesso che hai fatto il tuo primo commit, lo stage conserva una copia del commit che hai appena fatto e si aspetta che tu lo modifichi.

    filesystem   |   stage   |   repository
                        \           \
                         NEXT------> commit A



Proviamo a fare delle modifiche al file system

Sul filesystem hai

    /
    ├──libs
    |     └──foo.txt
    |
    ├──templates
            └──bar.txt

Il tuo Blob storage contiene:

 xxx disegno
   

Lo stage contiene la situazione dalla quale parti, quindi il tuo primo commit, con tutti i suoi tree object e i suoi blob.
Lo stage sta lì, in attesa di accogliere le modifiche che farai sul filesystem.

Modifica "foo.txt"

  echo "un contenuto" >> libs/foo.txt 

e aggiorna lo stage con

  git add libs/foo.txt

All'esecuzione di "git add" git ripete quel che aveva già fatto prima: analizza il contenuto di "libs/foo.txt", vede che c'è un contenuto che non ha mai registrato e quindi aggiunge al Blob storage un nuovo Blob col nuovo contenuto del file; contestualmente, aggiorna il tree "libs" perché il file "foo.txt" punti al suo nuovo contenuto

  xxx disegno


Prosegui aggiungendo un nuovo file "doh.html"

  echo "happy happy joy joy" > doh.html
  git add doh.html

Come prima: git aggiunge un nuovo blob object col contenuto del file e, contestualmente, aggiunge nel tree "/" un nuovo puntatore chiamato "doh.html" che punta al nuovo blob object

  xxx disegno

Il contenitore di tutta questa struttura è un oggetto Commit che git tiene posteggiato nello Stage.
Questa struttura rappresenta esattamente la nuova situazione sul file system.
Siccome a noi interessa anche che git conservi la storia del nostro filesystem, non resta che memorizzare da qualche parte il fatto che questa nuova situazione (lo stato attuale dello stage) sia figlia della precedente situazione (il vecchio commit).

In effetti, git aggiunge automaticamente un'informazione al commit posteggiato nello Stage: un puntatore al commit dal quale si proviene

  commit 1
     ↑
   stage

La freccia rappresenta il fatto che lo stage sia figlio del "commit 1". È un semplice puntatore. Nessuna sopresa, se ci pensi; tutto git, dopo tutto, utilizza il solito, medesimo, semplicissimo modello: un database chiave/valore per conservare il dato, e l'utilizzo delle chiavi come puntatori tra un elemento e l'altro.


Ok. Adesso committa


  git commit -m "Il mio secondo commit!"


Con l'operazione di commit si dice a git "Ok, prendi l'attuale stage e fallo diventare il tuo nuovo commit. Poi restituiscimi lo stage così che possa fare una nuova modifica"


Dopo il commita nel database di git ci ritroviamo

   commit 1
      ↑
   commit 2
      ↑
    stage


Ricapitolando:

1. git memorizza sempre i file nella loro interezza
2. il "commit" è uno dei tanti oggetti conservati dentro il database chiave-valore di git. È un contenitore di tanti puntatori ad altri oggetti del database: i tree che rappresentano directory con nomi di file che a loro volta puntano ad altri tree (sottodirectory) o a dei blob (il contenuto dei file)
3. ogni commit ha un puntatore al commit padre da cui deriva
3. lo stage è uno spazio di appoggio nel quale costruiamo a colpi di "git add" il nuovo commit
4. con "git commit" registriamo l'attuale stage facendolo diventare il nuovo commit.



Bene: adesso hai tutta la teoria per capire i concetti più astrusi di git come il rebase, il cherrypick, l'octopus-merge, il rebase interattivo, il revert e il reset.
Passiamo al pratico.



== I comandi di git ==

=== Obiettivo 1 : creare un repository e due commit ===

Crea da qualche parte una directory e lancia "git init"

  cd /tmp
  mkdir progetto
  cd progetto
  git init

 

La directory "progetto" è diventata un repository git. Fisicamente il repository è conservato in una directory nascosta, chiamata ".git", dentro "progetto"

  ls -A
    .git

Dentro ".git" c'è lo stage, il blog storage e un sacco di altra roba. Ma non avrai mai bisogno di accederci. Sappi solo che quello *è* il repository sul quale lavorerai.

Creiamo un file e committiamolo

  echo inizio > file.txt           # creo il file
  git add file.txt                 # aggiungo il file allo stage
  git commit -m "inizio"           # committo lo stage


|   file system    |    stage       |   repository
             ---add--->         ---commit--->


Sul repository si crea il commit A

    A

Proseguiamo modificando il file

  echo nuova riga >> file.txt
  git add file.txt
  git commit -m "cambiamento B"

Sul repository si crea il commit B figlio del commit A

   A--B

=== Obiettivo 2: tornare indietro nel tempo ===

Ricordi che in git tutto è conservato in un database chiave/valore?
Puoi usare la chiave per referenziare un qualunque oggetto.
Adesso proviamo a tornare indietro nel tempo, al commit A, utilizzando il comando "checkout".

|   file system    |    stage       |   repository
             ---add--->         ---commit--->
             <------------checkout-----------

Il comando checkout prende il commit indicato e lo copia nel filesystem e nello stage.
Già: ma qual è la chiave del commit A?
Lo scopriamo col comando "git log" che mostra tutto quello che abbiamo fatto fin'ora

  git log  --oneline
  00c6637 cambiamento B
  621f0a8 inizio


Ottimo! La chiave del commit A è "621f0a8". Uhm, un po' scomodo come sistema.
Comunque: torniamo indietro al passato, al momento del commit A

  git checkout 621f0a8 
  cat file.txt
    inizio

effettivamente, il file.txt è tornato allo stato del primo commit.



=== Obiettivo 3: creare un branch in modalità detached ===

Potremmo rappresentare la nostra situazione attuale con

  A*---B

Cioè: ci sono due commit, A e B. Il commit B è figlio di A. Noi al momento siamo sul commit A.
Che succederebbe se adesso facessimo qualche modifica e committassimo?
Accadrebbe che divergeremmo dalla linea A---B e di fatto divergeremmo.
Cioè, si creerebbe questa situazione

  A---B
   \
    C*

Cioè: avremmo creato un branch. Proviamo


  echo foobar > altrofile.txt
  git add altrofile.txt
  git commit -m "creo un branch"
  git log --graph --all --oneline

   * b8b1146 creo un branch
   | * 00c6637 cambiamento B
   |/
   * 621f0a8 inizio

Hai ottenuto un branch, senza il meccanismo della copia che utilizza SVN: il modello a chiave/valore e puntatori di git rende molto economico rappresentare un branch; semplicemente, basta memorizzare che i commit C e B abbiano come padre comune il commit A

  A---B
   \
    C

Due osservazioni importanti. 
La prima per ribadire il concetto chd git non ha mai memorizzato i "diff" tra i file. A, B e C sono snapshot dell'intero progetto. È molto importante ricordarselo, perché ti aiuterà a capire che tutte le considerazioni che sei sempre stato abituato a fare con SVN qui non valgono.
La seconda è un po' sorprendente: i rami che hai appena visto non sono "i branch" di git. In git i rami sono delle etichette. Te ne parlerò a brevissimo, ma abituati giù a ripeterti: in git i branch non sono rami di sviluppo.


=== Obiettivo 3: creare un branch  ===

Con il comando checkout hai imparato a spostarti da un commit all'altro

  git log --graph --all --oneline

   * b8b1146 creo un branch
   | * 00c6637 cambiamento B
   |/
   * 621f0a8 inizio

   git checkout 621f0a8
   git checkout b8b1146 
   git checkout 00c6637

Sì, però, ammettiamolo: gestire i commit A, B e C dovendoli chiamare b8b1146, 00c6637 e 621f0a8 è di una scomodità unica.
Per fortuna git fornisce uno strumento molto comodo.
Il modello è sempre lo stesso: un puntatore ad un elemento del database chiave/valore.
Immagina di avere a disposizione una variabile, con un certo nome, nella quale conservare la chiave di un certo commit.
Praticamente, avresti qualcosa di simile ad un'etichetta da applicare ad un commit per poterlo chiamare con un nome invece che con la sua chiave

   * b8b1146 creo un branch
   | * 00c6637 cambiamento B  <== master
   |/
   * 621f0a8 inizio

git aggiunge da solo 2 etichette: HEAD e master.
HEAD è l'etichetta che punta al commit corrente, quello che si sta visualizzando nel proprio filesystem.
L'etichetta "master", invece, è aggiunta automaticamente da git alla creazione del repository.
In questo momento "master" punta al commit 00c6637

A---B master


Per cui, per andare sul commit 00c6637 puoi eseguire

  git checkout master


Ora attento: queste etichette in git si chiamano "branch". Ripetiti mille volte: un branch in git non è un ramo, è un'etichetta che punta ad un commit. Tanti comportamenti di git che appaiono assurdi e complicati diventano molto semplici se eviti di pensare ai branch di git come ad un equivalente dei branch di SVN.





* torno ad A e cremo un branch
* checkout
* etichette

* il merge
* il cherrypick

* rebase
* eliminare un commit sbagliato

* p2p
* fetch
* push
