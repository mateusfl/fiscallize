[[Wiki de Produto/Iniciando na Lize/Banco de dados para testes|Banco de dados para testes]]

# Iniciando na Lize

Este documento apresenta um guia passo a passo para que o usuário possa iniciar um projeto em Python e Django que está no GitHub.

## Baixar código

O primeiro passo é baixar o código do projeto. Para isso, o usuário deve acessar o repositório do projeto no GitHub e clicar no botão "Clone or download". Em seguida, ele deve copiar a URL fornecida.

No terminal, o usuário deve navegar até o diretório em que deseja salvar o projeto e digitar o seguinte comando:

`git clone [URL do repositório]`

Isso fará com que o código do projeto seja baixado para o diretório escolhido.

## Criar ambiente virtual

Para evitar conflitos de dependências entre diferentes projetos, é recomendável criar um ambiente virtual para o projeto. Para isso, o usuário deve digitar os seguintes comandos no terminal:

`python3 -m venv [nome do ambiente]`

`source [nome do ambiente]/bin/activate`

Esses comandos criarão um ambiente virtual e o ativarão.

## Instalar dependências

Para instalar as dependências do projeto, o usuário deve navegar até o diretório raiz do projeto e digitar o seguinte comando no terminal:

`pip install -r requirements.txt`

Isso instalará todas as dependências do projeto listadas no arquivo `requirements.txt`.

## Configurar arquivo .env

O arquivo `.env` contém as configurações sensíveis do projeto, como chaves de API e informações de banco de dados. Para configurar esse arquivo, o usuário deve copiar o arquivo `.env.example` para `.env` e preencher as informações necessárias.

## Adicionar banco de dados

Se o projeto exigir um banco de dados, o usuário deve criar um banco de dados vazio com o nome especificado no arquivo `.env`. Em seguida, ele deve executar as migrações do Django para criar as tabelas do banco de dados:

`python manage.py migrate`

## Rodar projeto

Por fim, o usuário pode rodar o projeto com o seguinte comando:

`python manage.py runserver`

Isso iniciará o servidor de desenvolvimento do Django e o usuário poderá acessar o projeto em seu navegador no endereço `http://localhost:8000/`.

Com esses passos, o usuário deverá ser capaz de iniciar o projeto em Python e Django que está no GitHub.