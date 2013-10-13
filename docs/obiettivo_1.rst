.. _obiettivo_1:

Obiettivo 1: tornare indietro nel tempo
#######################################

Dunque, se in git tutto è conservato in un database chiave/valore,
probabilmente ci sarà modo per referenziare un qualunque oggetto del
database usando la sua chiave.

In effetti è proprio così.

Adesso proviamo a tornare indietro nel tempo, al ``commit A``,
utilizzando il comando ``git checkout``.

Il comando ``checkout`` prende il ``commit`` indicato e lo copia nel
``file system`` e nella ``staging area``.

.. figure:: img/index-add-commit-checkout.png

Già: ma qual è la chiave del ``commit A``? Lo puoi scoprire con un
client grafico o col comando ``git log`` che mostra un elenco di tutti
i ``commit`` memorizzati nel ``repository``

.. code-block:: bash

    git log --oneline
    2a17c43 Commit B, Il mio secondo commit
    56674fb commit A, il mio primo commit

Attenzione! Siccome nel ``commit`` vengono memorizzati anche la data e
l'autore, le tue chiavi risulteranno diverse dalle mie.

Sul mio ``repository`` la chiave del ``commit A`` è ``56674fb``. Bene:
torniamo indietro al passato, al momento del ``commit`` ``A``

.. code-block:: bash

    ls
    doh.html    libs    templates
    
    git checkout 56674fb
    
    ls
    libs    templates

Effettivamente, a parte un misterioso e prolisso messaggio di con cui
git si lamenta di essere in ``'detached HEAD' state`` (poi chiariremo
questo punto), il file system è tornato allo stato del primo commit e,
infatti, il file ``doh.html`` è scomparso.

:ref:`Indice <indice>` :: :ref:`Obiettivo 2: divergere <obiettivo_2>`
