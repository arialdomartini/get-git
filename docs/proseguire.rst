.. _proseguire:

########################
Risorse per approfondire
########################

Questa guida ha toccato solo una minima parte degli strumenti messi a disposizione da git.

git è un sistema composto da più di 150  comandi, ognuno dei quali richiederebbe
un capitolo a sé.

Per avere un elenco completo dei comandi disponibili esegui ``git help -a``


.. code-block:: bash

    git help -a
    usage: git [--version] [--help] [-c name=value]
               [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
               [-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
               [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
               <command> [<args>]
    
    available git commands in '/usr/local/Cellar/git/1.8.4/libexec/git-core'
    
      add                       config                    gc                        merge-one-file            relink                    show-ref
      add--interactive          count-objects             get-tar-commit-id         merge-ours                remote                    stage
      am                        credential                grep                      merge-recursive           remote-ext                stash
      annotate                  credential-cache          gui                       merge-resolve             remote-fd                 status
      apply                     credential-cache--daemon  gui--askpass              merge-subtree             remote-ftp                stripspace
      archimport                credential-store          hash-object               merge-tree                remote-ftps               submodule
      archive                   cvsexportcommit           help                      mergetool                 remote-http               svn
      bisect                    cvsimport                 http-backend              mktag                     remote-https              symbolic-ref
      bisect--helper            cvsserver                 http-fetch                mktree                    remote-testsvn            tag
      blame                     daemon                    http-push                 mv                        repack                    tar-tree
      branch                    describe                  imap-send                 name-rev                  replace                   unpack-file
      bundle                    diff                      index-pack                notes                     repo-config               unpack-objects
      cat-file                  diff-files                init                      p4                        request-pull              update-index
      check-attr                diff-index                init-db                   pack-objects              rerere                    update-ref
      check-ignore              diff-tree                 instaweb                  pack-redundant            reset                     update-server-info
      check-mailmap             difftool                  log                       pack-refs                 rev-list                  upload-archive
      check-ref-format          difftool--helper          lost-found                patch-id                  rev-parse                 upload-pack
      checkout                  fast-export               ls-files                  peek-remote               revert                    var
      checkout-index            fast-import               ls-remote                 prune                     rm                        verify-pack
      cherry                    fetch                     ls-tree                   prune-packed              send-email                verify-tag
      cherry-pick               fetch-pack                mailinfo                  pull                      send-pack                 web--browse
      citool                    filter-branch             mailsplit                 push                      sh-i18n--envsubst         whatchanged
      clean                     fmt-merge-msg             merge                     quiltimport               shell                     write-tree
      clone                     for-each-ref              merge-base                read-tree                 shortlog
      column                    format-patch              merge-file                rebase                    show
      commit                    fsck                      merge-index               receive-pack              show-branch
      commit-tree               fsck-objects              merge-octopus             reflog                    show-index
    
    git commands available from elsewhere on your $PATH

      credential-osxkeychain  subtree
    
    'git help -a' and 'git help -g' lists available subcommands and some
    concept guides. See 'git help <command>' or 'git help <concept>'
    to read about a specific subcommand or concept.


Probabilmente, il miglior modo per conoscere i dettagli di ognuno dei comandi 
è leggere la rispettiva *man page*. 

Per accedere alla *man page* di ``merge`` ti basta invocare

.. code-block:: bash

    git help merge


e altrettanto puoi fare per gli altri comandi.

Ma non spaventarti: non avrai bisogno di leggere la documentazione di tutti i comandi:
usa le *man page* come *reference guide* da consultare all'occorrenza, quando avrai
bisogno di dettagli su un comando specifico.


Learn git branching
###################

Piuttosto che buttarti di nuovo in letture chilometriche, io ti suggerisco di passare alla pratica.

Un sistema molto divertente per prendere dimestichezza con git è il meraviglioso `Learn git branching <http://pcottle.github.io/learnGitBranching/?demo>`_,
una guida interattiva molto pratica e molto sfidante, composta da una serie di esercizi
di difficoltà crescente.

Io lo considero un must. 

Affrontala, e sforzati di arrivare fino alla fine: merita tantissimo.

ProGit
######


Una lettura molto più pratica e discorsiva delle *man page* è costituita dal bellissimo ProGit 
di Scott Chacon.

Puoi aquistare il libro `su Amazon <http://www.amazon.com/Pro-Git-Scott-Chacon/dp/1430218339>`_ oppure
puoi leggerlo `gratuitamente online <http://www.git-scm.com/book>`_.

Ti suggerisco caldamente di dedicargli del tempo: è considerato uno tra i migliori testi su git
in circolazione.





:ref:`Indice <indice>`
