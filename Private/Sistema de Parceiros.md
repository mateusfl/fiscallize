O sistema de parceiros oferece a possibilidade de um cliente cadastrar usuários “parceiros” na plataforma. Usuários parceiros podem ser quaisquer pessoal que não esteja diretamente relacionado ao cliente.

  

## Tipos de parceiros

No momento da implementação do sistema só existe um tipo de parceiro implementado, o da equipe de impressão. A implementação de tipos diferentes de parceiros requer a atualização do modelo de parceiro, localizado em **Clients Models → Partner**, e também a atualização de algumas partes do projeto que renderizam telas diferentes dependendo do tipo de parceiro.

É recomendado usar **Ctrl+Shift+f (VScode)** e buscar por “is_printing_staff” para acessar os locais onde o tipo do usuário é usado, facilitando a adição de novos tipos em permissões e/ou telas condicionadas.

  

## Equipe de impressão

O tipo de parceiro “equipe de impressão” **(e a coordenação)** tem acesso a uma tela exclusiva para controle de impressões de aplicações presenciais.

A listagem de aplicações dessa tela fornece algumas informações básicas sobre a aplicação como: Quantos alunos estão cadastrados nela, data da aplicação, nome do caderno a ser impresso e as opções de impressão tanto do caderno, como do malote da aplicação.

Também é possível filtrar a listagem para exibir apenas aplicações impressas, etc.

### Controle das aplicações para imprimir

A coordenação precisa acessar a listagem de aplicações e marcar as aplicações como “pronta para impressão”, isso fará com que a aplicação apareça na listagem da tela de impressões.

Uma aplicação só poderá ser marcada como pronta para impressão caso um malote tenha sido gerado para ela pela coordenação. O malote é gerado na listagem de aplicações normalmente, e no momento, a opção de marcar como impresso **não** aparece até o malote ser gerado para a aplicação