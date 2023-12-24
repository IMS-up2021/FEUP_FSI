# Trabalho realizado na Semana #13
## Packet Sniffing and Spoofing Lab
### Task 1
Após iniciarmos os contentores respetivos, no contentor do atacante criamos um ficheiro chamado mycode.py e nele guardamos o código dado. Ao corrermos esse código podemos ver algumas informações do IP, fornecidas pelas funções do scapy, como maneira de introdução a este módulo.

![Alt text](/images/f2.png)
![Alt text](/images/f1.png)

#### Task 1.1A
Escrevemos o código dado num ficheiro com o nome de sniffer.py e substituimos os valores pelos valores da interface para os da nossa máquina.

![Alt text](/images/f26.png)

Ao corrermos este código com permição de root observamos que este fica à espera de detetar um ping, que neste caso enviamos da máquina host 10.9.0.6 para google.com, que resultou no seguinte:

![Alt text](/images/f4.png)
![Alt text](/images/f3.png)

Ao corrermos este mesmo código desta vez sem permição de root observamos que a operação que queremos realizar não nos é permitida. Isto acontece devido ao acesso às interfaces, que é necessário para realizar sniffing, requerir permições root.

![Alt text](/images/f5.png)

#### Task 1.1B
Nesta task é nos pedido para realizar sniffing usando alguns filtros. O primeiro pede para capturar apenas packets ICMP e este já foi realizado na task anterior, visto que era o filtro que ja vinha no código fornecido.

![Alt text](/images/f26.png)
![Alt text](/images/f4.png)
![Alt text](/images/f3.png)

O segundo pede para filtrarmos apenas packets TCP de um IP específico com destino à porta 23. Para isso mudamos o parametro filter para 'tcp && src host 10.9.0.6 && dst port 23' e observamos que os resultado são os esperados após inserirmos o comando telnet 10.9.0.5 no host 10.9.0.6 e fazermos o login.

![Alt text](/images/f7.png)
![Alt text](/images/f8.png)
![Alt text](/images/f6.png)

Na terceira pedianos para filtramos apenas packet que vem de uma específica subnet. Para isso mudamos o parametro filter para 'net 128.230.0.0/16' e demos ping a 128.230.0.1 desde o host 10.9.0.6 e os resultados foram os seguintes.

![Alt text](/images/f11.png)
![Alt text](/images/f10.png)
![Alt text](/images/f9.png)

#### Task 1.2
Nesta task começamos por criar um ficheiro task2.py no qual colocamos o código dado no lab. Este código da spoof a um packet com um IP arbitrário. Mudamos então o destino para ser o 5.5.5.5 e corremos o código com o wireshark ligado nesse mesmo host.

![Alt text](/images/f15.png)
![Alt text](/images/f13.png)
![Alt text](/images/f14.png)

O resultado foi o envio de um packet para o host desejado, este que foi capturado pelo wireshark.

![Alt text](/images/f12.png)

#### Task 1.3
Nesta task é nos pedido para observar o número de routers necessários para chegar ao host conhecido 8.8.8.8. Para isso, pegamos no código fornecido e fizemos-lhe algumas alterações, estas que guardamos no ficheiro task3.py.

![Alt text](/images/f17.png)

Ao correr este código com o wireshark ligado, observamos que apenas obtemos uma resposta ao packet que nós criamos no 22º packet, o que indica que passamos por 21 routers até chegarmos ao router desejado. Para além disso, é possível ver o IP de todos os routers pelos quais passamos no IP de origem da resposta de cada uma das mensagens que enviamos e morreram antes de chegar ao router desejado.

![Alt text](/images/f18.png)
![Alt text](/images/f16.png)

#### Task 1.4
Nesta task é nos pedido para juntarmos tudo o que aprendemos até agora e dar sniff a um packet e enviarmos uma resposta spoof de volta. Para isso criamos este código:

![Alt text](/images/f25.png)

Foi nos pedido para correr este código para 3 IP addresses diferentes. No primeiro, visto que o host não existe na Internet, a resposta obtida ao fazer ping é apenas a nossa resposta spoefed.

![Alt text](/images/f19.png)
![Alt text](/images/f20.png)

Já no segundo caso não é recebido nada e o host é dito unreachable, isto acontece porque visto que o host não está na LAN, o seu MAC address é desconhecido, e como este é necessário para criar os ARQ packets, nenhum packet é enviado.

![Alt text](/images/f21.png)
![Alt text](/images/f22.png)

Finalmente no terceiro caso ao corrermos o código e fazermos ping do host 8.8.8.8 recebemos um par de respostas, a resposta real e a resposta spoofed que criamos.

![Alt text](/images/f23.png)
![Alt text](/images/f24.png)

Em todos os casos o número de respostas que recebemos depende das respostas spoofed que criamos, sendo estas iguais ao número de vezes que o texto "Sent spoofed packet" é imprimido.

## Find-my-TLS CTF
Neste CTF recebemos um set de conexões TLS e é nos pedido para formar a flag com informações de uma dessas conexões, a que tem como random number da mensagem Client Hello igual a 52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27. Para isso selecionamos uma qualquer mensagem de Client Hello e usamos como filtro. Em seguida alteramos o filtro para ser igual ao valor que procuramos e assim encontramos que a mensagem Client Hello com random number 52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27 está na frame 814.

![Alt text](/images/ctf.png)

A partir desse ponto basta nos ir buscar a informação pedida. O frame inicial do handshake corresponde ao frame 814 enquanto o frame final corresponde ao framr 819. O nome do cipher suite utilizado pode ser encontrado no frame 816 (Serve Hello) e corresponde a TLS_RSA_WITH_AES_128_CBC_SHA256. O total app data basta somar a length da data nos frames 820 e 821 (application data) que resulta em 80 + 1184 = 1264. Finalmente o tamanho da mensagem encryptada basta ver o tamanho da mesma na frame que a contém (frame 819) e corresponde a 80. Combinando tudo obtemos a flag final: flag{814-819-TLS_RSA_WITH_AES_128_CBC_SHA256-1264-80}.
