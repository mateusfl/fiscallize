A plataforma usa o banco de dados vetorial [weaviate](https://weaviate.io/) para busca por similaridade. Até janeiro de 2024 este banco está sendo utilizado para qualificação de questões na BNCC e assuntos.

## Executando localmente

A execução do weaviate localmente se torna simplificada com o uso do docker compose. Sendo assim, segue o YAML para execução:

```JavaScript
version: '3.4'
services:
  weaviate:
    command:
    - --host
    - 0.0.0.0
    - --port
    - '8080'
    - --scheme
    - http
    image: semitechnologies/weaviate:1.23.0
    ports:
    - 8080:8080
    volumes:
    - weaviate_data:/var/lib/weaviate
    restart: on-failure:0
    environment:
      QUERY_DEFAULTS_LIMIT: 25
      AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
      PERSISTENCE_DATA_PATH: '/var/lib/weaviate'
      DEFAULT_VECTORIZER_MODULE: 'text2vec-openai'
      ENABLE_MODULES: 'text2vec-openai,generative-openai'
      CLUSTER_HOSTNAME: 'node1'
volumes:
  weaviate_data:
```

Para executar o banco a partir desse YAML, via terminal entre no diretório do arquivo e execute `docker compose up`.

  

## Populando com dados iniciais

O cadastro de habilidades, competências e assuntos se dá por meio de commands django:

```Shell
./manage.py update_abilities --rebuild
./manage.py update_competences --rebuild
./manage.py update_topics --rebuild
```