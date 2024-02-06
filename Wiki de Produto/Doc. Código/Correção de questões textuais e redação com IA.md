---
Proprietário: Mateus Felipe
tags:
  - IA
---
A ideia é criar uma tela simples e de acesso livre onde as pessoas possam testar o recurso, fornecendo feedback para melhorias.
Implementação inicial contém apenas duas sessões, uma para envio de redações com padrão ENEM para correção, e uma para correção de questões discursivas.

==Informações abaixo são referentes à implementação inicial (janeiro/2024)==
## Correção de redação
### Fluxo atual esperado:
Usuário faz o envio de:
- Tema da redação
- Imagem do escaneamento de uma redação, contendo todo o texto do aluno
- (opcional) Todo o conteúdo do texto do aluno, para fins de teste da avaliação da IA em um cenário "ideal"

É necessário enviar apenas uma das duas opções de texto da redação, caso envie a imagem da redação, não é possível enviar o conteúdo em texto.

Ao fornecer as duas informações mínimas para a correção, elas serão enviadas a um endpoint que se comunicará com o serviço da OpenAI, usando o modelo do GPT 4 com função de visão computacional (ou o modelo de texto tradicional caso o usuário tenha enviado o texto pronto).

A resposta da IA deverá ser um JSON contendo a pontuação do aluno em cada uma das competências avaliadas pelo ENEM (passadas para a IA via prompt), e uma avaliação geral do desempenho do aluno, como uma espécie de feedback para ser lido pelo aluno

### Observações
Alguns pontos podem ser melhorados nesse fluxo:
#### envio de imagens
Enviar uma única imagem contendo todo o texto da redação não parece ser a melhor alternativa com o modelo atual de visão da OpenAI. A compressão da imagem pelo serviço afeta muito a leitura do texto da imagem, gerando erros gramaticais "artificiais" na redação do aluno ou ofuscando erros que ele possa ter cometido (Ex: pontuação, acentuação ou palavras com pouca legibilidade)
- **Possível solução 1**: separar de alguma maneira a imagem original em parágrafos e enviar os recortes dos parágrafos da imagem.
	- Essa abordagem diminui a compressão e torna a imagem mais legível para a IA, além de manter os erros ortográficos para correção, mas demanda mais trabalho e cuidado no tratamento da imagem.
- **Possível solução 2**: Usar algum outro serviço para fazer uma espécie de "leitura inicial" da imagem, gerando texto convertido, com o qual a IA tende a ter muito mais facilidade de trabalhar.

#### sobre a solução 2
Testando a possibilidade de usar o OCR do Google, os resultados são positivos em termos de tempo de resposta, que leva em torno de um segundo para a leitura de um texto, um tempo muito bom comparado com o ChatGPT.

Porém, em termos da qualidade da leitura, experimentando com uma redação com caligrafia relativamente ruim, os resultados são praticamente inúteis.
Exemplo de leitura de um **único parágrafo**:
	Imagem enviada:
	![[redacao_letraruim_01.png]]
	Resposta do OCR:
	![[Pasted image 20240206085005.png]]

O problema acima se repete, embora com menos frequência, em textos com caligrafia mais legível também.
Exemplo:
	Imagem enviada:
	![[redacao_letraboa_01.png]]
	Resposta do OCR:
	![[Pasted image 20240206085536.png]]

---
## Correção de questões textuais (discursivas)

### fluxo atual esperado
Usuário precisa enviar:
- O enunciado da questão
- A resposta esperada para a questão
- A resposta do aluno a ser corrigida

Fornecidas essas informações, uma requisição será feita ao serviço da OpenAI, passando todas as informações acima e solicitando um grau de "similaridade" entre a resposta do aluno e a resposta esperada.
	A similaridade deve considerar não somente palavras parecidas mas também se o aluno forneceu as informações solicitadas pela questão, mesmo que à maneira dele.

O modelo utilizado no momento é o GPT 4, que parece se sair melhor na correção do que o GPT 3.5 em termos de consistência na qualidade da correção.

A resposta da requisição deve ser um JSON contendo o grau de similaridade avaliado pela IA, e uma explicação do porque ela avaliou daquela maneira.

## prompts

### Redação
Você é um corretor de redações do enem e precisa corrigir a redação fornecida.

A redação é um texto dissertativo argumentativo, criado com base no tema: {theme}".

Analise o texto e faça a avaliação com base nas competências do ENEM:

1. Domínio da norma culta: Avaliação da adequação à ortografia, acentuação, hífen, uso de maiúsculas/minúsculas, separação silábica, regência verbal e nominal, concordância, pontuação, paralelismo, pronomes e crase.

2. Compreender o tema e não fugir do que é proposto: Avaliação das habilidades integradas de leitura e escrita, focando na organização da redação em torno do tema proposto, que representa a delimitação de um assunto mais amplo.

3. Selecionar, organizar e relacionar argumentos: Exigência de apresentar claramente uma ideia e argumentos coesos que justifiquem a posição assumida, destacando a importância do planejamento prévio e elaboração de um projeto de texto.

4. Conhecimento dos mecanismos linguísticos necessários para a construção da argumentação: Avaliação da estrutura lógica e formal entre partes da redação, incluindo coesão textual por meio de preposições, conjunções, advérbios e locuções adverbiais, garantindo uma sequência coerente e interdependência entre as ideias.

5. Elaborar uma proposta de intervenção que respeite os direitos humanos: Apresentar uma proposta de intervenção para o problema abordado que respeite os direitos humanos. Exigência de apresentar uma proposta de intervenção que respeite os direitos humanos, demonstrando preparo para exercer a cidadania e agir de acordo com esses princípios na realidade.

Crie uma review sucinta sobre o texto, que será lida pelo candidado como uma espécie de feedback, portanto, não mencione especificamente o tema, parágrafos ou competências nessa etapa.

Atribua uma nota variando de 0 a 200 ao candidato, para cada uma das competências acima, onde 0 representa nenhum domínio naquela competência e 200 representa um excelente domínio dela. Em caso de erros pequenos e pontuais em alguma das competências, desconsidere-os e a nota máxima pode ser atribuída.

### Questões discursivas
Você é um professor encarregado de corrigir questões discursivas.

Você receberá uma resposta de um aluno, e deverá compará-la com a resposta esperada pelo criador da questão.

O enunciado da questão é: {enunciation}

A resposta esperada pelo autor da questão é: {commented_answer}

Retorne uma porcentagem de similaridade entre as duas respostas. A similaridade não diz respeito apenas a palavras iguais mas ao contexto e a intenção da resposta do aluno, considere esses fatores. Faça um breve comentário sobre a razão da sua avaliação.