.. _obiettivo_2:

Obiettivo 2: divergere
######################

Usando una convenzione grafica molto comune nella letteratura su git,
potremmo rappresentare la situazione attuale del tuo repository con

    **A**---B

Cioè: ci sono due ``commit``, ``A`` e ``B``. Il ``commit B`` è figlio di
``A`` (il tempo scorre verso destra). Il ``commit`` in grassetto indica
il punto in cui ti trovi attualmente.

Che succederebbe se adesso facessi qualche modifica e committassi?
Accadrebbe che il nuovo ``commit C`` che andresti a generare sarebbe
figlio di ``A`` (perché è da lì che parti), ma la linea di svilupppo
proseguirebbe divergendo dalla linea ``A---B``.

Cioè, si creerebbe questa situazione

.. code-block:: bash

    A---B
     \
      C     
Provalo davvero:

.. code-block:: bash

    echo "ei fu siccome immobile" > README.md
    git add README.md 
    git commit -m "Ecco il commit C"

.. figure:: img/repo1.png

Hai ottenuto una diramazione, senza ricorrere al meccanismo di copia dei
file utilizzato da SVN al momento deella creazione di un branch: il
modello a chiave/valore e puntatori di git rende molto economico
rappresentare una linea di sviluppo che diverge.

Due osservazioni importanti.

La prima per ribadire il concetto che git non ha mai memorizzato i delta
tra i file: ``A``, ``B`` e ``C`` sono snapshot dell'intero progetto. È
molto importante ricordarselo, perché ti aiuterà a capire che tutte le
considerazioni che sei sempre stato abituato a fare con SVN in git
potrebbero non valere.

La seconda potrebbe un po' sorprenderti: le due linee di sviluppo
divergenti che hai appena visto non sono ``branch``. In git i rami sono
dei puntatori dotati di nome, o delle etichette. Te ne parlerò nel
prossimo paragrafo, ma abituati già a ripeterti: in git i ``branch`` non
sono rami di sviluppo.

:ref:`Indice <indice>` :: :ref:`Obiettivo 3: creare un branch <obiettivo_3>`
