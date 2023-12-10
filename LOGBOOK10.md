# Trabalho realizado na Semana #10
## Secret Key Encryption Lab
### Task 1
Começamos por analisar a maneira como a monoalphabetic substitution cipher funciona e entender como analise de frequência funciona. Após esta análise, passamos então para o ficheiro que temos que decodificar. Corremos o código que nos é dado que nos deu então a frequência das 20 letras, combinações de 2 letras e combinações de 3 letras mais usadas.

Começamos então a decodificar o texto aos poucos, iniciando pela combinação de 3 letras mais usada, a qual assumimos que era "the" devido a esta ser a combinação de 3 letras mais usada na língua inglesa. A partir daí fomos testando quer com outra combinações/letras mais usadas quer com outras letras que completavam palavras no texto criado pelo código anterior, até que eventualmente chegamos ao ficheiro completamente decodificado.

### Task 2
Começamos por fazer uma rápida análise dos parametros tanto da função openssl como da função enc, na qual também encontramos as cifras que podem ser usadas.

Criamos então um novo ficheiro com o seguinte texto:

Decidimos então usar as cifras bf com modo cbc (bf-cbc), cast com modo cdc (cast-cbc) e aes-128 com modo cbc (aes-128-cbc). Utilizamos então os comandos fornecidos e analisamos que os documentos criados estavam encriptados, todos eles com resultados diferentes.

Em seguida corremos os mesmos códigos de cima, mas desta vez mudamos o -e para -d, para decriptar os ficheiros criados e observamos que todos os ficheiros criados desta vez eram iguais, não só entre si como também ao ficheiro original, como seria de esperar.

### Task 3
Após uma pesquisa sobre cada um dos modos, descobrimos que o ecb nao tem como parametro o iv, ao contrário de cbc. Sabendo isto encriptamos então o ficheiro pic_original.bmp com os dois modos e corremos os códigos dados para ambos e obtivemos o seguinte (ordem é original, cbc, ecb para ambas as fotos, quer a dada quer a segunda pedida pelo enunciado):

Observamos fácilmente que na encriptação em ecb mostra algumas partes da imagem original enquanto na encriptação em cbc isto não acontece. Testamos com outra imagem e obtivemos resultados semelhantes aos obtidos na imagem original.

Isto acontece visto que o ecb encripta cada bloco independentemente e em sequencia, enquanto o cbc encripta cada bloco combinado este com o bloco cifrado anterior usando um xor, o que significa que o resultado vai ser completamente diferente da imagem original.
