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

###Task 1 - Becoming a Certificate Authority (CA)

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

###Task 2 - Generating a Certificate Request for Your Web Server

Utilizamos o seguinte comando para gerar um certificado para o site www.bank32.com:

```bash
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" -passout pass:1234 -addext "subjectAltName=DNS:www.bank32.com,DNS:www.bank32A.com,DNS:www.bank32A.com"
```

Este comando resultou na criação de dois arquivos: a chave RSA do site (server.key) e o Pedido de Certificado do site (server.csr).

### Task 3 - 

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

### Task 4 - 

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


###Task 5 - Launching a Man-In-The-Middle Attack

A configuração do servidor foi alterada para apresentar o site www.example.com com as configurações anteriores. O arquivo "etc/apache2/sites-available/bank32_apache_ssl.conf" foi ajustado da seguinte maneira:

![Alt text](/images/Screenshot_from_2023-12-10_18-09-48.png)

Também alteramos o DNS da vítima, associando o hostname www.example.com ao IP do servidor web malicioso:

```bash
sudo nano /etc/hosts     # Adicionar a entrada '10.9.0.80 www.example.com'
```

Ao reconstruir o servidor e acessar o site www.example.com, observamos que o navegador emite um alerta indicando um potencial risco de segurança.

![Alt text](/images/Screenshot_from_2023-12-10_18-15-47.png)

###Task 6 - Launching a Man-In-The-Middle Attack with a Compromised CA
Assumindo que a nossa Autoridade de Certificação (CA) está comprometida, ela pode ser utilizada para gerar certificados para um site malicioso. Neste caso, desejamos criar um certificado para o site www.example.com, para isso repetimos os seguintes comandos da Tarefa 2:

```bash
$ openssl req -newkey rsa:2048 -sha256 -keyout example.key -out example.csr -subj "/CN=www.example.com/O=example Inc./C=US" -passout pass:1234
$ openssl ca -config myCA_openssl.cnf -policy policy_anything -md sha256 -days 3650 -in example.csr -out example.crt -batch -cert ca.crt -keyfile ca.key
```

Em seguida, ajustamos o arquivo de configuração do servidor em "etc/apache2/sites-available/bank32_apache_ssl.conf" para utilizar os dois arquivos gerados: example.crt e example.key.

![Alt text](/images/Screenshot_from_2023-12-10_18-30-00.png)

E, assim, a ligação já é segura.

![Alt text](/images/Captura_de_ecra_de_2023-12-10_23-31-37.png)
