  

# **Funcionamento de login dos responsáveis (Pais)**

  

Para os pais acessarem a plataforma e conseguir ver os resultados dos seus filhos ele precisa ter um usuário na plataforma **Fiscallize**.

  

Para isso foi adicionado um `APP` chamado parents e um `MODELO` chamado `Parent`

Para salvar a relação entre o responsável e seus respectivos filhos.

  

## **Criando o login do responsável…**

O processo de criação do usuário do responsável é automatizado, O **responsável recebe um e-mail** com **validade de 15 dias** sempre que houver alguma aplicação de seu(s) filho(s) com resultado liberado e o responsável ainda não tenha criado um usuário.

  

A função responsável por isso é: `send_mail_to_first_access` que fica no model `Parent`.

  

Basicamente a função **checa** se o responsável **já tem** algum **usuário criado**, caso não tenha, será enviado um novo e-mail, e a **data de validade** para a **criação do usuário** será **atualizada** para **15 dias**

  

## Hash para criação do usuário

Quando uma notificação é enviada para o responsável um **hash** é criado, esse hash contém 80 caracters aleatórios que servirá para identificação do responsável.

o seguinte link será enviado para o responsável `{% url ‘parents:parent_signup’ hash=hash %}` para facilitar a utilização dessa rota existe uma maneira mais simples de conseguir esse link, no próprio modelo Parent existe uma função chamada `urls` que retorna um **dict** com um atributo chamado `user_creation`

  

## Como o hash é definido

Sempre que é criado um objeto `Parent` um **hook** é rodado através da lib **Django Lifecycle Hooks** a função responsável pela criação do hash é a `create_hash` do próprio modelo `Parent`

  

## Como é definido os alunos que serão vinculados ao responsável

Através do campo `responsible_email` do modelo `Student` , em outras palavras, todos os alunos que tem o email do responsável será vinculado a ele.  
  

Ex:

**Joãozinho**

Email do responsável: **fulano@fulano**

**Mariazinha**

Email do responsável: **fulano@fulano**

  

**fulano@fulano** será responsável por **Joãozinho** e **Mariazinha**