A anulação de questões oferece duas possibilidades de uso, uma para o comportamento mais tradicional de dar a pontuação da questão anulada para todos os alunos, e uma para distribuir a nota da questão entre as demais questões da prova.

## Anulação e distribuição
Esse era o fluxo original da Lize para anulação de questões.

O cálculo da pontuação de cada questão no caderno, por padrão, não leva em conta as questões anuladas, efetivamente diminuindo o número de questões no caderno para fins de cálculo e assim, aumentando o valor de cada questão.


## Anulação e pontuação dada
O novo fluxo de anulação funciona da seguinte forma:

Ao anular uma questão em um caderno e dar a pontuação ao aluno, uma requisição é feita para criar um novo status de questão (**`StatusQuestion`**) registrando a anulação, porém com uma marcação no registro (**`annuled_give_score`**) indicando que a pontuação da questão deve ser contabilizada para fins de cálculo das notas dos alunos.
### locais de interesse
Os locais onde as notas dos alunos são calculadas foram alterados para considerar esse parâmetro, sendo eles:
- O método **`distribute_weights`**, hook presente nos modelos de Exams.
- O método **`get_annotation_count_answers`**, presente no manager de applications.
O método **`availables`** também foi alterado para selecionar corretamente as questões com base na mudança da função distribute_weights.

## Reverter anulação
É Possível reverter uma anulação de questão para casos em que, por exemplo, o usuário anule uma questão por engano.

Reverter uma anulação consiste basicamente em excluir o último status associado com a questão na prova, e tornar o status anterior a ele ativo. Isso acontece na APIView criada em Exams, localizado na variável **`revert_status_question_api_view`**.
	Excluir o último status implica na exclusão das últimas [[Sistema de Tags|tags de status]] associadas àquele status

A exclusão do status de anulação funciona nesse caso pois após anular uma questão, não é possível atribuir nenhum outro status a ela, então uma questão anulada sempre possui o último status como anulada.
