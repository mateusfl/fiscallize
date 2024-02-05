---
Proprietário: Luiz Silva
tags:
  - Integração
Hora da última edição: 2023-07-17T15:47
---
Na tela de sincronização de alunos e turma é bem simples, lá é possível trazer as coordenações, turmas e alunos que estão cadastrados na Activesoft.

  

Alguns modelos foram adaptados para que essa sincronização possa ser efetiva.

  

Para fazer o mapeamento de quais são as coordenações, turmas e alunos que vieram por meio da interação, a gente adicionou uma variável chamada `id_erp` nos seguintes modelos…

  

**SchoolCoordenation**

**SchoolClass**

**Student**

  

Essa variável é preenchida sempre que uma sincronização é bem sucedida…

  

As APIs utilizadas para as sincronizações podem ser encontradas no seguinte caminho dentro do projeto:

  

`fiscallizeon/integrations/apis/syncronizations.py`

  

Além das APIs de sincronização, eu preciso fazer a checagem das importações para ter um retorno preciso dos itens que já foram mapeados, e para isso existem algumas APIs que você pode encontrar em:

  

`fiscallizeon/integrations/apis/active.py`

  

Essas são responsáveis pela checagem de importação, mostrando no frontend um (bool) chamado `sync` quando um item já estiver no nosso banco de dados.

  

é tudo muito simples. porém vale a pela dar uma estudada com um **token de homologação** (Teste)