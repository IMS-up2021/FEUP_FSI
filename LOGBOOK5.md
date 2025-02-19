# Trabalho realizado na Semana #5

## Buffer-Overflow Attack Lab - Set-UID Version

### Environment Setup

Começamos por desativar algumas defesas do sistem operativo, nomeadamente a randomização do espaço de endereços e configurar o /bin/sh para contornar a diminuiçao de privilegios quando chamado através de um processo Set-UID, usando estes comandos:

![Alt text](/images/image-1.png)

### Task 1

Após uma análise do código apresesntado e perceber como funciona, compilámos o código guardado em call_shellcode.c usando o comando make, visto que o que é requerido para compilar o código já se encontrava na makefile. Após compilado, são criados 2 ficheiro, a32.out e a64.out,  que após serem corridos podemos observar que abriam shells no mesmo diretório de onde os ficheiros se encontram:

![Alt text](/images/image-2.png)

### Task 2

Estudando o código guardado em stack.c, observamos que o código faz uma chamada a uma função bof, na qual é guardada uma string str num buffer de 100 bytes, no entanto, a função utiliza strcpy para guardar a string, esta que não limita o tamanho da string e visto que na função onde é chamada a função bof a string str pode ter ate 517 bytes de tamanho, é possível existir overflow o que significa também que o programa está então vulnerável a um ataque por buffer overflow.

Desativarmos o StackGuard, compilámos o código, demos-lhe permisão de root e tornamo-lo no programa Set-UID usando o comando make, visto que todos estes passos já eram feitos na Makefile. Isto criou 8 ficheiros, stack-L(1 a 4) e stack-L(1 a 4)-dbg, dos quais vamos usar 4 (os com L1 e L2) nas tasks requeridas.

### Task 3

Nesta task começamos a tentar procurar uma maneira de utilizar a vulnerabilidade que descobrimos. Começamos por criar um ficheiro badfile, onde vai ser guardada a payload. Depois corremos o ficheiro stack-L1-dbg com o debugger e após a inserção dos comandos indicados no tutorial e os indicados no moodle (para garantir que os endereços da stack são iguais tanto no modo debugging como fora dele) e descobrimos o ebp e o endereço de início do buffer:

![Alt text](/images/image-3.png)
![Alt text](/images/image-4.png)
![Alt text](/images/image-5.png)
![Alt text](/images/image-6.png)
![Alt text](/images/image-7.png)

Com a informação obtida, preenchemos então os campos necessários no ficheiro exploit.py, sendo estes: 

- a variável shellcode, em que inserimos o shellcode em 32-bit para executar uma shell;
- a variável start, em que definimos o local de início do shellcode no ficheiro, ou seja os 517 bytes totais menos o tamanho da variável shellcode;
- a variável ret, que indica o novo endereço de retorno, o qual definimos como o ebp + 400 (400 foi escolhido por ser um valor superior aos 100 bytes referidos anteriormente, para ter a certeza que este estará na zona onde ocoreu overflow);
- a variával offset, esta que indica a localização do endereço de retorno relativamente ao início do array, sendo que esta corresponde à fórmula ebp - buffer + 4 (o 4 é devido ao valor que queremos mudar se encontrar diretamente um adress antes ao valor de ebp - buffer e como a arquitetura é de 32 bit, os adresses tem tamanho de 4 bytes).

![Alt text](/images/image-8.png)

Com esta informação preenchida, podemos então correr o código python, que preenche o ficheiro badfile, seguido do ficheiro stack-L1, e podemos então observar que obtemos uma shell com privilégios de root, como previsto:

![Alt text](/images/image-16.png)

### Task 4

A task 4 é bastante semelhante a task 3, só que desta vez não temos acesso à variável ebp.
Repetindo o processo de debugging feito na task 3, desta vez para stack-L2-dbg, mas desta vez sem o comando p $ebp, obtemos então a nova informação com que vamos preencher o ficheiro exploit.py, mas desta vez temos que improvizar um pouco, mudando a fórmula do ret para buffer + 400 (400 continua aqui pela mesma razão da task anterior) e como não temos acesso ao offset, vamos colocar return adresses em todos os sítios possíveis de serem o correto ao mesmo tempo, fazendo uso de um for loop e modificando ligeiramente o código dado pelos professores.

![Alt text](/images/image-9.png)
![Alt text](/images/image-10.png)
![Alt text](/images/image-11.png)
![Alt text](/images/image-12.png)
![Alt text](/images/image-15.png)


Após realizadas estas modificações no ficheiro exploit.py, corremos o ficheiro para preencher o ficheiro badfile com a payload e corremos então o ficheiro stack-L2 e vamos obter novamente uma shell com privilégios root como esperado:

![Alt text](/images/image-14.png)

## CTF 5 - Buffer Overflow

### Primeira Parte

#### Análise Inicial

Inicialmente, explorámos os arquivos fornecidos na plataforma CTF, executados no servidor na porta 4003.
Utilizando o comando `checksec`, verificamos que o programa (`main.c` compilado) tem arquitetura x86. O seu binário não é randomizado e não há proteções contra overflow no endereço de retorno ou execução na stack.

![Alt text](/images/_image1.png)

#### Funcionamento do Código

Analisámos o código no `main.c` e identificámos a alocação de 8 bytes de memória para a variável `meme_file` e 32 bytes para a resposta do usuário, `buffer`.
A função `scanf` permite a cópia de até 40 bytes do stdin para o `buffer`, o que pode resultar em buffer overflow quando o input ultrapassa 32 bytes.
A disposição contínua da memória na stack permite a sobrescrita da área alocada para `meme_file`. Dado que as instruções subsequentes exibem o conteúdo do ficheiro contido em `meme_file`, o nosso objetivo é reescrever o conteúdo da variável para direcionar as instruções ao ficheiro `flag.txt`.

![Alt text](/images/_image2.png)

#### Exploração

No terminal usando ```nc ctf-fsi.fe.up.pt 4003```, bastou escrever 32 caracteres (no nosso caso a) seguidos do nome do arquivo desejado.
Após o processamento, conseguimos acessar o conteúdo do arquivo `flag.txt` e obter a flag do desafio: `flag{be513fd339bd72a40365cea1da913558}`.

![Alt text](/images/_image3.png)

### Segunda Parte

#### Análise de Segurança

Assim como no desafio anterior, verificámos as proteções do programa em execução no servidor. A arquitetura é novamente x86, o binário também não é randomizado e não há proteções contra overflow no endereço de retorno ou execução na stack.

![Alt text](/images/_image4.png)

#### Funcionamento do Código

Em `main.c`, há alocação de 9 bytes para `meme_file`, 4 bytes para um valor `val`, e 32 bytes para a resposta do usuário, `buffer`.
A abordagem é semelhante à primeira parte; no entanto, o conteúdo do arquivo só é exibido quando o valor em `val` é igual a `0xfefc2324`.

![Alt text](/images/_image5.png)

#### Exploração

O output inicial dá a conhecer o valor inicial de `val`. Com essa informação, reconstruímos os bytes necessários para que `val` seja igual a `0xfefc2324`, ou seja `val` tem guardado o valor `\x24\x23\xfc\xfe`.
Após copiar o programa Python fornecido no desafio anterior e alterar o remote, na seção de injeção de conteúdo no servidor, bastou escrever 32 caracteres (mais uma vez a) seguidos do novo valor de `val` e do nome do arquivo desejado.
Ao executar, conseguimos acessar o conteúdo do arquivo `flag.txt` e adquirir a flag do desafio: `flag{6e694ce7b04d592f2882fccac8453b4a}`.

![Alt text](/images/_image6.png)
![Alt text](/images/_image7.png)
