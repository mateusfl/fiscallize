A infraestrutura da Lize está primordialmente localizada na Digital Ocean (DO), com alguns serviços auxiliares executando na Google Cloud Plataform e Microsoft Azure. Os tópicos a seguir detalharão com maior abrangência os serviços da plataforma e a cloud relacionada.

## Digital Ocean

O monolito da Lize é executado no kubernetes da Digital Ocean (DOKS), o qual utiliza um banco de dados gerenciado (PostgreSQL) com duas instâncias _readonly._ Utiliza-se também um banco de dados em memória gerenciado (Redis) para gerenciamento de sessão do Django e cache. O tráfego é recebido por um load balancer da DO, cuja função é encaminhar o tráfego para os pods e gerenciar o SSL de modo automático. Os arquivos estáticos são armazenados no block storage da DO, o Spaces. Esse armazenamento possui políticas de _lifecycle_ a fim de remover determinados arquivos de acordo com uma retenção estabelecida. O diagrama simplificado da infraestrutura é exibido abaixo:

  

![[Untitled.png]]

## Google Cloud Plataform

Como descrito na documentação do [[Módulo de gabaritos]], a infraestrutura da GCP é utilizada para a realização da impressão de páginas em PDF a fim de serem exportados como malote de provas. Para esta finalidade, o Google Cloud Run é o principal recurso utilizado, o qual permite a execução de containers em ambiente serverless, o que reduz o custo e aumenta a escalabilidade.

Além da exportação de malotes, utiliza-se o Google Cloud Vision (OCR) como identificação secundária para QRCodes que não sejam identificados pela leitra tradicional do módulo de gabaritos. O consumo desse serviço se dá por meio de API rest.

Um terceiro uso da GCP está descrito na página que trata da [[Análise de dados]]. Nesse recurso, utiliza-se o Google Bigquery como _data wharehouse_ e Google Cloud Functions para tratamento dos dados obtidos.data wharehousedata wharehouse

## Microsoft Azure

A azure está sendo utilizada como nuvem para serviços acessórios, apenas com serviços de máquinas virtuais (VPS) e bancos de dados gerenciados, conforme tabela a seguir:

|   |   |
|---|---|
|**Serviço**|**Recursos utilizados**|
|Sentry|Máquina virtual|
|Chatwoot|Máquina virtual e PostgreSQL gerenciado|
|Wiris|Máquina virtual|
|Staging|PostgreSQL gerenciado|