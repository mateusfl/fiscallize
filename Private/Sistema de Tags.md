O sistema de tags de questões consiste no cadastramento de tags contendo os motivos da ação para cada vez que uma questão sofrer alterações em situações específicas.

## Possibilidades

Existem, até então, duas possibilidades de requisição de tags ao usuário, são elas:

- O status de **revisão** da questão foi alterado para algum dos status abaixo:
    - Correção
    - Reprovada
    - Anulada

Nesse caso, ao finalizar a mudança no status da questão, um novo registro é criado associando a mudança no status com a lista de tags fornecidas pelo usuário.

Caso não sejam fornecidas tags no processo, um registro ainda é criado, mas associado a uma lista vazia de tags

- A questão sofreu uma **edição** e:
    - O usuário que está fazendo a edição não é o mesmo que criou a questão
    - O status do caderno pelo qual a questão está sendo acessada é diferente de “elaborando”

Nesse caso, um registro é criado para associar a edição da questão às tags selecionadas pelo usuário, caso não seja informada nenhuma, o registro ainda é criado associado com uma lista vazia de tags.

---

Em ambos os casos, **caso o cliente não tenha cadastrado tags no sistema** para aquela determinada ação, a tela de solicitação de tags não é exibida e o fluxo de alteração na questão segue normalmente.

  

## Modelos Usados

- QuestionTag (Clients → models)
    - Modelo base de tags. Mudanças nas questões são associadas a uma lista de instâncias desse modelo.
    - Criado em: Gerenciamento → Tags de Correção. O cliente deve especificar o tipo da tag, que determina aonde ela será usada, para revisões ou edições.
- QuestionTagStatusQuestion (Exam → models)
    - Modelo usado para associar um determinado registro de status de questão com uma lista de tags.
- QuestionHistoryTags (Question → models)
    - Modelo usado para associar uma edição de questão a uma lista de tags. Esse modelo possui um campo que guarda a pk de um registro no histórico da questão, tornando possível associar as mudanças feitas nela com as tags.

  

### Detalhes do processo

Para mudanças no status das questões, a criação da instância de QuestionTagStatusQuestion se dá através de uma requisição para a rota de API de criação do modelo, semelhante à criação do Status da questão em si.

Já para as edições de questão, foi utilizada a própria view de atualização de questão para criar a instância de QuestionHistoryTags, por ser mais fácil acessar o registro de histórico criado quando a questão é atualizada.