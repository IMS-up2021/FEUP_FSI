# Trabalho realizado na Semana #5

## Buffer-Overflow Attack Lab - Set-UID Version

### Environment Setup

Começamos por desativar algumas defesas do sistem operativo, nomeadamente a randomização do espaço de endereços e configurar o /bin/sh para contornar a diminuiçao de privilegios quando chamado através de um processo Set-UID, usando estes comandos:

### Task 1

Após uma análise do código apresesntado e perceber como funciona, compilámos o código guardado em call_shellcode.c usando o comando make, visto que o que é requerido para compilar o código já se encontrava na makefile. Após compilado, são criados 2 ficheiro, a32.out e a64.out,  que após serem corridos podemos observar que abriam shells no mesmo diretório de onde os ficheiros se encontram:

### Task 2

Estudando o código guardado em stack.c, observamos que o código faz uma chamada a uma função bof, na qual é guardada uma string str num buffer de 100 bytes, no entanto, a função utiliza strcpy para guardar a string, esta que não limita o tamanho da string e visto que na função onde é chamada a função bof a string str pode ter ate 517 bytes de tamanho, é possível existir overflow o que significa também que o programa está então vulnerável a um ataque por buffer overflow.

Desativarmos o StackGuard, compilámos o código, demos-lhe permisão de root e tornamo-lo no programa Set-UID usando o comando make, visto que todos estes passos já eram feitos na Makefile. Isto criou 8 ficheiros, stack-L(1 a 4) e stack-L(1 a 4)-dbg, dos quais vamos usar 4 (os com L1 e L2) nas tasks requeridas.

### Task 3

Nesta task começamos a tentar procurar uma maneira de utilizar a vulnerabilidade que descobrimos. Começamos por criar um ficheiro badfile, onde vai ser guardada a payload. Depois corremos o ficheiro stack-L1-dbg com o debugger e após a inserção dos comandos indicados no tutorial e os indicados no moodle (para garantir que os endereços da stack são iguais tanto no modo debugging como fora dele) e descobrimos o ebp e o endereço de início do buffer:

Com a informação obtida, preenchemos então os campos necessários no ficheiro exploit.py, sendo estes: 
a variável shellcode, em que inserimos o shellcode em 32-bit para executar uma shell;
a variável start, em que definimos o local de início do shellcode no ficheiro, ou seja os 517 bytes totais menos o tamanho da variável shellcode;
a variável ret, que indica o novo endereço de retorno, o qual definimos como o ebp + 400 (400 foi escolhido por ser um valor superior aos 100 bytes referidos anteriormente, para ter a certeza que este estará na zona onde ocoreu overflow);
a variával offser, esta que indica a localização do endereço de retorno relativamente ao início do array, sendo que esta corresponde a conta ebp - buffer + 4 (o 4 é devido ao valor que queremos mudar se encontrar diretamente um adress antes ao valor de ebp - buffer e como a arquitetura é de 32 bit, os adresses tem tamanho de 4 bytes.

Com esta informação preenchida, podemos então correr o código python, que preenche o ficheiro badfile, seguido do ficheiro stack-L1, e podemos então observar que obtemos uma shell com privilégios de root, como previsto:

### Task 4

A task 4 é bastante semelhante a task 3, só que desta vez não temos acesso à variável ebp.
Repetindo o processo de debugging feito na task 3, desta vez para stack-L2-dbg, mas desta vez sem o comando p $ebp, obtemos então a nova informação com que vamos preencher o ficheiro exploit.py, mas desta vez temos que improvizar um pouco, mudando a fórmula do ret para buffer + 400 (400 continua aqui pela mesma razão da task anterior) e como não temos acesso ao offset, vamos colocar return adresses em todos os sítios possíveis de serem o correto ao mesmo tempo, fazendo uso de um for loop e modificando ligeiramente o código dado pelos professores.

Após realizadas estas modificações no ficheiro exploit.py, corremos o ficheiro para preencher o ficheiro badfile com a payload e corremos então o ficheiro stack-L2 e vamos obter novamente uma shell com privilégios root como esperado:
