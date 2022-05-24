---
title: Comece por aqui!
description: >
  Uma relação de material em português que eu recomendo para que você comece DO ZERO e chegue a um nível razoável de proficiência em programação shell.
tags:
  - fundamentos
date: 2022-05-22T10:29:28-03:00
cover:
  image: "img/por-que-shell.gif"
  alt: prompt
---

Neste artigo eu ofereço uma relação de conteúdo **em português** você comece DO ZERO e chegue a um nível razoável de proficiência em programação shell.

Eu reconheço que alguns artigos aqui do meleu.sh não são lá tão focados no leitor que está iniciando suas aventuras no shell.

Conforme eu digo na [página "sobre"](/sobre/#por-que-criei-esse-site), na seção "Por que criei esse site?":

> Meu objetivo é conseguir difundir no mercado brasileiro a adoção de boas práticas referentes a shell-script (principalmente bash). Deixar o código mais legível e de mais fácil manutenção.

Como podemos inferir, se o meu objetivo é divulgar boas práticas de shell-script, está implícito que o leitor conheça o mínimo do assunto para então aplicar estas práticas.

Eu pretendo manter o meleu.sh nessa mesma pegada, mas não quero deixar de fora o pessoal que é completamente iniciante. E para evitar "reinventar a roda" e escrever eu mesmo material voltado para iniciantes, vou listar aqui material que eu considero de ótima qualidade publicado em português.

(Curiosidade: em 2002 eu escrevi um material bastante extenso sobre bash em português. Este sim, era voltado para iniciantes. Esse documento [ainda existe](https://meleu.gitbooks.io/bashscripting), mas nesses 20 anos muita coisa boa já foi escrita em português.)

## Sou completamente iniciante

Imaginando o cenário em que você não sabe nem o que é variável, `if`, `for`, `while`, `case`, etc.

Se você quer aprender essas coisas usando o shell. Eu recomendo fortemente que você confira a série de vídeos do Blau Araújo intitulada **[Curso Básico de Bash](https://www.youtube.com/playlist?list=PLXoSGejyuQGpf4X-NdGjvSlEFZhn2f2H7)**.

O Blau tem uma didática incrível, explica tudo com bastante paciência e vai progredindo devagarinho, adicionando novos conceitos só depois de ter explicado o anterior com detalhes.

Se você acabou de chegar no universo Linux e quer começar a se aventurar na telinha preta, essa série do Blau vai te ajudar muito!


## Já tenho uma noção de programação/algoritmos

Se você já conhece alguma coisa de algoritmos e programação, o que eu recomendo é o **[Curso intensivo de programação em Bash](https://www.youtube.com/playlist?list=PLXoSGejyuQGr53w4IzUzbPCqR4HPOHjAI)**, também do Blau Araújo. Trata-se de uma série de vídeos onde ele já parte do princípio que você já tem alguma noção básica de programação e vai evoluindo de uma maneira um pouco mais rápida do que o Curso Básico.

Existe ainda uma outra série de vídeos do Blau que eu achei bem legal, que é a **[Criação de scripts em Bash](https://www.youtube.com/playlist?list=PLXoSGejyuQGrjEIS_tIJ7XYJTcc1ggQy-)**. Onde ele vai escrevendo um script para o "jogo da adivinhação" e no caminho vai explicando vários conceitos de programação shell. Também recomendo.


## Livros

Para aqueles que, como eu, gostam de ter um material na forma escrita, o Blau (eita p\*rr@! esse cara tá em todas!) também escreveu o **[Pequeno Manual do Programador Bash](https://blauaraujo.com/livro/)**. Conteúdo bastante valioso que pode ser baixado gratuitamente.

Outro livro bastante valioso para se ter a mão é o **[Programação Shell Linux: Referência Definitiva da Linguagem Shell](https://novatec.com.br/livros/programacao-shell-linux-12ed/)**, do grande mestre Júlio Cezar Neves. Como o próprio título diz, este é um livro de referência. Excelente para ter ao alcance para tirar uma dúvida e/ou aprender algo que ainda não esteja dominando. (Observação: esse livro foi originalmente publicado pela editora Brasport, mas em sua 12a. edição foi para a editora Novatec e está com uma diagramação mais caprichada).

Após já ter começado a não só brincar com shell scripts, mas também a escrever scripts mais sérios, o próximo material que eu recomendo é o **[Shell Script Profissional](https://www.shellscript.com.br/)**, do [Aurelio Marinho Jargas]. Aqui vamos sair do básico e começar a levar o shell script mais a sério.

Cedo ou tarde você vai precisar se sentir confortável com Expressões Regulares, e na nossa língua o melhor livro que existe nesse tema é o **[Expressões Regulares - Uma abordagem divertida](https://www.piazinho.com.br/)**, também do Aurelio Marinho Jargas. Esse livro eu recomendo para qualquer profissional de TI, até mesmo se não vai trabalhar com shell. Conteúdo extremamente valioso.


## Artigos mais importantes do meleu.sh

Eu uso a tag [fundamentos](/tags/fundamentos) para categorizar artigos que considero importantes para iniciantes. Também uso a tag [boas-praticas](/tags/boas-praticas) para artigos que mostram algumas técnicas que podem até não ser totalmente compreendidas por iniciantes, mas se um iniciante seguir, certamente evitará muitas dores de cabeça.

Vou listar aqui, em ordem de importância (o que vem primeiro é o mais importante):

### O que o #! realmente faz?

[link](/shebang)

O "*shebang*" é a coisa mais fundamental de um script, pois é onde você define quem vai interpretar os comandos que estão no arquivo. Portanto você **precisa** entender o que ele faz.

### Por que você deve usar aspas SEMPRE

[link](/aspas-sempre)

Resumindo a importância desse artigo com uma citação: "uma variável em bash é como uma granada, tire suas aspas e uma hora ela explodirá".


### Deixe o bash mais rigoroso com seu script e evite dores de cabeça

[link](/bash-rigoroso)

Com essa técnica o seu script irá falhar o mais rápido possível. Desta forma você terá poderá repará-lo enquanto ainda está trabalhando nele, com o contexto do problema ainda fresquinho na cabeça.

**Use essa técnica mesmo que você seja um iniciante!**

### Como detectar precisamente onde seu script está quebrando

[link](/trap-err)

Com essa técnica você será capaz de evitar muita perda de tempo tentando encontrar onde seu script está bugando.

Nesse artigo falamos de `trap`, o que não é lá um tema muito voltado para iniciantes. Mas recomendo que você faça um esforcinho para usar essa técnica, pois também é uma coisa que vai te poupar muitas dores de cabeça e perda de tempo.

### Use shellcheck e livre-se dos bugs no seu código antes mesmo de executá-lo

[link](/shellcheck)

Isso também não é algo super-simples para uma pessoa que ainda está se iniciando na linha de comando. Pode requerer um pouquinho de esforço pra colocar o shellcheck no seu fluxo de trabalho. Mas acredite em mim: a quantidade de tempo que você vai economizar se você usar shellcheck, vai recompensar o seu esforço!

O shellcheck é capaz de te avisar que seu código está perigoso e/ou bugado, antes mesmo de você executá-lo!

O shellcheck vai te explicar o motivo que aquele código é considerado perigoso e ainda vai te ensinar alternativas mais seguras de alcançar o mesmo resultado. Ou seja, o shellcheck é também uma excelente ferramenta de aprendizado! (observação: as mensagens do shellcheck são em inglês).


## Conclusão

Neste artigo listei os conteúdos **em português** que considero mais relevantes para você começar sua jornada de programador shell e atingir um nível de proficiência bem rápido.

Não se esqueça de praticar! Sem prática, todo esse conteúdo e todas essas ferramentas são praticamente inúteis!




