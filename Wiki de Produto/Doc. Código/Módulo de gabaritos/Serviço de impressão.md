  

  

A impressão de PDFs na Lize utiliza um serviço externo, a fim de evitar complexidades na base de código da plataforma, além de dispor de maior performance dado que é escrito em uma linguagem com esse propósito (Go). Atualmente está rodando principalmente no serviço Cloud Run da Google Cloud Plataform (GCP), mas possui containers executando na infraestrutura interna da Lize para renderização do preview da página de diagramação e realização dos merges. Tal execução local tem a finalidade de reduzir a latência e custos com banda de saída da GCP.

  

## Rotas

Como o serviço se trada de uma API REST, seguem abaixo o resumo das informações para cada rota:

### ==/status==

**Descrição:** Retorna 200 se o serviço estiver funcionando. Utilizada para healthcheck do kubernetes.

**Parâmetros:** não há.

### ==/print==

**Descrição:** retorna um arquivo PDF com o resultado.

**Parâmetros:**

```JSON
{
	"url": "https://remote.fiscallize.com.br/provas/id",
	"filename": "prova.pdf",
	"blank_pages": ["odd"|"between"]
	"wait_seconds": [true, false]
	"check_loaded_element": [true, false]
}
```

**Descrição dos parâmetros:**

- **url**: endereço o qual o serviço deve acessar para gerar o PDF. Atualmente são enviadas as views de impressão de prova, folhas de resposta objetivas e discursivas, lista de presença e lista de pátio (ensalamento)
- **filename:** nome do arquivo gerado no serviço, utilizado para evitar conflito de arquivos remotamente
- **blank_pages:** string que informa se o serviço deve retornar um PDF com um número par de páginas (”odd”) ou se deve inserir uma folha em branco entre cada uma das páginas geradas (”between”)
- **wait_seconds:** se true, o serviço aguarda 3 segundos antes de retornar o PDF. Utilizado em casos onde há grande renderização de javascripts. Atualmente apenas a impressão de cadernos seta esse parâmetro, a fim de evitar problemas na renderização de fórmulas matemáticas.
- **check_loaded_element:** se true, o serviço irá esperar a renderização de um elemento HTML com id = “loaded”. Utilizado em todas as views impressas atualmente na plataforma. Tem função de aumentar a resiliência quando o serviço está sobrecarregado, evitando que as páginas sejam impressas incompletas.

### ==/merge==

**Descrição:** retorna um arquivo PDF com o resultado do merge das URLs enviadas. Os arquivos precisam obrigatoriamente estar na especificação PDF 32000-1:2008.

**Parâmetros:**

```JSON
{
	"output_name": "merge.pdf",
	"urls": [
		"https://cdn.digitalocean.com/arquivo1.pdf",
		"https://cdn.digitalocean.com/arquivo2.pdf",
		"https://cdn.digitalocean.com/arquivo3.pdf",
		...
		"https://cdn.digitalocean.com/arquivoN.pdf"
	]
}
```

**Descrição dos parâmetros:**

- **output_name:** nome do arquivo gerado no serviço, utilizado para evitar conflito de arquivos remotamente
- **urls:** lista de URLs a serem baixadas para serem mergeadas

/

### ==/validate==

**Descrição:** retorna 200 se o arquivo PDF estiver de acordo com a especificação PDF 32000-1:2008. Essa rota ainda não é utilizada pela plataforma, mas pode vir a ser útil em caso de necessidade de merge de arquivos externos.

**Parâmetros:**

```JSON
{
	"content": <base64>
}
```

**Descrição dos parâmetros:**

- **content:** conteúdo do arquivo PDF no formato base64

  

## Execução local

Para evitar o uso de docker e simplificar a execução localmente, pode-se usar o executável do serviço (para linux). O download do arquivo pode ser feito [neste link](https://fiscallizeremote.nyc3.cdn.digitaloceanspaces.com/fiscallizeremote/binaries/pdf-service). Para a correta execução do serviço localmente, é necessárrio setar a variável de ambiente **AUTHORIZATION_TOKEN** com o valor de token gerado via admin (/admin/authtoken/tokenproxy/) para o usuário Administrador do Sistema ou qualquer outro com permissões de superusuário. Esse token dará permissão ao serviço acessa páginas restritas da Lize. Segue o exemplo de execução:

  

```Shell
chmod +x pdf-service \#Permissão de execução do arquivo (apenas uma vez)
export AUTHORIZATION_TOKEN=123456789ABCDEF
./pdf-service
```