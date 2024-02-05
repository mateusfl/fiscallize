A plataforma Lize possui como uma das principais features a impressão e leitura de gabaritos. Por se tratar de uma funcionalidade mais complexa, esta documentação será dividida em dois tópicos distintos, abrangendo o módulo de exportação e o módulo de leitura. O módulo de exportação é responsável por permitir que o usuário crie cadernos personalizados, definindo opções de layout e impressão, bem como o modelo das folhas de resposta. Já o módulo de leitura é responsável por ler os gabaritos preenchidos pelos alunos e criar as respostas objetivas e discursivas na plataforma.

## Organização de filas do celery

Por se tratarem de tarefas de alto consumo de recursos e representarem ações críticas para os nossos clientes, os dois módulos descritos acima são processados por filas separadas em produção. Ou seja, tasks de exportação rodam em containers distintos daqueles que rodam tasks de leitura de gabaritos. É por essa razão que, ao executar o celery localmente, deve-se especificar o parâmetro -Q com o nome das filas. O se executar o celery localmente, não há necessidade de diferenciar as filas em diferentes workers. Sendo assim, deve-se especificar todas as filas no mesmo comando:

```Shell
celery -A fiscallizeon worker -l info -Q celery,merge,answer-sheet-print,exam-print,omr-import
```

## Exportação de malotes

Quando o usuário realiza uma exportação de malote, seja via listagem de aplicações ou ensalamentos, uma série de passos é executada para gerar um arquivo final com os documentos. Será explicada nessa documentação a exportação de malote de uma aplicação, todavia os passos aqui apresentados serão bastante similares àqueles relacionados ao ensalamento. Todas as tarefas de exportação de PDF utilizam o serviço de impressão externo, cuja documentação está descrita no documento anexo a seguir:

  

[[Serviço de impressão]]

  

### Passo a passo da exportação

Após a confirmação de impressão na modal de configuração do malote, será criada uma task do celery “_fiscallizeon/omr/tasks/export_answer_sheet.py”_ cujos parâmetros são:

- **application_pk:** str com UUID da aplicação
- **sheet_model**: modelo da folha de respostas objetivas (Lize, ENEM, Híbrida)
- **export_version**: versão de exportação (incremental e tem a função de evitar que exportações simultâneas da mesma aplicação se misturem no arquivo final)
- **blank_pages**: booleano, se _true_ o serviço de impressão imprimirá folhas em branco para viabilizar a impressão frente e verso
- **include_exams**: booleano, se _true_ incluirá os cadernos de prova no arquivo final
- **include_discursives**: booleano, se _true_ incluirá as folhas de resposta discursivas no arquivo final
- **exam_params**: dict com as configurações de impressão dos cadernos de prova (fonte, layout de página, modelo de cabeçalho etc). Consultar opções no arquivo “_fiscallizeon/exams/templates/dashboard/exams/includes/exam/print/modal_print.html_”
- **discursive_params**: dict com as configurações de impressão das folhas de resposta discursivas. Atualmente utiliza-se apenas a opção de separação por disciplina.

  

Antes de iniciar o processo de impressão, a task verifica se o malote conterá os cadernos de prova se a prova possui opções de randomização. Em caso afirmativo, executa a função de randomização “fiscallizeon.applications.randomization.randomize_application”.

  

A próxima etapa da exportação consiste em setar um array de tasks (_assets_tasks_) a serem executadas para cada application_student, baseado nos parâmetros (include_exams, include_discursives) ou se a prova possui questões objetivas e discursivas para incluir ou não a impressão dos PDFs.

  

As tasks appendadas ao array em questão são:

- **export_application_student_answer_sheet**: responsável por exportar as folhas de respostas objetivas. Possui como parâmetros:
    - **application_student_pk**: ID do application_student (str)
    - **output_dir**: caminho no diretório remoto (spaces) onde o arquivo PDF será salvo
    - **sheet_model**: modelo da folha de respostas (Lize, ENEM ou híbrido)
    - **blank_pages**: se devem ser adicionadas folhas em branco para impressão frente e verso (booleano)
    - **randomization_version**: versão de randomização (caso realizada no passo anterior)
- **export_application_student_discursive_sheet**: responsável por exportar as folhas de respostas discursivas. Possui como parâmetros:
    - **application_student_pk**: ID do application_student (str)
    - **output_dir**: caminho no diretório remoto (spaces) onde o arquivo PDF será salvo
    - **blank_pages**: se devem ser adicionadas folhas em branco para impressão frente e verso (booleano)
    - **print_params**: dict com as configurações de impressão (discursive_params)
- **export_exam_application_student**: responsável por exportar os cadernos de prova. Possui como parâmetros:
    - **application_student_pk**: id do application_student (str)
    - **output_dir**: caminho no diretório remoto (spaces) onde o arquivo PDF será salvo
    - **print_params**: dict com as configurações de impressão (exam_params)
    - **blank_pages**: se devem ser adicionadas folhas em branco para impressão frente e verso (booleano)
    - **randomization_version**: versão de randomização (caso realizada no passo anterior)
- **export_school_class_attendance_list:** diferenciando-se das tasks anteriors, é executada para cada turma da aplicação. Possui como parâmetros:
    - **application_pk**: ID do application_student (str)
    - **school_class_pk**: ID da school_class (str)
    - **output_dir**: diretório onde o arquivo PDF será salvo

  

Uma vez montado o array de tasks para impressão, utiliza-se o recurso de _chord_ do celery para executar as tarefas de impressão. Detalhes sobre a implementação de fluxos de tarefas no celery podem ser encontradas em [https://docs.celeryq.dev/en/stable/userguide/canvas.html](https://docs.celeryq.dev/en/stable/userguide/canvas.html)

  

Após concluídas todas as tasks de impressão o celery chama a task de merge, a qual fará a junção dos arquivos na estrutura correta. Para malotes de aplicação, é utilizada a task **group_answer_sheet_files**. Essa função possui os seguintes parâmetros:

- **application_pk**: ID da aplicação (str)
- **export_version**: versão da exportação. Utilizado para determinar quais arquivos devem ser agrupados
- **include_exams**: booleano, se _true_, deve incluir os cadernos no arquivo final
- **include_discursives**: booleano se _true_, deve incluir as folhas de resposta objetivas no arquivo final

  

A partir do ID da aplicação e versão da exportação, a task de merge irá ordenar as URLs dos arquivos gerados no passo inicial do _chord_ e fará uma request ao serviço de merge local (consultar documentação correspondente) que retornará um arquivo PDF. A task realiza esse procedimento para cada unidade que estiver relacionada à aplicação e ao final será gerado um arquivo ZIP com todos os PDFs das unidades, o qual será enviado para o spaces. A task é finalizada, alterando o seu status no celery e no campo “**status**” do modelo Application.

  

---

  

## Leitura de folhas de resposta

Após a aplicação das provas pela instituição, as folhas serão escaneadas e enviadas em PDF para a plataforma. Trata-se de uma tarefa um pouco mais complicada em relação à exportação, tendo em vista a quantidade de operações necessárias e os tipos de gabarito envolvidos. O fluxo de trabalho segue o esquema da figura abaixo:

  

![[Diagrama_sem_nome.drawio.svg]]

  

### Passo a passo da leitura

Após o usuário subir o arquivo PDF com o escaneamento das folhas de resposta, o sistema cria uma instância de OMRUpload e a task de processamento de folhas de resposta (_proccess_sheets)_ é executada, a qual consiste em uma chain (ver documentação do celery no passo anterior) de outras tasks com funções bem definidas. Essa arquitetura foi escolhida a fim de simplificar a manutenção do código, seguindo a ideia de [tasks idempotentes](https://docs.celeryq.dev/en/stable/glossary.html#term-idempotent), além de facilitar a execução em ambientes distribuídos como o kubernetes. Abaixo segue o detalhamento das tarefas mencionadas no fluxograma anterior:

  

- **Armazenamento de arquivo enviado:** executado na view de upload de gabarito, essa etapa consiste em salvar o arquivo enviado pelo usuário no spaces da Digital Ocean, apenas para finalidade de auditoria e debug em caso de problemas. Como não se trata de um arquivo essencial, seu aramazenamento é temporário e depois de alguns dias é removido do spaces.
- **Conversão de arquivo PDF em imagens:** executado pela task “_split_pdf_”, a função cria um diretório temporário, separa cada página do PDF em um arquivo JPG e salva o número de páginas processadas no OMRUpload. É importante que o diretório não seja alterado uma vez que em produção é utilizado um recurso de sistema de arquivos via rede (NFS) o qual utiliza esse diretório como canal de comunicação entre as máquinas.
- **Leitura do QRCode do cabeçalho:** executado pela task “_read_files_qr_”, essa função lê cada arquivo JPG em busca do QRCode de identificação do application_student (ou application, se for avulso). Utiliza a função “_fiscallizeon.omr.utils.process_header_qr_” cujo objetivo é tentar identificar o conteúdo do QR via libs locais ou OCR da Google Cloud Plataform (GCP) em caso de falha do primeiro método, fazendo uso do código do QR por extenso acima do cabeçalho da página. Uma vez identificado o conteúdo do QRCode, é realizado um split na string de modo que são obtidos os parâmetros:
    
    - **operation_type**: define se a folha de respostas é padrão (S), avulsa (A) ou randomizada (R). Neste último caso, é obtida também a versão da randomização.
    - **sequential**: trata-se do identificador sequencial do tipo de folha de resposta (1 - Lize objetivas, 2 - Enem, 8 - Discursivas 0.25 etc)
    - **object_id**: é o ID do objeto em questão. Se for uma folha de resposta padrão (S) o valor desse parâmetro será o ID do application_student. Se for avulso (A), será o ID da aplicação.
    
    Os dados do gabarito são então inseridos em um dict e appendados num array para tratamento pelas tasks seguintes.
    
- **Gerenciamento de páginas avulsas:** executado pela task “_handle_avulse_sheets_”, essa função tem como objetivo selecionar as páginas que tenham “A” como operation_type (avulsas), identificar a matrícula do aluno para criar o application_student correspondente e converter o operation_type para padrão (S). Desse modo, as tasks subsequentes não precisam lidar com essa particularidade. Atualmente apenas os modelos de folha objetiva e híbrida dão suporte ao campo de matrícula. Caso o cartão de respostas seja de algum outro modelo ou o sistema seja incapaz de identificar a matrícula do aluno corretamente, será criado um OMRError, para identificação manual pelo usuário.
- **Tasks de leitura das páginas:** as tasks descritas até então não processam nenhuma informação das respostas dos alunos, apenas metadados para identificação do aluno, modelo de gabarito, versão de randomização e outros detalhes menores. A partir desse ponto, o sistema irá de fato manipular as respostas, utilizando o recurso de group do celery para acelerar o processo de leitura. Referente à manipulação de respostas discursivas, são executados dois passos:
    
    - **Leitura de folhas objetivas:** executado pela task “_read_sheets_directory_fiscallize_”_,_ essa função realiza a leitura das respostas utilizando o mapeamento prévio do template JSON realizado na folha de respostas. Esse mapeamento é realizado a partir de uma modificação do projeto [OMRChecker](https://github.com/Udayraj123/OMRChecker) (branch master-legacy). Uma vez identificadas as respostas, o módulo de leitura retornará um dict com o numero da questão e a(s) alternativa(s) identificadas. A esse dict são adicionados os campos “category” (tipo de folha de respostas), “instance” (ID do application_student), “file_url” (caminho do arquivo lido no diretório temporário) e “randomization_version” (versão da randomização, caso exista). Os dicts de cada página lida são adicionados a um array que será passado para a task seguinte. Nesta etapa também serão criadas as instâncias de OMRStudents, associadas ao application_student correspondente.
    - **Criação das respostas objetivas:** executado pela task “_handle_answers_”, o array com as respostas lidas na etapa anterior é iterado e são criadas as OptionAnswer para cada elemento presente no dict. Caso hajam elementos do tipo discursivo no mesmo dict (gabaritos híbridos), serão criados TextualAnswers com a nota correspondente.
    
    Referente à leitura de respostas discursivas, as etapas de tratamento das folhas se dá com as tasks:
    
    - **Leitura de folhas discursivas:** executado pela task “crop_answer_area”, essa função possui o objetivo de identificar as áreas de resposta (utilizam-se técnicas de processamento de imagem para realçar as bordas pretas dos retângulos), cortar as áreas de resposta em arquivos distintos e identificar a questão do arquivo cortado utilizando-se do QRCode com o código do exam_question correspondente. As respostas identificadas são adicionadas a um array que será utilizado pela próxima task. Caso a exam_question não seja identificada, será criado um OMRDiscursiveError para identificação manual da questão pelo usuário.
    - **Criação das respostas discursivas:** executado pela task “_handle_discursive_answers_”, essa tarefa tem como finalidade identificar a nota dada pelo professor, utilizando técnicas de processamento de imagens (verificar função “__get_question_grade_” no mesmo arquivo da task), converter a cateforia da questão para FILE (caso seja TEXTUAL) e criar um FileAnswer associado, com o arquivo recebido da etapa anterior. Nesta etapa também serão criadas as instâncias de OMRStudents, associadas ao application_student correspondente.
- **Finalização do processamento:** executado pela task “_finish_processing_”, essa etapa consiste apenas em alterar o status do OMRUpload para FINISHED e remover o diretório temporário com os arquivos lidos.