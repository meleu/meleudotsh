## Tutorial de Bashly - parte 1

Neste artigo conheceremos o bashly. Pra que ele serve e as principais vantagens de utilizá-lo.

> **Observação**: para usar o Bashly é necessário saber lidar com arquivos YAML (o que é uma coisa bem simples).

## Por que usar Bashly?

Pra começar a falar dos motivos de usar o bashly, vamos imaginar o seguinte cenário...

Temos um código bash bem simples que serve para gerar números aleatórios. Chamemos esse programa de `rndm`, nosso gerador de números randômicos. Nosso programa é tão simples que ele contém apenas uma linha: `echo $RANDOM`.

Talvez a gente nem precisasse escrever um script só pra isso, mas é começamos a pensar em novas funcionalidades para nosso programa.

Por exemplo, queremos especificar o número máximo a ser gerado, pra poder chamar nosso programa assim:

```bash
# gerar numero entre 1 e 100
rndm --max 100
```

Também queremos especificar o número mínimo, pra poder fazer algo assim:

```bash
# gerar número entre 75 e 100
rndm --min 75 --max 100
```

Talvez você até saiba como gerar números aleatórios dentro de uma faixa específica. A lógica pra fazer isso nem é tão complicada assim. Mas se algum dia já escreveu um programa bash que faça _parsing_ de `--opções` da linha de comando você sabe bem o que vai acontecer: o nosso programinha, que originalmente era um simples `echo $RANDOM`, vai explodir em complexidade só por conta do código necessário para lidar com essas opções.

Ah! E já que você adicionou opções ao seu programa, você também tem que providenciar um `--help` para que o usuário saiba quais são opções disponíveis e como usá-las corretamente.

No final das contas você gastar mas energia mental lidando com todas essas minúcias de parsing de opções e help do que com o problema que você realmente quer resolver (no nosso exemplo: gerar números aleatórios).

É pra resolver esse tipo de problema que o Bashly foi criado! O Bashly vai te ajudar a:

- fazer parsing de `--opções`
- criar mensagens de help facilmente
- validar input
- verificar dependências
- criar subcomandos (no estilo `git add`, `git commit`, `git push`)
- e mais muitas outras coisas...

Ao usar Bashly essas coisas, que são tediosas de lidar, porém importantes de se ter em um típico programa CLI, serão resolvidas. Assim você pode se concentrar no problema que você realmente precisa resolver.

E para ilustrar como criar um CLI usando o Bashly, criaremos um programa gerador de número randômicos. Ele começará bem simples, mas irá receber muitas funcionalidades interessantes ao longo do tutorial.

## Instalando o Bashly

O Bashly é uma gem do Ruby. E ele depende que você tenha o Ruby instalado numa versão 3.2 ou maior.

Eu gosto muito de usar "runtime version managers" como o [mise](https://mise.jdx.dev/) (uso e recomendo) ou [asdf](https://asdf-vm.com) para instalar interpretadores e compiladores em diversas versões. Recomendo que você faça o mesmo para instalar o Ruby numa versão 3.2 ou maior.

No caso do mise, eu simplesmente faço algo assim:

```bash
# instalando ruby 3.4 e atribuindo como default
mise use --global ruby@3.4
```

Uma vez que o Ruby tá instalado, vamos instalar o Bashly:

```bash
gem install bashly
```

Pra conferir que está tudo certinho:

```
$ bashly --version
1.2.13
```

## Iniciando o projeto

Vamos começar criando um diretório para o nosso projeto

```bash
mkdir rndm
cd rndm
```

Uma maneira de iniciar um projeto com o bashly, é usar `bashly init`, isso irá criar um arquivo chamado `src/bashly.yml`. Se você fizer isso observará que o arquivo já vem com muita informação e isso pode ser um pouco confuso para um primeiro contato.

Aqui nós vamos escrever o `bashly.yml` totalmente "na mão", e vamos aprender cada configuração com calma. Portanto eu sugiro que você abra o `src/bashly.yml` e remova toda o conteúdo que encontrar lá. Em seguida adicione apenas isso:

```yaml
name: rndm
help: Prints a random number
version: 0.0.1
```

Agora basta executarmos `bashly generate`, e teremos um output assim:

```
$ bashly generate
creating user files in src
created src/root_command.sh
created ./rndm
run ./rndm --help to test your bash script
```

Pois vamos fazer exatamente o que ele está sugerindo ali no final da mensagem:

```
$ ./rndm --help
rndm - Prints a random number

Usage:
  rndm
  rndm --help | -h
  rndm --version | -v

Options:
  --help, -h
    Show this help

  --version, -v
    Show version number
```

🤩 - Olha isso!!

Não escrevemos uma única linha de bash e veja que help lindão que já temos!

Vamos entender rapidamente o que acabou de acontecer. De maneira simplificada, o comando `bashly generate` fez o seguinte:

1. leu o conteúdo do `src/bashly.yml`
2. entendeu que queremos criar um script chamado `rndm`
3. leu a descrição do script no `help:`
4. leu a versão do script no `version:`
5. gerou o script final, já implementando as funcionalidades de `--help` e  `--version`

Se olharmos direitinho o output do `bashly generate`, veremos que ele também gerou um arquivo chamado `src/root_command.sh`. Vamos dar uma olhadinha no conteúdo dele:

```bash
echo "# This file is located at 'src/root_command.sh'."
echo "# It contains the implementation for the 'rndm' command."
echo "# The code you write here will be wrapped by a function named 'rndm_command()'."
echo "# Feel free to edit this file; your changes will persist when regenerating."
inspect_args
```

Traduzindo o que está naqueles comentários, temos:

```
Esse arquivo está localizado em 'src/root_command.sh'.
Ele contem a implementação do comando 'rndm'.
O código que você escrever aqui ficará em uma função chamada 'rndm_command()'
The code you write here will be wrapped by a function named 'rndm_command()'."
Feel free to edit this file; your changes will persist when regenerating."
```