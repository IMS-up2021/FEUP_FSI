# Trabalho realizado na Semana #10
## Secret Key Encryption Lab
### Task 1
Começamos por analisar a maneira como a monoalphabetic substitution cipher funciona e entender como analise de frequência funciona. Após esta análise, passamos então para o ficheiro que temos que decodificar. Corremos o código que nos é dado que nos deu então a frequência das 20 letras, combinações de 2 letras e combinações de 3 letras mais usadas.

![Alt text](/images/im3.png)
![Alt text](/images/im4.png)
![Alt text](/images/im5.png)

Começamos então a decodificar o texto aos poucos, iniciando pela combinação de 3 letras mais usada, a qual assumimos que era "the" devido a esta ser a combinação de 3 letras mais usada na língua inglesa. A partir daí fomos testando quer com outra combinações/letras mais usadas quer com outras letras que completavam palavras no texto criado pelo código anterior, até que eventualmente chegamos ao ficheiro completamente decodificado.

![Alt text](/images/im1.png)
![Alt text](/images/im2.png)

### Task 2
Começamos por fazer uma rápida análise dos parametros tanto da função openssl como da função enc, na qual também encontramos as cifras que podem ser usadas.

![Alt text](/images/im6.png)

Criamos então um novo ficheiro com o seguinte texto:

![Alt text](/images/im11.png)

Decidimos então usar as cifras bf com modo cbc (bf-cbc), cast com modo cdc (cast-cbc) e aes-128 com modo cbc (aes-128-cbc). Utilizamos então os comandos fornecidos e analisamos que os documentos criados estavam encriptados, todos eles com resultados diferentes.

![Alt text](/images/im7.png)
![Alt text](/images/im8.png)
![Alt text](/images/im9.png)
![Alt text](/images/im10.png)

Em seguida corremos os mesmos códigos de cima, mas desta vez mudamos o -e para -d, para decriptar os ficheiros criados e observamos que todos os ficheiros criados desta vez eram iguais, não só entre si como também ao ficheiro original, como seria de esperar.

![Alt text](/images/im12.png)
![Alt text](/images/im11.png)

### Task 3
Após uma pesquisa sobre cada um dos modos, descobrimos que o ecb nao tem como parametro o iv, ao contrário de cbc. Sabendo isto encriptamos então o ficheiro pic_original.bmp com os dois modos e corremos os códigos dados para ambos e obtivemos o seguinte (ordem é original, cbc, ecb para ambas as fotos, quer a dada quer a segunda pedida pelo enunciado):

![Alt text](/images/im13.png)
![Alt text](/images/im15.png)
![Alt text](/images/im14.png)

Observamos fácilmente que na encriptação em ecb mostra algumas partes da imagem original enquanto na encriptação em cbc isto não acontece. Testamos com outra imagem e obtivemos resultados semelhantes aos obtidos na imagem original.

![Alt text](/images/im17.png)
![Alt text](/images/im16.png)

Isto acontece visto que o ecb encripta cada bloco independentemente e em sequencia, enquanto o cbc encripta cada bloco combinado este com o bloco cifrado anterior usando um xor, o que significa que o resultado vai ser completamente diferente da imagem original.

## CTF 10 - Weak Encryption
### Análise Inicial

Ao analisar o código de encriptação, descobrimos que a alteração feita ocorre na geração da chave de encriptação, gerando um valor de 3 bytes que é anexado a 13 bytes com o valor 0. Sabendo que a chave só pode ter um valor compreendido entre 0 e 2²⁴, percebemos que um brute-force attack torna-se viável.

### Execução do ataque

Tendo acesso às funções de encriptação e desencriptação, primeiro obtemos o nosso texto encriptado e o nonce respetivo, chamando a função netcat na porta 6003 do endereço do ctf.
Obtendo os valores encriptados, iteramos chamadas da função de desencriptação com valores da chave de 0 a 2²⁴, e assim que o texto retornado começar com b'flag{', sabemos que utilizamos a chave correta e obtemos assim a flag.
