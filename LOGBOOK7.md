# Trabalho realizado na Semana #7
## Format-String Vulnerability Lab
### Environment Setup
Começamos por desativar a randomição de endereços, sendo que vai ser útil saber a localização de alguns endereços relevantes às tasks.

![Alt text](/images/i1.png)

Após isto, começamos a analisar o programa com a vulnerabilidade e procedemos então a compilá-lo e ligamos então os contentores com os servidores para este lab.
### Task 1: Crashing the Program
Iniciamos esta parte ligando-nos ao servidor que corre com 10.9.0.5 e abrindo um segundo terminal com o serverside. Começamos por correr o programa normalmente usando o primeiro terminal (terminal de cliente) para podermos ver o que acontece no serverside e obtivemos o seguinte:

![Alt text](/images/i2.png)
![Alt text](/images/i3.png)

Podemos então obter algumas informações, nomeadamente alguns adresses que nos vão ser úteis para algumas tasks futuras e é nos apresentado também um print com a informação que a format string vai buscar.
Focando nos agora no objetivo da task, que é crashar o programa, podemos simplesmente enviar o input %s, visto que a format string vai buscar o endereço posicionado acima da posição da mesma na stack e imprime a string q está nesse endereço. Como o programa não estava a espera de ter que conter uma string nesse local, esta zona muito provavelmente vai estar fora da memória virtual do processo.

![Alt text](/images/i4.png)
![Alt text](/images/i5.png)

Como demonstrado pela print, no serverside não foi apresentada a mensagem "(ˆ_ˆ)(ˆ_ˆ) Returned properly (ˆ_ˆ)(ˆ_ˆ)", o que significa que o servidor crashou.
###  Task 2: Printing Out the Server Program’s Memory
**Task 2.A: Stack Data**

Para podermos saber se estamos a dar print à data ou não, definimos o inicio da data como sendo as letras "FNAF" (46414e46 em hexadecimal (small-endian)) e o resto do input como sendo %x até encontrarmos o código hexadecimal destas letras. Para o tal vamos usar o código python fornecido para criar e guardar a string num ficheiro chamado badfile durante o resto das tasks. O código python usado foi o seguinte:

![Alt text](/images/i7.png)

Usando então o ficheiro badfile, enviamos o seu conteúdo para o servidor e obtemos o seguinte:

![Alt text](/images/i6.png)
![Alt text](/images/i8.png)

Observando o output, podemos ver que os últimos números imprimidos são os números que procuravamos (46414e46), logo precisamos de 63 %x a imprimir endereços intermédios até chegarmos ao adress que contém os primeiros 4 bytes de dados, que corresponde ao 64 %x.

**Task 2.B: Heap Data**

Agora que sabemos onde os dados estão guardados, podemos fácilmente descobrir qual a mensagem secreta guardada no adress 0x080b4008. Para isso, basta substituir os primeiros 4 bytes da mensagem por "\x08\x40\x0b\x08" (visto que a máquina é small-endian) e substituir o último %x por um %s para imprimir a string.

![Alt text](/images/i9.png)

Enviando então a nova string criada para o servidor, obtemos o seguinte:

![Alt text](/images/i10.png)

Se olharmos para o fim do output podemos observar a mensagem secreta, que neste caso foi "A secret message".
### Task 3: Modifying the Server Program’s Memory

**Task 3.A: Change the value to a different value**

Esta task é semelhante à 2.B, logo basta mudar os primeiros 4 bytes da mensagem para "\x68\x50\x0e\08" e substituir o %s por um %n, este que escreve na memória o número de carateres escritos até à sua posição.

![Alt text](/images/i11.png)

Enviando então a nova string criada para o servidor, obtemos o seguinte:

![Alt text](/images/i12.png)

Como podemos ver, o valor da target variable mudou para 0x000000ec.

**Task 3.B: Change the value to 0x5000**

Mais uma vez, esta task é semelhante à anterior mas, neste caso, apenas temos que mudar o penúltimo valor de %x para %nx, em que n corresponte ao número de carateres que falta para chegarmos a 0x00005000 (20480 em decimal). Começamos por enviar para o servidor uma mensagem apenas com os 4 bytes iniciais e 63 %x e contamos que eram imprimidos 233 carateres.

![Alt text](/images/i13.png)
![Alt text](/images/i14.png)

Logo, o nosso valor de n será 20480-233, o que resulta em 20247. Inserindo esse valor como o n em %nx e substituindo este no 63 %x, ao enviar esta mensagem para o servidor obtemos então o seguinte resultado:

![Alt text](/images/i15.png)
![Alt text](/images/i16.png)
![Alt text](/images/i17.png)

Podemos observar que o valor da target variable é 0x00005000, como pretendido.

## Format Strings CTF
### Desafio 1
Após o download do ficheiros que nos eram disponibilizados, começamos por executar o comando checksec no ficheiro program que estava dentro destes contidos, descobrindo que este não tem os endereços randomizados e tem canários no endereço de retorno, isto tudo numa arquitetura little endian de 32 bits.

![Alt text](/images/ima1.png)

Passando então a analisar o código contido em main.c descobrimos que este aceita um input de 32 bytes que é impresso por um printf, o que significa que é vulnerável a um ataque de format strings, esta que nos permite tanto ler como escrever o que quisermos em memória, neste caso na heap.
Sabendo isto, abrimos o programa no debugger gdb e procuramos pela localizaçao da flag, guardada na variável global flag pela função load_flag.

![Alt text](/images/ima2.png)

![Alt text](/images/ima3.png)

Tendo acesso ao endereço da flag, foi então apenas uma questão de o transformar no formato string e inseri-lo no código python fornecido seguido de um "%s" para imprimir a string contida em flag e correr o mesmo para obtermos a flag.

![Alt text](/images/ima4.png)

![Alt text](/images/ima5.png)

### Desafio 2
Após o download dos novos ficheiros que nos eram disponibilizados, começamos por executar novamente o comando checksec no ficheiro program que estava dentro destes contidos, descobrindo que este tinha caracteristicas semelhantes ao anterior.

![Alt text](/images/ima6.png)

Passando então a analisar o código contido no novo main.c, descobrimos que este código sobre da mesma vulnerabilidade que o anterior mas desta vez não nos dará acesso direto à flag, mas sim a uma shell com permição de root.  Para isso, uma variável key teria que ter o valor 0xBEEF, que inicialmente está definido como 0. Sabendo isto, usamos novamente o gdb para descobrir o endereço da variável global key.

![Alt text](/images/ima7.png)

Transformando então esse valor da mesma maneira que o desafio anterior e colocando-o numa copia ligeiramente modificada do código python fornecido no desafio 1 que, juntamente com "%48875x" (devido a 0xBEEF = 48879 em decimal e subtrai-se 4 devido ao espaço que o endereço que o antecede ocupa) e um "%1$n", nos dá acesso à backdoor e à shell respetiva.

![Alt text](/images/ima8.png)

![Alt text](/images/ima9.png)

Depois basta usar um simples cat flag.txt para obter a flag desejada.

![Alt text](/images/ima10.png)
