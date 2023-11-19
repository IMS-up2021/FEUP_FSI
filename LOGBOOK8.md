# Trabalho realizado na Semana #8
## SQL Injection Lab
### Lab Environment
Começamos por adicionar nome do host juntamente com o IP do container ao ficheiro /etc/hosts.

![Alt text](/images/img1.png)

Depois iniciamos os containers fornecidos e abrimos uma shell de mysql para ter-mos acesso direto à base de dados usada nas tasks.

![Alt text](/images/img2.png)
![Alt text](/images/img3.png)

### Task 1: Get Familiar with SQL Statements
Após inserirmos os comandos indicados pelo guião e descobrirmos o nome da tabela, escrevemos um comando que vai buscar todos os dados de um utilizador com nome 'Alice'.

![Alt text](/images/img4.png)
![Alt text](/images/img5.png)
![Alt text](/images/img6.png)

### Task 2: SQL Injection Attack on SELECT Statement

**2.1: SQL Injection Attack from webpage**

Após aceder ao site, procedemos a analizar o ficheiro unsafe_home.php. Neste descobrimos que a maneira como tratam dos dados recebidos não é segura, visto que o comando é criado dinamicamente com strings não verificadas após o input do utilizador, o que significa que para podermos entrar na conta do admin podemos simplesmente escrever no username "admin';#" (a secçao de input da password não foi escolhida devido a esta ser codificada após o input). O que isto vai fazer é fazer apenas a query SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password FROM credentials WHERE name = 'admin'; , visto que todo o código que apareceria a seguir ao indicado passará a ser comentário devido ao #.

![Alt text](/images/img8.png)

Ao clicar em login entramos na conta do admin como esperado, e temos então acesso aos dados de todos os outros utilizadores.

![Alt text](/images/img7.png)

**Task 2.2: SQL Injection Attack from command line**

Esta tarefa é semelhante à anterior, só que desta vez temos que aceder ao site através da command line. Para isso usamos um pedido GET com o comando curl. Usando o exemplo que nos é dado no guião, mudamo-lo um pouco para que o username corresponde-se à mesma string que usamos na task anterior. Para isso usamos %27 (equivalente a ') e %23 (equivalente a #).

![Alt text](/images/img9.png)

Ao correr este comando, somos apresentados com o HTML da página do admin, esta que contém todas as informações dos utilizadores.

![Alt text](/images/img10.png)
![Alt text](/images/img11.png)

**Task 2.3: Append a new SQL statement**

Nesta task utilizamos o mesmo método de acesso à conta admin usado na task 2.1, mas desta vez adicionamos uma nova query após o ;, ficando com o input "admin'; UPDATE credentials SET Password = '1' WHERE Name = 'admin'; #", que em teoria para além de entrar na conta de admin, mudaria a sua hashed password para 1.

![Alt text](/images/img12.png)

No entanto foi nos apresentado um erro por causa do que foi dito anteriormente não acontecer devido a uma particularidade no código que nos é dado. No código é usada a função mysqli::query, esta que apenas permite uma query correr (ao contrário de mysqli::multi_query, que permite que várias queries sejam executadas), o que significa que a partir do momento em que mysqli::query chega à segunda query, a mesma não corre e apresenta erro. Esta informação foi retirada do manual do sistema de sql, nomeadamenta das páginas https://www.php.net/manual/en/mysqli.quickstart.multiple-statement.php, https://www.php.net/manual/en/mysqli.multi-query.php e https://www.php.net/manual/en/mysqli.query.php.

![Alt text](/images/img13.png)

### Task 3: SQL Injection Attack on UPDATE Statement

**Task 3.1: Modify your own salary**

Após acedermos ao perfil da Alice da mesma maneira que acedemos ao perfil do admin, acedemos então à secção de editar perfil. Analisamos então o ficheiro unsafe_edit_backend e descobrimos que o input nesta página tem exatamente o mesmo problema do input da página anterior. Aproveitando então esse conhecimento, inserimos no input adress a string "ali',Salary='123456. Com isto, adicionamos mais um parametro a mudar à query de update realizada pelo site, esta que é uma mudança de salário.

![Alt text](/images/img14.png)

Ao voltar à pagina anterior e comparar com os valores antigos, podemos ver que para além do adress da Alice estar agora definido como "ali", o seu salário também mudou para "123456".

Antes do edit

![Alt text](/images/img15.png)

Após o edit

![Alt text](/images/img16.png)

**Task 3.2: Modify other people’ salary**

Esta task acaba por ser semelhante à anterior, sendo que a única diferença será que temos que acabar a query mais cedo para a podermos redirecionar para o Boby e comentar todo o código que precede o nosso input. Para isso, o nosso input, desta vez em nickname, passou a ser "Bobo', Salary = '1' WHERE Name = 'Boby'; #".

![Alt text](/images/img17.png)

Após enviar este input, se acedermos à conta do admin podemos ver que os valores de nickname e salary do Boby foram mudados para os valores inceridos, tal como previsto.

![Alt text](/images/img18.png)

**Task 3.3: Modify other people’ password**

Nesta task reutilizamos a string de input da task anterior e mudamos salary='1' para Password = 'e2e3cc534e7ddfa925bca121c5007f0f5b5d961d', esta que antes de ser hashed por SHA1 será "gotcha", ficamos então com a string "Bobo', Password = 'e2e3cc534e7ddfa925bca121c5007f0f5b5d961d' WHERE Name = 'Boby'; #".

![Alt text](/images/img19.png)

Após o input, se tentarmos então aceder ao perfil do Boby usando a password gotcha, podemos ver que conseguimos, e como tanto o nome como a password são apresentados no url da página podemos demonstrar que de facto usamos o nome Boby e a password gotcha.

![Alt text](/images/img20.png)
