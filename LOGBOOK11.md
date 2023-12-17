# Trabalho realizado na Semana #11
## Public-Key Infrastructure (PKI) Lab
### Setup

Nesta fase inicial, começamos por adicionar novas entradas aos arquivos de host reconhecidos pela VM e em seguida executamos os containers fornecidos no lab:

Adicionar Entradas ao Arquivo de Hosts -
Abrir o arquivo de hosts, usando o editor de texto nano:

```bash
sudo nano /etc/hosts
```

Construir os Containers usando Docker Compose:

```bash
dcbuild
```

Iniciar os Containers:

```bash
dcup
```

### Task 1 - Becoming a Certificate Authority (CA)

Começamos por copiar o arquivo de configuração de certificado padrão, localizado em /usr/lib/ssl/openssl.cnf, para o diretório de trabalho local. Em seguida, comentamos a linha "unique_subject" e executamos os seguintes comandos para criar o ambiente da nossa própria CA:

mkdir myCA && cd ./myCA
mkdir certs crl newcerts
touch index.txt
echo "1000" >> serial

Em seguida, configuramos a CA utilizando o seguinte comando:

openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt

E durante o processo, fornecemos os seguintes dados:

    Senha (passphrase): 1234
    País: PT
    Região: Porto
    Cidade: Paranhos
    Organização: UP
    Secção: FEUP
    Nome: l10g02
    Email: aluno@fe.up.pt

Para visualizar o conteúdo dos arquivos gerados, decodificamos o certificado X.509 e a chave RSA usando os seguintes comandos:

```bash
openssl x509 -in ca.crt -text -noout
openssl rsa -in ca.key -text -noout
```

Podemos identificar que se trata de um certificado de Autoridade de Certificação (CA) observando a presença do atributo "certificate authority" na seção "basic constraints", que possui o valor verdadeiro.

![Alt text](/images/Screenshot_from_2023-12-10_13-02-14.png)

Além disso, este certificado é autoassinado, indicado pelo fato de que os campos "issuer" e "subject" são idênticos.

![Alt text](/images/Screenshot_from_2023-12-10_13-03-03.png)

Este ficheiro contém:
- Dois números primos (presentes nos campos prime1 e prime2).
- O módulo (campo modulus).
- Os expoentes público e privado (nos campos publicExponent e privateExponent, respectivamente).
- O coeficiente (campo coeficient).

### Task 2 - Generating a Certificate Request for Your Web Server

Utilizamos o seguinte comando para gerar um certificado para o site www.bank32.com:

```bash
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" -passout pass:1234 -addext "subjectAltName=DNS:www.bank32.com,DNS:www.bank32A.com,DNS:www.bank32A.com"
```

Este comando resultou na criação de dois arquivos: a chave RSA do site (server.key) e o Pedido de Certificado do site (server.csr).

### Task 3 - Generating a Certificate for your server

Para emitir um certificado para o nosso servidor www.bank32.com, executamos o seguinte comando:

```bash
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

Como resultado, obtivemos o arquivo server.crt. O conteúdo desse arquivo confirma que se trata de um certificado destinado ao servidor mencionado.

![Alt text](/images/Screenshot_from_2023-12-10_16-30-26.png)
![Alt text](/images/Screenshot_from_2023-12-10_16-30-35.png)

### Task 4 - Deploying Certificate in an Apache-Based HTTPS Website

Copiamos os arquivos "server.crt" e "server.key" para o diretório compartilhado /volumes e renomeamos para "bank32.crt" e "bank32.key", respectivamente. Em seguida, ajustamos o arquivo "etc/apache2/sites-available/bank32_apache_ssl.conf" dentro do container para utilizar o certificado e a chave da pasta compartilhada da seguinte forma:

```apache
<VirtualHost *:443> 
    DocumentRoot /var/www/bank32
    ServerName www.bank32.com
    ServerAlias www.bank32A.com
    ServerAlias www.bank32B.com
    ServerAlias www.bank32W.com
    DirectoryIndex index.html
    SSLEngine On 
    SSLCertificateFile /volumes/bank32.crt
    SSLCertificateKeyFile /volumes/bank32.key
</VirtualHost>
```

Para iniciar o servidor Apache, primeiro abrimos um terminal no contêiner e inserimos os seguintes comandos:

```bash
$ service apache2 start
```
![Alt text](/images/Screenshot_from_2023-12-10_18-02-14.png)
![Alt text](/images/Screenshot_from_2023-12-10_18-05-47.png)


### Task 5 - Launching a Man-In-The-Middle Attack

A configuração do servidor foi alterada para apresentar o site www.example.com com as configurações anteriores. O arquivo "etc/apache2/sites-available/bank32_apache_ssl.conf" foi ajustado da seguinte maneira:

![Alt text](/images/Screenshot_from_2023-12-10_18-09-48.png)

Também alteramos o DNS da vítima, associando o hostname www.example.com ao IP do servidor web malicioso:

```bash
sudo nano /etc/hosts     # Adicionar a entrada '10.9.0.80 www.example.com'
```

Ao reconstruir o servidor e acessar o site www.example.com, observamos que o navegador emite um alerta indicando um potencial risco de segurança.

![Alt text](/images/Screenshot_from_2023-12-10_18-15-47.png)

### Task 6 - Launching a Man-In-The-Middle Attack with a Compromised CA
Assumindo que a nossa Autoridade de Certificação (CA) está comprometida, ela pode ser utilizada para gerar certificados para um site malicioso. Neste caso, desejamos criar um certificado para o site www.example.com, para isso repetimos os seguintes comandos da Tarefa 2:

```bash
$ openssl req -newkey rsa:2048 -sha256 -keyout example.key -out example.csr -subj "/CN=www.example.com/O=example Inc./C=US" -passout pass:1234
$ openssl ca -config myCA_openssl.cnf -policy policy_anything -md sha256 -days 3650 -in example.csr -out example.crt -batch -cert ca.crt -keyfile ca.key
```

Em seguida, ajustamos o arquivo de configuração do servidor em "etc/apache2/sites-available/bank32_apache_ssl.conf" para utilizar os dois arquivos gerados: example.crt e example.key.

![Alt text](/images/Screenshot_from_2023-12-10_18-30-00.png)

E, assim, a ligação já é segura.

![Alt text](/images/Captura_de_ecra_de_2023-12-10_23-31-37.png)

## RSA CTF
Começamos por seguir as tarefas recomendadas na página do moodle. Encontramos o algoritmo de Miler-Rabin na página https://www.geeksforgeeks.org/primality-test-set-3-miller-rabin/ e usamos o mesmo no nosso código. Aproveitando o código que nos é dado, construímos um código para encriptar e decriptar uma string.

```
def miillerTest(d, n):
     
    # Pick a random number in [2..n-2]
    # Corner cases make sure that n > 4
    a = 2 + random.randint(1, n - 4);
 
    # Compute a^d % n
    x = pow(a, d, n);
 
    if (x == 1 or x == n - 1):
        return True;
 
    # Keep squaring x while one 
    # of the following doesn't 
    # happen
    # (i) d does not reach n-1
    # (ii) (x^2) % n is not 1
    # (iii) (x^2) % n is not n-1
    while (d != n - 1):
        x = (x * x) % n;
        d *= 2;
 
        if (x == 1):
            return False;
        if (x == n - 1):
            return True;
 
    # Return composite
    return False;
 
# It returns false if n is 
# composite and returns true if n
# is probably prime. k is an 
# input parameter that determines
# accuracy level. Higher value of 
# k indicates more accuracy.
def isPrime(n, k):
     
    # Corner cases
    if (n <= 1 or n == 4):
        return False;
    if (n <= 3):
        return True;
 
    # Find r such that n = 
    # 2^d * r + 1 for some r >= 1
    d = n - 1;
    while (d % 2 == 0):
        d //= 2;
 
    # Iterate given number of 'k' times
    for i in range(k):
        if (miillerTest(d, n) == False):
            return False;
 
    return True;
```

Podemos então responder as perguntas pedidas:
- Como consigo usar a informação que tenho para inferir os valores usados no RSA que cifrou a flag?
    - Sabendo p e q, conseguimos facilmente calcular todos os outros valores usando formulas conhecidas (n = p*q e ed % ((p-1)*(q-1)) = 1).
- Como consigo descobrir se a minha inferência está correta?
    - Se encriptarmos uma string e a decriptarmos em seguida e o resultado for igual à string inicial sabemos que a inferência está correta.
- Finalmente, como posso extrair a minha chave do criptograma que recebi?
    - Ao sabermos os primos p e q usado conseguimos calcular as chaves tanto publicas como privadas, como referido na primeira pergunta, que depois podemos utilizar para decifrar a cifra recebida.

Após esta análise, usamos o código que escrevemos em cima para criar uma nova função que calcula o primo seguinte a um valor dado. Passando 2^512 e 2^513 a esta função conseguimos determinar p e q respetivamente e depois basta calcular d. Com estes novos valores, passamos os mesmos a função dec juntamente com unhexlify(cifra) e obtemos um valor que após usarmos o .decode() nos revela a flag em string: flag{7860aa809d11eaebd4d9ebb67ec56c26}.
```
def nextPrime(number):

	while True:
		if isPrime(number, 4) and n % number == 0:
			return number
		else:
			number += 1

ciphertext = "6366343333613637386263303333656230393261356662633636383834373165643964316235363436353961303564306237376233373465653639393235353939613439613033636337306630376132306564626338393461396439343132623262343232303835373366656438626135663634653761663731633233663061333632633361306530633366666563383731323665613338323331333535656663363838666665623934633033623430303538396335623532353662623762333736363963653661313230383063323666326662646438623566373136306263653964373034653331383235313834396564656135653066366464323365366230313030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030"
n = 359538626972463181545861038157804946723595395788461314546860162315465351611001926265416954644815072042240227759742786715317579537628833244985694861278997871052684504401596494019122799039009989371808178352060966438579515613360227960308969957859170183912581829276320781354104234997246446865042463594219967161283
p = nextPrime(2**512)
q = nextPrime(2**513)
e = 0x10001 # 0x10001 é o equivalente do valor 65537 em decimal transformado em hexadecimal
d = pow(e, -1, ((p-1)*(q-1)))

flag = dec(unhexlify(ciphertext), d, n)
print(flag.decode())
```
