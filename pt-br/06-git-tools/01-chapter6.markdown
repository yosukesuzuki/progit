Até aqui, você aprendeu a maioria dos comandos e fluxos de trabalho do dia-a-dia que você precisa para gerenciar ou manter um repositório Git para o controle de seu código fonte. Você concluiu as tarefas básicas de rastreamento e commit de arquivos, e você aproveitou o poder da área de seleção e branches tópicos e merges.
    $ git log
    commit 734713bc047d87bf7eac9674765ae793478c50d3
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Fri Jan 2 18:32:33 2009 -0800
        fixed refs handling, added gc auto, updated tests
    commit d921970aadf03b3cf0e71becdaab3147ba71cdef
    Merge: 1c002dd... 35cfb2b...
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Dec 11 15:08:43 2008 -0800
        Merge commit 'phedders/rdocs'
    commit 1c002dd4b536e7479fe34593e72e6c6c1819e53b
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Dec 11 14:58:32 2008 -0800
        added some blame and merge stuff
    $ git show 1c002dd4b536e7479fe34593e72e6c6c1819e53b
    $ git show 1c002dd4b536e7479f
    $ git show 1c002d
Git pode descobrir uma abreviação curta, única para seus valores SHA-1. Se você passasr a opção `--abbrev-commit` para o comando `git log`, a saída irá usar valores curtos mas os mantém únicos; por padrão ele usa sete caracteres mas, usa mais se necessário para manter o SHA-1 não ambíguo:
    $ git log --abbrev-commit --pretty=oneline
    ca82a6d changed the verison number
    085bb3b removed unnecessary test code
    a11bef0 first commit
    $ git show ca82a6dff817ec66f44342007202690a93763949
    $ git show topic1
Se você quer ver para qual SHA específico um branch aponta, ou se você quer ver o que qualquer desses exemplos se resumem em termos de SHAs, você pode usar uma ferramenta de Git plumbing chamada `rev-parse`. Você pode ver o *Capítulo 9* para mais informações sobre ferramentas de plumbing (canalização); basicamente, `rev-parse` existe para operações de baixo nível e não é projetada para ser usada em operações do dia-a-dia. Entretanto, ela as vezes pode ser útil quando você precisa ver o que realmente está acontecendo. Aqui você pode executar `rev-parse` no seu branch.
    $ git rev-parse topic1
    ca82a6dff817ec66f44342007202690a93763949
    $ git reflog
    734713b... HEAD@{0}: commit: fixed refs handling, added gc auto, updated
    d921970... HEAD@{1}: merge phedders/rdocs: Merge made by recursive.
    1c002dd... HEAD@{2}: commit: added some blame and merge stuff
    1c36188... HEAD@{3}: rebase -i (squash): updating HEAD
    95df984... HEAD@{4}: commit: # This is a combination of two commits.
    1c36188... HEAD@{5}: rebase -i (squash): updating HEAD
    7e05da5... HEAD@{6}: rebase -i (pick): updating HEAD
    $ git show HEAD@{5}
    $ git show master@{yesterday}
    $ git log -g master
    commit 734713bc047d87bf7eac9674765ae793478c50d3
    Reflog: master@{0} (Scott Chacon <schacon@gmail.com>)
    Reflog message: commit: fixed refs handling, added gc auto, updated
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Fri Jan 2 18:32:33 2009 -0800
        fixed refs handling, added gc auto, updated tests
    commit d921970aadf03b3cf0e71becdaab3147ba71cdef
    Reflog: master@{1} (Scott Chacon <schacon@gmail.com>)
    Reflog message: merge phedders/rdocs: Merge made by recursive.
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Dec 11 15:08:43 2008 -0800
        Merge commit 'phedders/rdocs'
A outra principal maneira de especificar um commit é através de seu ancestral. Se você colocar um `^` no final da referência, Git interpreta isso como sendo o pai do commit. Suponha que você veja o histórico do seu projeto:
    $ git log --pretty=format:'%h %s' --graph
    * 734713b fixed refs handling, added gc auto, updated tests
    *   d921970 Merge commit 'phedders/rdocs'
    |\
    | * 35cfb2b Some rdoc changes
    * | 1c002dd added some blame and merge stuff
    |/
    * 1c36188 ignore *.gem
    * 9b29157 add open3_detach to gemspec file list
    $ git show HEAD^
    commit d921970aadf03b3cf0e71becdaab3147ba71cdef
    Merge: 1c002dd... 35cfb2b...
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Dec 11 15:08:43 2008 -0800
        Merge commit 'phedders/rdocs'
Você também pode informar um número depois do `^` — por exemplo, `d921970^2` significa "o segundo pai de d921970." Essa sintaxe só é útil para commits com merge, que têm mais de um pai. O primeiro pai é o branch que você estava quando fez o merge, e o segundo é o commit no branch que você fez o merge:
    $ git show d921970^
    commit 1c002dd4b536e7479fe34593e72e6c6c1819e53b
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Dec 11 14:58:32 2008 -0800
        added some blame and merge stuff
    $ git show d921970^2
    commit 35cfb2b795a55793d7cc56a6cc2060b4bb732548
    Author: Paul Hedderly <paul+git@mjr.org>
    Date:   Wed Dec 10 22:22:03 2008 +0000
        Some rdoc changes
    $ git show HEAD~3
    commit 1c3618887afb5fbcbea25b7c013f4e2114448b8d
    Author: Tom Preston-Werner <tom@mojombo.com>
    Date:   Fri Nov 7 13:47:59 2008 -0500
        ignore *.gem
    $ git show HEAD^^^
    commit 1c3618887afb5fbcbea25b7c013f4e2114448b8d
    Author: Tom Preston-Werner <tom@mojombo.com>
    Date:   Fri Nov 7 13:47:59 2008 -0500
        ignore *.gem
Agora que você pode especificar commits individuais, vamos ver como especificar intervalos de commits. Isso é particularmente útil para gerenciar seus branches — se você tem muitos branches, você pode usar especificações de intervalos para responder perguntas como, "Que modificações existem nesse branch que ainda não foram mescladas (merge) no meu branch principal?".
A especificação de intervalo mais comum é a sintaxe de ponto-duplo. Isso basicamente pede ao Git para encontrar um intervalo de commits que é acessível a partir de um commit, mas não são acessíveis a partir de outro. Por exemplo, digamos que você tem um histórico de commits como a Figure 6-1.
Insert 18333fig0601.png
Figura 6-1. Exemplo de histórico de seleção de intervalo.
Você quer ver o que existe no seu branch mas não existe no branch master. Você pede ao Git para mostrar um log apenas desses commits com `master..experiment` — isso significa "todos os commits acessíveis por experiment que não são acessíveis por master." Para deixar os exemplos mais breves e claros, vou usar as letras dos objetos dos commits do diagrama no lugar da saída real do log na ordem que eles seriam mostrados:
    $ git log master..experiment
    D
    C
    $ git log experiment..master
    F
    E
    $ git log origin/master..HEAD
Esse comando lhe mostra qualquer commit no seu branch atual que não está no branch `master` no seu remoto `origin`. Se você executar um `git push` e seu branch atual está rastreando `origin/master`, os commits listados por `git log origin/master..HEAD` são os commits que serão transferidos para o servidor. Você também pode não informar um lado da sintaxe que o Git assumirá ser HEAD. Por exemplo, você pode obter os mesmos resultados que no exemplo anterior digitando `git log origin/master..` — Git substitui por HEAD o lado que está faltando.
A sintaxe ponto-duplo é útil como um atalho; mas talvez você queira especificar mais de dois branches para indicar suar revisão, como ver quais commits estão em qualquer um dos branches mas não estão no branch que você está atualmente. Git permite que você faça isso usando o caractere `^` ou `--not` antes de qualquer referência a partir do qual você não quer ver commits acessíveis. Assim, estes três comandos são equivalentes:
    $ git log refA..refB
    $ git log ^refA refB
    $ git log refB --not refA
    $ git log refA refB ^refC
    $ git log refA refB --not refC
Este é um sistema de consulta de revisão muito poderoso que deve ajudá-lo a descobrir o que existe nos seus branches.
A última grande sintaxe de intervalo de seleção é a sintaxe ponto-triplo, que especifica todos os commits que são acessíveis por qualquer uma das duas referências mas não por ambas. Veja novamente o exemplo de histórico de commits na Figure 6-1. Se você quer ver o que tem em `master` ou `experiment` mas sem referências comuns, você pode executar
    $ git log master...experiment
    F
    E
    D
    C
    $ git log --left-right master...experiment
    < F
    < E
    > D
    > C
Com essas ferramentas, você pode informar ao Git mais facilmente qual ou quais commits você quer inspecionar.
Git vem com alguns scripts que facilitam algumas tarefas de linha de comando. Aqui, você verá alguns comandos interativos que podem ajudar você a facilmente escolher combinações ou partes de arquivos para incorporar em um commit. Essas ferramentas são muito úteis se você modificou vários arquivos e decidiu que quer essas modificações em commits separados em vez de um grande e bagunçado commit. Desta maneira, você pode ter certeza que seus commits estão logicamente separados e podem ser facilmente revisados pelos outros desenvolvedores trabalhando com você. Se você executar `git add` com a opção `-i` ou `--interactive`, Git entra em um modo interativo de shell, exibindo algo desse tipo:
    $ git add -i
               staged     unstaged path
      1:    unchanged        +0/-1 TODO
      2:    unchanged        +1/-1 index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    *** Commands ***
      1: status     2: update      3: revert     4: add untracked
      5: patch      6: diff        7: quit       8: help
    What now>
Você pode ver que esse comando lhe mostra uma visão muito diferente da sua área de seleção — basicamente a mesma informação que você recebe com `git status` mas um pouco mais sucinto e informativo. Ele lista as modificações que você colocou na área de seleção à esquerda e as modificações que estão fora à direita.
    What now> 2
               staged     unstaged path
      1:    unchanged        +0/-1 TODO
      2:    unchanged        +1/-1 index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    Update>>
    Update>> 1,2
               staged     unstaged path
    * 1:    unchanged        +0/-1 TODO
    * 2:    unchanged        +1/-1 index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    Update>>
    Update>>
    updated 2 paths
    *** Commands ***
      1: status     2: update      3: revert     4: add untracked
      5: patch      6: diff        7: quit       8: help
    What now> 1
               staged     unstaged path
      1:        +0/-1      nothing TODO
      2:        +1/-1      nothing index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    *** Commands ***
      1: status     2: update      3: revert     4: add untracked
      5: patch      6: diff        7: quit       8: help
    What now> 3
               staged     unstaged path
      1:        +0/-1      nothing TODO
      2:        +1/-1      nothing index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    Revert>> 1
               staged     unstaged path
    * 1:        +0/-1      nothing TODO
      2:        +1/-1      nothing index.html
      3:    unchanged        +5/-1 lib/simplegit.rb
    Revert>> [enter]
    reverted one path
    *** Commands ***
      1: status     2: update      3: revert     4: add untracked
      5: patch      6: diff        7: quit       8: help
    What now> 1
               staged     unstaged path
      1:    unchanged        +0/-1 TODO
      2:        +1/-1      nothing index.html
      3:    unchanged        +5/-1 lib/simplegit.rb

Para ver o diff do que você adicionou, você pode usar o comando `6` ou `d` (para diff). Ele exibe uma lista dos seus arquivos adicionados, e você pode selecionar aqueles que você gostaria de ver a diferença. Isso é muito parecido com informar o comando `git diff --cached`:

    *** Commands ***
      1: status     2: update      3: revert     4: add untracked
      5: patch      6: diff        7: quit       8: help
    What now> 6
               staged     unstaged path
      1:        +1/-1      nothing index.html
    Review diff>> 1
    diff --git a/index.html b/index.html
    index 4d07108..4335f49 100644
    --- a/index.html
    +++ b/index.html
    @@ -16,7 +16,7 @@ Date Finder

     <p id="out">...</p>

    -<div id="footer">contact : support@github.com</div>
    +<div id="footer">contact : email.support@github.com</div>

     <script type="text/javascript">
No Git também é possível adicionar certas partes de arquivos e deixar o resto de fora. Por exemplo, se você faz duas mudanças no seu arquivo simplegit.rb e quer adicionar uma delas mas não a outra, fazer isso é muito fácil no Git. A partir do prompt interativo, digite `5` ou `p` (para patch). Git irá perguntar quais arquivos você gostaria de adicionar parcialmente; então, para cada seção dos arquivos selecionados, ele irá exibir partes do diff do arquivo e perguntar se você gostaria de adicioná-los, um por um:
    diff --git a/lib/simplegit.rb b/lib/simplegit.rb
    index dd5ecc4..57399e0 100644
    --- a/lib/simplegit.rb
    +++ b/lib/simplegit.rb
    @@ -22,7 +22,7 @@ class SimpleGit
       end
       def log(treeish = 'master')
    -    command("git log -n 25 #{treeish}")
    +    command("git log -n 30 #{treeish}")
       end
       def blame(path)
    Stage this hunk [y,n,a,d,/,j,J,g,e,?]?
    Stage this hunk [y,n,a,d,/,j,J,g,e,?]? ?
    y - stage this hunk
    n - do not stage this hunk
    a - stage this and all the remaining hunks in the file
    d - do not stage this hunk nor any of the remaining hunks in the file
    g - select a hunk to go to
    / - search for a hunk matching the given regex
    j - leave this hunk undecided, see next undecided hunk
    J - leave this hunk undecided, see next hunk
    k - leave this hunk undecided, see previous undecided hunk
    K - leave this hunk undecided, see previous hunk
    s - split the current hunk into smaller hunks
    e - manually edit the current hunk
    ? - print help
    What now> 1
               staged     unstaged path
      1:    unchanged        +0/-1 TODO
      2:        +1/-1      nothing index.html
      3:        +1/-1        +4/-0 lib/simplegit.rb
Finalmente, você não precisa estar no modo interativo para adicionar um arquivo parcialmente — você pode executar o mesmo script usando `git add -p` ou `git add --patch` na linha de comando.
    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #      modified:   index.html
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #
    #      modified:   lib/simplegit.rb
    #
    $ git stash
    Saved working directory and index state \
      "WIP on master: 049d078 added the index file"
    HEAD is now at 049d078 added the index file
    (To restore them type "git stash apply")
    $ git status
    # On branch master
    nothing to commit (working directory clean)
    $ git stash list
    stash@{0}: WIP on master: 049d078 added the index file
    stash@{1}: WIP on master: c264051... Revert "added file_size"
    stash@{2}: WIP on master: 21d80a5... added number to log
Nesse caso, duas stashes tinham sido feitas anteriormente, então você tem acesso a três trabalhos stashed diferentes. Você pode aplicar aquele que acabou de fazer o stash com o comando mostrado na saída de ajuda do comando stash original: `git stash apply`. Se você quer aplicar um dos stashes mais antigos, você pode especificá-lo, assim: `git stash apply stash@{2}`. Se você não especificar um stash, Git assume que é o stash mais recente e tenta aplicá-lo:
    $ git stash apply
    # On branch master
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #
    #      modified:   index.html
    #      modified:   lib/simplegit.rb
    #
    $ git stash apply --index
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #      modified:   index.html
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #
    #      modified:   lib/simplegit.rb
    #
    $ git stash list
    stash@{0}: WIP on master: 049d078 added the index file
    stash@{1}: WIP on master: c264051... Revert "added file_size"
    stash@{2}: WIP on master: 21d80a5... added number to log
    $ git stash drop stash@{0}
    Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)
    $ git stash branch testchanges
    Switched to a new branch "testchanges"
    # On branch testchanges
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #      modified:   index.html
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #
    #      modified:   lib/simplegit.rb
    #
    Dropped refs/stash@{0} (f0dfc4d5dc332d1cee34a634182e168c4efc3359)
Muitas vezes, trabalhando com o Git, você pode querer revisar seu histórico de commits por alguma razão. Uma das melhores funcionalidades do Git é que ele permite você tomar decisões no último momento possível. Você pode decidir quais arquivos vai em qual commit um pouco antes de fazer o commit da área de seleção, você pode decidir que não quer trabalhar em alguma coisa ainda com o comando stash, e você pode reescrever commits que já aconteceram para que eles pareçam ter acontecido de outra maneira. Isso pode envolver mudar a ordem dos commits, alterar mensagens ou arquivos em um commit, juntar ou separar commits, ou remover commits completamente — tudo isso antes de compartilhar seu trabalho com os outros.
Nesta seção, você verá como realizar essas tarefas muito úteis de modo que você pode fazer seu histórico de commits parecer da forma que você quiser antes de compartilhá-lo com outros.
    $ git commit --amend
Isso abre seu editor de texto, com sua última mensagem de commit nele, pronto para você modificar a mensagem. Quando você salva e fecha o editor, ele salva um novo commit contendo essa mensagem e fazendo esse seu novo commit o mais recente.
Para modificar um commit mais antigo em seu histórico, você deve usar ferramentas mais complexas. Git não tem uma ferramenta de modificação de histórico, mas você pode usar o rebase para alterar uma série de commits no HEAD onde eles estavam originalmente em vez de movê-los para um novo. Com a ferramenta de rebase interativo, você pode parar depois de cada commit que quer modificar e alterar a mensagem, adicionar arquivos, ou fazer o que quiser. Você pode executar o rebase de forma interativa adicionando a opção `-i` em `git rebase`. Você deve indicar quão longe você quer reescrever os commits informando ao comando qual commit quer fazer o rebase.
    $ git rebase -i HEAD~3
Executando esse comando dará a você uma lista de commits no seu editor de texto que se parece com isso:
    pick f7f3f6d changed my name a bit
    pick 310154e updated README formatting and added blame
    pick a5f4a0d added cat-file
    # Rebase 710f0f8..a5f4a0d onto 710f0f8
    #
    # Commands:
    #  p, pick = use commit
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    # However, if you remove everything, the rebase will be aborted.
    #
É importante notar que esses commits são listados na ordem inversa do que você normalmente vê usando o comando `log`. Se você executar um `log`, você vê algo como isto:
    $ git log --pretty=format:"%h %s" HEAD~3..HEAD
    a5f4a0d added cat-file
    310154e updated README formatting and added blame
    f7f3f6d changed my name a bit
Você precisa editar o script para que ele pare no commit que você deseja editar. Para fazer isso, mude a palavra "pick" para a palavra "edit" para cada um dos commits que você deseja que o script pare. Por exemplo, para alterar somente a terceira mensagem de commit, você altera o arquivo para ficar assim:
    edit f7f3f6d changed my name a bit
    pick 310154e updated README formatting and added blame
    pick a5f4a0d added cat-file
    $ git rebase -i HEAD~3
    Stopped at 7482e0d... updated the gemspec to hopefully work better
    You can amend the commit now, with
           git commit --amend
    Once you’re satisfied with your changes, run
           git rebase --continue
    $ git commit --amend
    $ git rebase --continue
Esse comando irá aplicar os outros dois commits automaticamente, e pronto. Se você alterar "pick" para "edit" em mais linhas, você pode repetir esses passos para cada commit que mudar para "edit". Cada vez, Git irá parar, permitir que você altere o commit, e continuar quando você tiver terminado.
    pick f7f3f6d changed my name a bit
    pick 310154e updated README formatting and added blame
    pick a5f4a0d added cat-file
    pick 310154e updated README formatting and added blame
    pick f7f3f6d changed my name a bit
    #
    # Commands:
    #  p, pick = use commit
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    # However, if you remove everything, the rebase will be aborted.
    #
Se, em vez de "pick" ou "edit", você especifica "squash", Git modifica essa e a alteração imediatamente anterior a ela e faz com que você faça o merge das mensagens de commits. Então, se você quer fazer um único commit a partir desses três commits, você modifica o script para algo como isso:
    pick f7f3f6d changed my name a bit
    squash 310154e updated README formatting and added blame
    squash a5f4a0d added cat-file
    # This is a combination of 3 commits.
    # The first commit's message is:
    changed my name a bit
    # This is the 2nd commit message:
    updated README formatting and added blame
    # This is the 3rd commit message:
    added cat-file
Quando você salvar isso, você terá um único commit com as alterações dos três commits anteriores.
Dividir um commit significa desfazer um commit e parcialmente adicionar a área de seleção e commits dependendo do número de commits que você quer. Por exemplo, digamos que você quer dividir o commit do meio daqueles seus três commits. Em vez de "updated README formatting and added blame", você quer dividí-lo em dois commits: "updated README formatting" no primeiro, e "added blame" no segundo. Você pode fazer isso no script `rebase -i` mudando a instrução para "edit" no commit que você quer dividir:
    pick f7f3f6d changed my name a bit
    edit 310154e updated README formatting and added blame
    pick a5f4a0d added cat-file
Depois, quando o script colocar retornar para a linha de comando, você faz o reset desse commit, pega as alterações desse reset, e cria vários commits delas. Quando você salvar e sai do editor, Git volta ao pai do primeiro commit da sua lista, altera o primeiro commit (`f7f3f6d`), altera o segundo (`310154e`), e retorna você ao console. Lá, você pode fazer um reset desse commit com `git reset HEAD^`, que efetivamente desfaz o commit e deixa os arquivos alterados fora da área de seleção. Agora você pode colocar na área de seleção e fazer vários commits, e executar `git rebase --continue` quando estiver pronto:
    $ git reset HEAD^
    $ git add README
    $ git commit -m 'updated README formatting'
    $ git add lib/simplegit.rb
    $ git commit -m 'added blame'
    $ git rebase --continue
    $ git log -4 --pretty=format:"%h %s"
    1c002dd added cat-file
    9b29157 added blame
    35cfb2b updated README formatting
    f3cc40e changed my name a bit
Mais uma vez, isso altera os SHAs de todos os commits na sua lista, então certifique-se que você não fez o push de nenhum commit dessa lista para um repositório compartilhado.
    $ git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
    Rewrite 6b9b3cf04e7c5686a9cb838c3f36a8cb6a0fc2bd (21/21)
    Ref 'refs/heads/master' was rewritten
Digamos que você importou arquivos de outro sistema de controle de versão e ele tem subdiretórios que não fazem sentido (trunk, tags, e outros). Se você quer fazer o subdiretório `trunk` ser a nova raiz do projeto para todos os commits, `filter-branch` pode ajudar a fazer isso, também:
    $ git filter-branch --subdirectory-filter trunk HEAD
    Rewrite 856f0bf61e41a27326cdae8f09fe708d679f596f (12/12)
    Ref 'refs/heads/master' was rewritten
Agora a sua nova raiz do projeto é o que estava no subdiretório `trunk`. Git também apagará automaticamente os commits que não afetaram o subdiretório.
    $ git filter-branch --commit-filter '
            if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
            then
                    GIT_AUTHOR_NAME="Scott Chacon";
                    GIT_AUTHOR_EMAIL="schacon@example.com";
                    git commit-tree "$@";
            else
                    git commit-tree "$@";
            fi' HEAD
    $ git blame -L 12,22 simplegit.rb
    ^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 12)  def show(tree = 'master')
    ^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 13)   command("git show #{tree}")
    ^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 14)  end
    ^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 15)
    9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 16)  def log(tree = 'master')
    79eaf55d (Scott Chacon  2008-04-06 10:15:08 -0700 17)   command("git log #{tree}")
    9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 18)  end
    9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 19)
    42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 20)  def blame(path)
    42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 21)   command("git blame #{path}")
    42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 22)  end
    $ git blame -C -L 141,153 GITPackUpload.m
    f344f58d GITServerHandler.m (Scott 2009-01-04 141)
    f344f58d GITServerHandler.m (Scott 2009-01-04 142) - (void) gatherObjectShasFromC
    f344f58d GITServerHandler.m (Scott 2009-01-04 143) {
    70befddd GITServerHandler.m (Scott 2009-03-22 144)         //NSLog(@"GATHER COMMI
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 145)
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 146)         NSString *parentSha;
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 147)         GITCommit *commit = [g
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 148)
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 149)         //NSLog(@"GATHER COMMI
    ad11ac80 GITPackUpload.m    (Scott 2009-03-24 150)
    56ef2caf GITServerHandler.m (Scott 2009-01-05 151)         if(commit) {
    56ef2caf GITServerHandler.m (Scott 2009-01-05 152)                 [refDict setOb
    56ef2caf GITServerHandler.m (Scott 2009-01-05 153)

Isto é realmente útil. Normalmente, você recebe como commit original, o commit de onde o código foi copiado, porque essa foi a primeira vez que você mecheu nessas linhas do arquivo. Git lhe informa o commit original onde você escreveu aquelas linhas, mesmo que seja em outro arquivo.
Anotar um arquivo ajuda se você sabe onde está o problema. Se você não sabe o que está o problema, e houveram dezenas ou centenas de commits desde a última vez que você sabe que o código estava funcionando, você provavelmente vai usar `git bisect` para ajudá-lo. O comando `bisect` faz uma pesquisa binária em seu histórico de commits para ajudar você a indentificar o mais rápido possível qual commit inseriu o erro.
Digamos que você acabou de enviar seu código para um ambiente de produção, você recebe relatos de erros sobre algo que não estava acontecendo no seu ambiente de desenvolvimento, e você não tem ideia do motivo do código estar fazendo isso. Você volta para seu código e consegue reproduzir o problema, mas não consegue descobrir o que está errado. Você pode usar o "bisect" para descobrir. Primeiro você executa `git bisect start` para começar, e depois você usa `git bisect bad` para informar ao sistema que o commit atual está quebrado. Em seguida, você deve informar ao "bisect" quando foi a última vez que estava correto, usando `git bisect good [good_commit]`:
    $ git bisect start
    $ git bisect bad
    $ git bisect good v1.0
    Bisecting: 6 revisions left to test after this
    [ecb6e1bc347ccecc5f9350d878ce677feb13d3b2] error handling on repo
Git descobre que cerca de 12 commits estão entre o commit que você informou como o commit correto (v1.0) e a versão atual incorreta, e ele faz o um check out do commit do meio para você. Neste momento, você pode executar seus testes para ver se o problema existe neste commit. Se existir, então ele foi inserido em algum momento antes desse commit do meio; se não existir, então o problema foi inserido algum momento após o commit do meio. Acontece que não há nenhum problema aqui, e você informa isso ao Git digitando `git bisect good` e continua sua jornada:
    $ git bisect good
    Bisecting: 3 revisions left to test after this
    [b047b02ea83310a70fd603dc8cd7a6cd13d15c04] secure this thing
    $ git bisect bad
    Bisecting: 1 revisions left to test after this
    [f71ce38690acf49c1f3c9bea38e09d82a5ce6014] drop exceptions table
    $ git bisect good
    b047b02ea83310a70fd603dc8cd7a6cd13d15c04 is first bad commit
    commit b047b02ea83310a70fd603dc8cd7a6cd13d15c04
    Author: PJ Hyett <pjhyett@example.com>
    Date:   Tue Jan 27 14:48:32 2009 -0800
        secure this thing
    :040000 040000 40ee3e7821b895e52c1695092db9bdc4c61d1730
    f24d3c6ebcfc639b1a3814550e62d60b8e68a8e4 M  config
Quando você terminar, você deve executar `git bisect reset` para fazer o "reset" do seu HEAD para onde você estava antes de começar, ou você vai acabar em uma situação estranha:
    $ git bisect reset
    $ git bisect start HEAD v1.0
    $ git bisect run test-error.sh
    $ git submodule add git://github.com/chneukirchen/rack.git rack
    Initialized empty Git repository in /opt/subtest/rack/.git/
    remote: Counting objects: 3181, done.
    remote: Compressing objects: 100% (1534/1534), done.
    remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
    Receiving objects: 100% (3181/3181), 675.42 KiB | 422 KiB/s, done.
    Resolving deltas: 100% (1951/1951), done.

Agora você tem um projeto do Rack no subdiretório `rack` dentro do seu projeto. Você pode ir nesse subdiretório, fazer alterações, adicionar seus próprios repositórios remotos para fazer o push de suas modificações, fazer o fetch e o merge do repositório original, e outras coisas. Se você execurar `git status` logo depois de adicionar o submódulo, você verá duas coisas:

    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #      new file:   .gitmodules
    #      new file:   rack
    #
    $ cat .gitmodules
    [submodule "rack"]
          path = rack
          url = git://github.com/chneukirchen/rack.git
    $ git diff --cached rack
    diff --git a/rack b/rack
    new file mode 160000
    index 0000000..08d709f
    --- /dev/null
    +++ b/rack
    @@ -0,0 +1 @@
    +Subproject commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433
Esse é um ponto importante sobre submódulos: você os salva como o commit exato onde eles estão. Você não pode salvar um submódulo no `master` ou em outra referência simbólica.
    $ git commit -m 'first commit with submodule rack'
    [master 0550271] first commit with submodule rack
     2 files changed, 4 insertions(+), 0 deletions(-)
     create mode 100644 .gitmodules
     create mode 160000 rack
Note o modo 160000 para a entrada do rack. Esse é um modo especial no Git que basicamente significa que você está salvando um commit como um diretório em vez de um subdiretório ou um arquivo.
Você pode tratar o diretório `rack` como um projeto separado e atualizar seu projeto-pai de vez em quando com uma referência para o último commit nesse subprojeto. Todos os comandos do Git funcionam independente nos dois diretórios:
    $ git log -1
    commit 0550271328a0038865aad6331e620cd7238601bb
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Apr 9 09:03:56 2009 -0700
        first commit with submodule rack
    $ cd rack/
    $ git log -1
    commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433
    Author: Christian Neukirchen <chneukirchen@gmail.com>
    Date:   Wed Mar 25 14:49:04 2009 +0100
        Document version change
    $ git clone git://github.com/schacon/myproject.git
    Initialized empty Git repository in /opt/myproject/.git/
    remote: Counting objects: 6, done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 6 (delta 0), reused 0 (delta 0)
    Receiving objects: 100% (6/6), done.
    $ cd myproject
    $ ls -l
    total 8
    -rw-r--r--  1 schacon  admin   3 Apr  9 09:11 README
    drwxr-xr-x  2 schacon  admin  68 Apr  9 09:11 rack
    $ ls rack/
    $
    $ git submodule init
    Submodule 'rack' (git://github.com/chneukirchen/rack.git) registered for path 'rack'
    $ git submodule update
    Initialized empty Git repository in /opt/myproject/rack/.git/
    remote: Counting objects: 3181, done.
    remote: Compressing objects: 100% (1534/1534), done.
    remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
    Receiving objects: 100% (3181/3181), 675.42 KiB | 173 KiB/s, done.
    Resolving deltas: 100% (1951/1951), done.
    Submodule path 'rack': checked out '08d709f78b8c5b0fbeb7821e37fa53e69afcf433'
    $ git merge origin/master
    Updating 0550271..85a3eee
    Fast forward
     rack |    2 +-
     1 files changed, 1 insertions(+), 1 deletions(-)
    [master*]$ git status
    # On branch master
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #      modified:   rack
    #
    $ git diff
    diff --git a/rack b/rack
    index 6c5e70b..08d709f 160000
    --- a/rack
    +++ b/rack
    @@ -1 +1 @@
    -Subproject commit 6c5e70b984a60b3cecd395edd5b48a7575bf58e0
    +Subproject commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433

A causa disso é que a referência que você tem para o submódulo não é exatamente o que está no diretório do submódulo. Para corrigir isso, você precisa executar `git submodule update` novamente:

    $ git submodule update
    remote: Counting objects: 5, done.
    remote: Compressing objects: 100% (3/3), done.
    remote: Total 3 (delta 1), reused 2 (delta 0)
    Unpacking objects: 100% (3/3), done.
    From git@github.com:schacon/rack
       08d709f..6c5e70b  master     -> origin/master
    Submodule path 'rack': checked out '6c5e70b984a60b3cecd395edd5b48a7575bf58e0'
    $ git submodule update
    fatal: reference isn’t a tree: 6c5e70b984a60b3cecd395edd5b48a7575bf58e0
    Unable to checkout '6c5e70b984a60b3cecd395edd5ba7575bf58e0' in submodule path 'rack'
    $ git log -1 rack
    commit 85a3eee996800fcfa91e2119372dd4172bf76678
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Thu Apr 9 09:19:14 2009 -0700
        added a submodule reference I will never make public. hahahahaha!
Às vezes, desenvolvedores querem obter uma combinação de subdiretórios de um grande projeto, dependendo de qual equipe eles estão. Isso é comum se você está vindo do CVS ou Subversion, onde você define um módulo ou uma coleção de subdiretórios, e você quer manter esse tipo de fluxo de trabalho.
Usar submódulos tem seus problemas. Primeiro, você tem que ser relativamente cuidadoso quando estiver trabalhando no diretório do submódulo. Quando você executa `git submodule update`, ele faz o checkout de uma versão específica do projeto, mas fora de um branch. Isso é chamado ter uma cabeça separada (detached HEAD) — isso significa que o HEAD aponta diretamente para um commit, não para uma referência simbólica. O problema é que geralmente você não quer trabalhar em um ambiente com o HEAD separado, porque é fácil perder alterações. Se você executar `submodule update`, fizer o commit no diretório do submódulo sem criar um branch para trabalhar, e em seguida executar `git submodule update` novamente no projeto-pai sem fazer commit nesse meio tempo, Git irá sobrescrever as alterações sem lhe informar. Tecnicamente você não irá perder o trabalho, mas você não terá um branch apontando para ele, por isso vai ser um pouco difícil de recuperá-lo.
    $ git checkout -b rack
    Switched to a new branch "rack"
    $ git submodule add git@github.com:schacon/rack.git rack
    Initialized empty Git repository in /opt/myproj/rack/.git/
    ...
    Receiving objects: 100% (3184/3184), 677.42 KiB | 34 KiB/s, done.
    Resolving deltas: 100% (1952/1952), done.
    $ git commit -am 'added rack submodule'
    [rack cc49a69] added rack submodule
     2 files changed, 4 insertions(+), 0 deletions(-)
     create mode 100644 .gitmodules
     create mode 160000 rack
    $ git checkout master
    Switched to branch "master"
    $ git status
    # On branch master
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #      rack/

Você tem que tirá-lo de lá ou removê-lo, em todo caso você tem que fazer o clone novamente quando você voltar — e você pode perder alterações ou branches locais que não foram enviados com um push.

O último problema que muitas pessoas encontram envolve mudar de subdiretórios para submódulos. Se você está rastreando arquivos no seu projeto e quer movê-los para um submódulo, você deve ser cuidadoso ou "o Git vai ficar com raiva de você". Digamos que você tem os arquivos do "rack" em um subdiretório do seu projeto, e você quer transformá-los em um submódulo. Se você apagar o subdiretório e em seguida executar `submodule add`, Git exibe isto:

    $ rm -Rf rack/
    $ git submodule add git@github.com:schacon/rack.git rack
    'rack' already exists in the index
    $ git rm -r rack
    $ git submodule add git@github.com:schacon/rack.git rack
    Initialized empty Git repository in /opt/testsub/rack/.git/
    remote: Counting objects: 3184, done.
    remote: Compressing objects: 100% (1465/1465), done.
    remote: Total 3184 (delta 1952), reused 2770 (delta 1675)
    Receiving objects: 100% (3184/3184), 677.42 KiB | 88 KiB/s, done.
    Resolving deltas: 100% (1952/1952), done.
    $ git checkout master
    error: Untracked working tree file 'rack/AUTHORS' would be overwritten by merge.
Você tem que mover o diretório do submódulo do `rack` de lá antes de mudar para um branch que não tem ele:
    $ mv rack /tmp/
    $ git checkout master
    Switched to branch "master"
    $ ls
    README    rack
## Merge de Sub-árvore (Subtree Merging) ##
Agora que você viu as dificuldades do sistema de submódulos, vamos ver uma maneira alternativa de resolver o mesmo problema. Quando o Git faz o merge, ele olha para as partes que vão sofrer o merge e escolhe a estratégia adequada de merge para usar. Se você está fazendo o merge de dois branches, Git usa uma estratégia _recursiva_ (_recursive_ strategy). Se você está fazendo o merge de mais de dois branches, Git usa a estratégia do _polvo_ (_octopus_ strategy). Essas estratégias são automaticamente escolhidas para você, porque a estratégia recursiva pode lidar com situações complexas de merge de três vias — por exemplo, mais de um ancestral comum — mas ele só pode lidar com o merge de dois branches. O merge octopus pode lidar com vários branches mas é cauteloso para evitar conflitos difíceis, por isso ele é escolhido como estratégia padrão se você está tentando fazer o merge de mais de dois branches.
Porém, existem também outras estratégias que você pode escolher. Uma delas é o merge de _sub-árvore_, e você pode usá-lo para lidar com o problema do subprojeto. Aqui você vai ver como resolver o problema do "rack" da seção anterior, mas usando merge de sub-árvore.
A ideia do merge de sub-árvore é que você tem dois projetos, e um deles está mapeado para um subdiretório do outro e vice-versa. Quando você escolhe um merge de sub-árvore, Git é inteligente o bastante para descobrir que um é uma sub-árvore do outro e faz o merge adequado — é incrível.
    $ git remote add rack_remote git@github.com:schacon/rack.git
    $ git fetch rack_remote
    warning: no common commits
    remote: Counting objects: 3184, done.
    remote: Compressing objects: 100% (1465/1465), done.
    remote: Total 3184 (delta 1952), reused 2770 (delta 1675)
    Receiving objects: 100% (3184/3184), 677.42 KiB | 4 KiB/s, done.
    Resolving deltas: 100% (1952/1952), done.
    From git@github.com:schacon/rack
     * [new branch]      build      -> rack_remote/build
     * [new branch]      master     -> rack_remote/master
     * [new branch]      rack-0.4   -> rack_remote/rack-0.4
     * [new branch]      rack-0.9   -> rack_remote/rack-0.9
    $ git checkout -b rack_branch rack_remote/master
    Branch rack_branch set up to track remote branch refs/remotes/rack_remote/master.
    Switched to a new branch "rack_branch"

Agora você tem a raiz do projeto Rack no seu branch `rack_branch` e o seu projeto no branch `master`. Se você fizer o checkout de um e depois do outro, você pode ver que eles têm raízes de projeto diferentes:

    $ ls
    AUTHORS           KNOWN-ISSUES   Rakefile      contrib           lib
    COPYING           README         bin           example           test
    $ git checkout master
    Switched to branch "master"
    $ ls
    README
    $ git read-tree --prefix=rack/ -u rack_branch
Quando você faz o commit, parece que você tem todos os arquivos do Rack nesse subdiretório — como se você tivesse copiado de um arquivo. O que é interessante é que você pode facilmente fazer merge de alterações de um branch para o outro. Assim, se o projeto Rack for atualizado, você pode fazer um pull das modificações mudando para o branch e fazendo o pull:
    $ git checkout rack_branch
    $ git pull
    $ git checkout master
    $ git merge --squash -s subtree --no-commit rack_branch
    Squash commit -- not updating HEAD
    Automatic merge went well; stopped before committing as requested
    $ git diff-tree -p rack_branch
    $ git diff-tree -p rack_remote/master