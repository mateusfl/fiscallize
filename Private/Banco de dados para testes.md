O banco de testes gerado automaticamente pelo django está quebrado por conta de uma ou mais migrações conflitantes no projeto, portanto, é necessário criar um banco manualmente para ser usado nos testes, e sempre que for executa-los, usar a flag —keepdb no comando de testes para o django não sobre-escrever o banco funcional.

### Passos para criar o banco de testes

==**Atenção:**== Os passos a seguir assumem um sistema operacional Linux. Para um sistema windows a estrutura de arquivos e a forma de executar comandos pode ser diferente.

1. Apagar (se houver) o banco com o prefiro “teste_” + nome do banco de desenvolvimento (Ex: test_fiscallizeon)
    1. Esse é o banco que o django busca na hora de rodar os testes e, caso não exista, cria um novo. É ele que precisa ser substituído
2. Criar um banco vazio com o mesmo nome (test_+nome do banco de desenvolvimento)
3. Copiar o arquivo de dump do banco funcional para uma pasta que o usuário “postgres” tenha acesso. O arquivo está localizado no projeto, em assets/db-schema.sql
    1. Exemplo de como fazer o processo acima:
        1. Acessar o caminho `~/etc/postgresql/14/main/`
        2. Criar uma pasta “backups” com `sudo mkdir backups`
        3. Acessar a pasta com `cd backups`
        4. Copiar o arquivo para a pasta atual com `sudo cp caminho-absoluto-para-o-projeto/assets/db-schema.sql ./` lembrando de substituir **“caminho-absoluto-para-o-projeto”** com o caminho na máquina, por exemplo: home/usuario/fiscallize-online/
4. Ainda dentro da pasta que contém o arquivo de dump, mudar o usuário para “postgres” com `sudo su postgres`
5. Fazer o dump do esquema funcional para o banco recém criado com `psql nome_do_banco_de_testes < db-schema.sql` lembrando de substituir **“nome_do_banco_de_testes”** com o nome dado ao banco criado no passo 2
6. Testar o funcionamento do banco rodando os testes do projeto com `./manage test` **`--keepdb`** `-v 2` lembrando de sempre usar “—keepdb” para que o django não substitua o banco que acabou de ser criado. Caso aconteça, todo o processo precisa ser repetido.
    1. A parte “-v 2” é opcional e serve apenas pra mostrar as migrações do banco sendo realizadas.