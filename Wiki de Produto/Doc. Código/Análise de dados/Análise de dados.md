Atualmente a plataforma está migrando a arquitetura de análise de dados a fim de eliminar problemas de desempenho de consultas extensas no banco de dados principal (PostgreSQL). Para tal, está sendo utilizado o Google Bigquery (BQ) como data warehouse em conjunto com o Google Cloud Functions (GCF) para comunicação via API e tratamento dos dados. Os parágrafos seguintes detalharão o fluxo de alimentação do BQ e como obter os dados via GCF.

## Google Bigquery

Por se tratar de um modelo de análise de dados, os dados enviados ao Google Bigquery (BQ) são exportados da forma mais desnormalizada possível, a fim de se obter um melhor desempenho das queries e consequentemente menor custo. Desse modo, são exportados os seguintes modelos:

  

|   |   |
|---|---|
|**Modelo**|**Descrição**|
|Respostas|Estão agregados todos os dados de `OptionAnswer`, `FileAnswer` e `TextualAnswer`. Os principais dados dessa tabela são nome e ID do aluno, nome e ID do cliente, data e ID da aplicação, ID do `ApplicationStudent`, peso da questão, nota do aluno, tipo da questão, disciplina, área do conhecimento, habilidades e competências BNCC, turma do aluno e unidade|
|Questões|Estão agregados todos os dados de `ExamQuestion`. As principais colunas dessa tabela são ID, ID da questão, ID do caderno, peso, ordem, tipo da questão, se é questão de língua estrangeira, nome e ID da disciplina e área do conhecimento|

O envio dos dados para o BQ se dá por meio de dois commands `export_answers_bigquery` e `export_examquestions_bigquery`. Em síntese, cada command realiza uma query no banco de dados e a queryset retornada é salva no formato parquet utilizando-se pandas e pyarrow. Em seguida o arquivo parquet é enviado para o Google Cloud Storage e por fim, importado como uma tabela do BQ. A estrutura escolhida para armazenamento no BQ consiste em uma tabela por cliente por ano. Sendo assim, cada cliente vai possuir tabelas de resposta do BQ com a seguinte nomenclatura `[answers|examquestions]_[client_id]_[ano]`.

  

## Google Cloud Functions

O processamento de dados oriundos do Bigquery é feito pelo Google Cloud Functions (GCF) em virtude da necessidade de pós processamento de dados utilizando bibliotecas como pandas e afins. Atualmente o GCF conta com os serviços implementados na listagem abaixo:

  

[[TRI ENEM]]

[[Simulador SISU]]