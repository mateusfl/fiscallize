O serviço de simulação do SISU tem a finalidade de verificar, baseado nos dados do SISU, em quais cursos o candidato seria aprovado baseado nas suas notas das grandes áreas e redação. Os dados para consumo desse serviço são descritos abaixo:

  

**Importante**: Atualmente a Lize possui apenas os dados do ano de 2022. Esse valor deve ser passado na querystrig dos endpoints abaixo.

## Metadados

Este endpoint tem como finalidade informar quais dados estão disponíveis para filtragem. Por exemplo, quais cidades, instituições e cursos estão disponíveis para refinar a busca do endpoint de simulação.

**Endpoint**: `[``**POST]**``GCP_FUNCTIONS_ANALYTICS_URL/sisu-data?year=[INT]`

**Body**:

```JSON
{
		"states": [<str>] \#Lista de siglas de estados para filtragem de metadados
}
```

**Response**:

```JSON
{
	"cities": [                  \#Lista de todas as cidades disponíveis
		{
			"city": <str>,           \#Nome da cidade
			"uf": <str>,             \#Sigla do estado
	],
	"courses": [                 \#Lista de todos os cursos disponíveis
		{
      "city": <str>,           \#Cidade do campus
      "course_grade": <str>,   \#Modalidade [LICENCIATURA | BACHARELADO]
      "course_id": <int>,      \#ID do curso
      "course_name": <str>,    \#Nome do curso
      "min_grade_ch": <float>, \#Nota mínima em Ciências Humanas
      "min_grade_cn": <float>, \#Nota mínima em Ciências da Natureza
      "min_grade_lc": <float>, \#Nota mínima em Linguagens e Códigos
      "min_grade_mt": <float>, \#Nota mínima em Matemática e Tecnologias
      "min_grade_r": <float>,  \#Nota mínima na Redação
      "passing_score": <float>,\#Nota de corte
      "shift": <str>,          \#Turno
      "uf": <str>,             \#Sigla do estado
      "unity_id": <int>,       \#ID da instituição de ensino
      "unity_initials": <str>, \#Sigla da insituição de ensino
      "unity_name": <str>,     \#Nome da instituição de ensino
      "weight_ch": <float>,    \#Peso Ciências Humanas
      "weight_cn": <float>,    \#Peso Ciências da Natureza
      "weight_lc": <float>,    \#Peso Linguagens e Códigos
      "weight_mt": <float>,    \#Peso Matemática e Tecnologias
      "weight_r": <float>      \#Peso Redação
    }
	],
	"unities": [                 \#Lista de todas as instituições disponíveis
		{
      "uf": <str>,             \#Sigla do estado
      "unity_name": <str>,     \#Nome da instituição de ensino
      "unity_initials": <str>, \#Sigla da insituição de ensino
    }
	]
}
```

  

## Simulador

Este endpoint tem como função filtrar os cursos os quais o candidato seria aprovado, baseado nos argumentos de cada grande área e a nota da redação. Diversas filtragens podem ser realizadas, com obrigatoriedade apenas para o campo `state` que deve ser passado como parâmetro body, conforme descrito abaixo:

  

**Endpoint**: `[``**POST]**``GCP_FUNCTIONS_ANALYTICS_URL/sisu?year=[INT]`

**Body**:

```JSON
{
		"grades": {
        "grade_cn": <int>,  \#Nota de ciências da natureza
        "grade_ch": <int>,  \#Nota de ciências humanaas
        "grade_mt": <int>,  \#Nota de matemática
        "grade_lc": <int>,  \#Noata de linguagens
        "grade_r": <int>    \#Nota da redação
    },
		\#Filtros
		"cities": [<str>],      \#Lista de cidades
		"courses_ids": [<int>], \#Lista de IDs de cursos
    "unities_ids": [<int>], \#Lista de IDs de instituições de ensino
    "waitlist": <bool>,     \#Caso true considera apenas lista de espera. Caso false, apenas chamada regular 
		"states": [<str>]       \#Lista de siglas de estados para filtragem (obrigatório)
}
```

**Response**:

```JSON
[{
        "approval_status": <str>, \#Status de aprovação [APPROVED|CLOSE|FAR]
        "call": <str>,            \#Chamada [REGULAR|ESPERA]
        "campus": <str>,          \#Campus do curso
        "city": <str>,            \#Cidade do campus
        "course_grade": <str>,    \#Modalidade [LICENCIATURA | BACHARELADO]
        "course_id": <int>,       \#ID do curso
        "course_name": <str>,     \#Nome do curso
        "course_shift": <str>,    \#Turno
			  "difference": <float>,    \#Diferença entre as notas de corte e do aluno
                                  \#Valores negativos indicam aprovação
        "unity_id": <int>,        \#ID da instituição de ensino
        "unity_initials": <str>,  \#Sigla da instituição de ensino
				"unity_name": <str>       \#Nome da instituição de ensino
}]
```

  

Na aplicação existem duas rotas de API cuja função é se comunicar com o GCF. Para consumi-las, devem ser utilizados os endpoints a seguir:

  

**Metadados**: `/analitico/api/sisu_data/`

Body:

```Shell
{
    "year": 2022, 
		"states": ['RN', 'PB', 'CE'] \#Opcional
}
```

  

**Simulador**: `/analitico/api/sisu/`

Body:

```Shell
{
    "year": 2022,
    "grades": {
        "grade_cn": 500,
        "grade_ch": 500,
        "grade_mt": 500,
        "grade_lc": 500,
        "grade_r": 500
    },
    "states": ["RN"]
}
```

Os retornos de ambas as rotas serão idênticos ao JSON recebido do GCF.