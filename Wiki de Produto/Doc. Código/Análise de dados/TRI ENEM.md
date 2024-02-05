O cálculo de TRI modelo ENEM trata-se de uma request realizada para o endpoint abaixo. Por se tratar de um cálculo mais lento e custoso, apenas na primeira execução será realizada as operações de TRI. Execuções posteriores utilizarão um cache com validade de 24h. Os dados para consumo desse serviço estão descritos abaixo:

**Endpoint**: `[``**POST]**``GCP_FUNCTIONS_ANALYTICS_URL/tri`

**Body**:

```JSON
{
	"year": <int>,
	"exams": [<uuid>],
	"unities": [<uuid>] ,       \#Opcional
	"school_classes": [<uuid>], \#Opcional
	"students": [<uuid>] ,      \#Opcional
	"ignore_cache": <bool>      \#Opcional
}
```

**Response**:

```JSON
{
	"from_cache": <bool>, \#Caso false, o dado foi calculado na última execução
	"knowledge_areas_agg": [
		{
			"correct_count_avg": <float>,  \#Média de acertos na área do conhecimento
	    "grade_avg": <float>,          \#Nota média na área do conhecimento
			"id": <uuid>,                  \#ID da área do conhecimento
			"name": <str>                  \#Nome da área do conhecimento
	],
	"students": [
		{
			"student_id": <uuid>,          \#ID do aluno
			"student_name": <str>,         \#Nome do aluno
			"school_class_id": <uuid>,     \#ID da turma
			"unity_id": <uuid>,            \#ID da unidade
			"knowledge_areas": [
					{
						"correct_count": <int>,  \#Número de acertos na área do conhecimento
						"grade": <float>,        \#Média na área do conhecimento
					  "id": <uuid>,            \#ID da área do conhecimento
						"name": <str>            \#Nome da área do conhecimento
			]
		}
	]
}
```

  

Na aplicação existe uma rota de API cuja função é se comunicar com o GCF. Para consumi-la, deve ser utilizado o endpoint `/analitico/api/exams_tri/` e passar como parâmetro POST os IDs dos cadernos os quais deverão ser calculadas as notas, conforme exemplo a seguir:

```Shell
{
	"exams": [
		"09ac5c75-e7e4-4662-9adb-130423fb795f",
		"18322b71-4cdd-4fe9-9bc2-004290e4eb3f"
	],
	"students": [
		"ea179f31-5b4e-439d-8eed-dea4278b25ca",
		"d10f25ff-54c8-46e3-8527-7ebef31952c8"
}
```

O retorno será idêntico ao JSON recebido do GCF.

## Análise de itens

Como recurso auxiliar implementado para trazer mais informações acerca das questões de um conjunto de provas, existe um endpoint de análise de itens. A documentação está descrita abaixo:

**Endpoint**: `[``**POST]**``GCP_FUNCTIONS_ANALYTICS_URL/items`

**Body**:

```JSON
{
	"year": <int>,       \#Opcional
	"exams": [<uuid>]
}
```

**Response**:

```JSON
[{
	"bisserial": <float>,   \#Coeficiente bisserial
  "difficulty": <float>,  \#Dificuldade
	"question_id": <uuid>,  \#ID da questão
  "tri_a": <float>,       \#Discriminação (TRI)
  "tri_b": <float>,       \#Dificuldade (TRI)
  "tri_c": <float>        \#Acerto ao acaso (TRI)
}]
```

  

Na aplicação existe uma rota de API cuja função é se comunicar com o GCF. Para consumi-la, deve ser utilizado o endpoint `/analitico/api/items_params/` e passar como parâmetro POST os IDs dos cadernos e o ano de filtragem das respostas (opcional) conforme descrito acima.