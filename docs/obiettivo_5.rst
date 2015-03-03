.. _obiettivo_5:

Obiettivo 5: unire due rami
###########################

Passiamo ad un'operazione che farai spessissimo: il ``merge``. Confronta
le ultime due immagini che abbiamo visto, cioè il tuo ``repository``
prima e dopo il ``rebase``\ 

.. figure:: img/rebase-5-6.png

Nella prima si vede chiaramente come ``sviluppo`` non contenga i due
contributi ``developer 1`` ``developer 2`` dei tuoi colleghi. Quei due
``commit`` non sono *raggiungibili* dal tuo ramo. Cioè: percorrendo a
ritroso la storia a partire dal tuo ramo ``sviluppo`` non attraverserai
quei due ``commit``.

Guarda adesso la seconda immagine, cioè la storia che hai ottenuto dopo
il ``rebase``: adesso i due ``commit`` sono *raggiungibili* da
``sviluppo``.

Ha un senso: avevi fatto ``rebase`` appositamente per allinearti con il
lavoro dei tuoi colleghi quindi, giustamente, git ha fatto in modo che
il tuo ramo *contenesse* anche i loro contributi.

``rebase`` e ``cherry-pick`` non sono i soli strumenti con i quali puoi
*integrare* nel tuo ramo il contenuto di altri rami. Anzi: uno degli
strumenti che utilizzerai più spesso è ``merge``

``merge`` funziona proprio come te lo aspetti: fonde tra loro due
``commit``.

Ci sono solo 3 particolarità sulle quali credo valga la pena di
soffermarsi. La prima è che il ``merge`` di git funziona spaventosamente
bene. Merito del modello di storage di git: durante i merge git non deve
stare ad impazzire, come SVN, per capire se un delta sia già stato
applicato o no, perché parte dal confronto di fotografie del progetto.
Ma non entriamo nel dettaglio: goditi la potenza di ``git merge`` e
dimentica tutte le difficoltà che hai sempre incontrato con SVN.

Le altre due particolarità sono il ``fast-forward`` e
l'\ ``octopus merge``.

Ma preferisco mostrarteli subito con degli esempi

Il ``merge``
============

L'ultima fotografia del tuo ``repository`` è

.. figure:: img/rebase-6.png

Stacca un ramo da ``dev`` e aggiungi un paio di ``commit``

.. code-block:: bash

    git checkout -b bugfix dev

Nota: qui ho usato una forma ancora più concisa equivalente ai comandi:

.. code-block:: bash

    git checkout dev
    git branch bugfix
    git checkout bugfix

Prosegui aggiungendo i due ``commit``

.. code-block:: bash

    touch fix1 && git add fix1 && git commit -m "bugfixing 1"
    touch fix2 && git add fix2 && git commit -m "bugfixing 2"

.. figure:: img/merge-1.png

Benissimo. Hai riprodotto nuovamente una situazione piuttosto comune:
due ``branch``, su due linee di sviluppo divergenti, contenenti entrambi
dei contributi che prima o poi si vogliono integrare.

Supponi, per esempio, che sia tu, una volta completato il tuo lavoro di
bugfixing sull'apposito ramo, a chiedere ai tuoi colleghi di integrare
il tuo lavoro nel loro.

Per integrare il ``bugfix`` in ``sviluppo`` i tuoi colleghi potrebbero
fare

.. code-block:: bash

    git checkout sviluppo
    git merge bugfix

.. figure:: img/merge-2.png

Con ``git merge bugfix`` hai chiesto a git: "*procurami un ``commit``
che contenga tutto quello che c'è nel mio ``branch`` corrente e
aggiungici tutte le modifiche introdotte dal ramo ``bugfix``*\ ".

Prima di eseguire il merge, git guarda nel suo ``Object Database`` e cerca
se per caso esista già un ``commit`` contenente entrambi i rami. Dal
momento che non lo trova, git lo crea, fonde i due file system e poi
assegna come genitori del nuovo ``commit`` entrambi i ``commit`` di
provenienza. In effetti, il risultato è un nuovo ``commit`` che ha due
genitori. Nota anche che l'etichetta del tuo ramo, ``sviluppo`` si è
spostata sul nuovo ``commit``. Non dovrebbe essere una sopresa: il
``branch`` corrente è pensato per seguirti, ``commit`` dopo ``commit``.

Il ``fast-forward merge``
-------------------------

Se ti torna questo ragionamento, non avrai difficoltà a capire il
``fast-forward``. Mettiti alla prova; prova a rispondere a questa
domanda:

Partendo dall'ultimo stato del tuo ``repository``

.. figure:: img/merge-2.png

cosa accadrebbe se ti spostassi sul ramo ``dev`` e chiedessi un
``merge`` col ramo ``sviluppo``, cioè se facessi ``git merge sviluppo``?

Per risponderti, ripeti il ragionamento che abbiamo fatto in occasione
del precedente ``merge``: stai chiedendo a git "*procurami un ``commit``
che contenga sia il mio ramo corrente ``dev`` che il ramo
``sviluppo``*\ ". git consulterebbe i ``commit`` nel suo database per
asicurarsi che un ``commit`` con queste caratteristiche sia già
presente.

E lo troverebbe! Guarda il ``commit`` puntato proprio dal ramo
``sviluppo``: senza dubbio contiene ``sviluppo`` (per definizione!); e,
siccome percorrendo la storia verso il basso da ``sviluppo`` è possibile
raggiungere ``dev``, non c'è nemmeno dubbio che ``sviluppo`` contenga
già le modifiche introdotte da ``dev``. Quindi, quello è il ``commit``
che contiene il ``merge`` tra ``dev`` e ``sviluppo``. Ti torna?

Allora, git non ha motivo per creare un nuovo ``commit`` e si limiterà a
spostarvi sopra la tua etichetta corrente.

Prova:

.. code-block:: bash

    git checkout dev
    git merge sviluppo

.. figure:: img/fast-forward.png

Prova a confrontare la storia prima e dopo il merge

.. figure:: img/fast-forward-2.png

Vedi cosa è accaduto? Che l'etichetta ``dev`` è stata *spinta in
avanti*.

Ecco: hai appena visto un caso di ``fast-forward``. Tieni a mente
questo comportamento: di tanto in tanto capita di averne a che fare,
soprattutto quando vuoi evitare che avvenga. Per esempio, in questa
occasione il ``fast-forward`` non è molto espressivo: si è creata una
storia nella quale risulta un po' difficile capire *quando* il ramo
``dev`` sia stato staccato. Non si vede nemmeno bene quando il ``merge``
sia stato effettuato, perché manca un ``commit`` con un commento tipo
``merge branch 'dev' into sviluppo``.

``fast-forward`` è un argomento cruciale nell'interazione con altri
``repository``. Ne parleremo nel paragrafo su ``push``.

Per adesso cerca solo di tenere a mente il concetto:

-  il ``merge`` di due ``branch`` è eseguito in ``fast-forward`` quando
   è possibile spostare il primo ramo sul secondo semplicemente
   spingengolo in avanti
-  il ``merge`` non può essere ``fast-forward`` quando i due ``branch``
   si trovano su linee di sviluppo divergenti

Un esempio potrebbe aiutarti a fissare il concetto

In questo ``repository``, un merge di ``bugfix`` su ``dev`` avverrà in
``fast-forward``

.. figure:: img/fast-forward.png

In quest'altro caso, un merge di ``sviluppo`` su ``bugfix`` non potrà
essere in ``fast-forward``, e risulterà in un nuovo ``commit``

.. figure:: img/merge-1.png

``octopus merge``
-----------------

E per chiudere l'argomento vediamo l'\ ``octopus merge``. Ma ci vorranno
pochi secondi, perché è una cosa di una semplicità sconcertante.

Guarda un ``commit`` nato da un ``merge``: non è diverso dagli altri
``commit`` se non per il fatto di avere due genitori invece di uno solo.

.. figure:: img/fast-forward.png

Ecco: su git il numero di genitori di un ``commit`` non è limitato a
due. In altre parole, puoi mergiare tra loro quanti ``branch`` vuoi, in
un colpo solo.

Guarda. Crea 4 ``branch`` qualsiasi


.. code-block:: bash

    git branch uno 
    git branch due 
    git branch tre 
    git branch quattro 

    git checkout uno
    touch uno && git add uno && git commit -m "uno" 
    
    git checkout due
    touch due && git add due && git commit -m "due" 
    
    git checkout tre
    touch tre&& git add tre && git commit -m "tre"
    
    git checkout quattro
    touch quattro && git add quattro && git commit -m "e quattro"

.. figure:: img/octopus-1.png

Bene. Hai 4 rami. Adesso chiedi a ``dev`` di mergiarli tutti, in un
colpo solo

.. code-block:: bash

    git checkout dev 
    git merge uno due tre quattro

.. figure:: img/octopus-2.png

Et voilà! Un ``merge`` di 4 ``branch``.

E ora qualcosa di completamente diverso. Vediamo un po' come si comporta
git con i server remoti.

:ref:`Indice <indice>` :: :ref:`Obiettivo 6: mettere il repository in rete <obiettivo_6>`
