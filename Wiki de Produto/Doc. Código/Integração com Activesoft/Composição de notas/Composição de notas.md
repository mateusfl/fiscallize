---
Proprietário: Luiz Silva
tags:
  - Integração
Hora da última edição: 2023-09-19T20:36
---
Nessa tela é possível enviar as notas dos alunos em atividades realizadas na Fiscallize para a Activesoft como você pode observar no fluxograma abaixo.

  

![[Untitled 3.png|Untitled 3.png]]

  

A Ideia é que após a finalização de uma ou mais provas, os coordenadores que tenha acesso a Activesoft possam enviar as notas dos alunos para o sistema.

  

---

  

A Active disponibiliza rotas de API para tornar isso possível.. você pode acompanhar nessa documentação:

  

`[https://siga.activesoft.com.br/docs/](https://siga.activesoft.com.br/docs/)`

  

As rota utilizada para o envio é a seguinte:

**GET**

**/api/v{version}/correcao_prova/**

  

Mas para que possamos obter os dados corretos dos alunos é necessário fazer uma request para a rota, Passando os dados do aluno, como matrícula e turma… como pede na documentação

**GET**

**/api/v{version}/detalhe_boletim/**

  

Em `fiscallizeon/integrations/templates/integrations/notes_integrations.html` existe uma função chamado: `getBoletins(student, subject, idPhase, getAll)` que fica responsável por pegar apenas os boletins válidos para a disciplina ou fase de notas selecionados.

  

Nessa função é possível observar que existem alguns filtros, os filtros estão descritos na seguinte task do clickup: [https://app.clickup.com/t/864dphxff](https://app.clickup.com/t/864dphxff), que basicamente é isso que ta descrito logo abaixo

  

**TipoComposicao**: Somente disciplina com N ou D

**RequerNota**: Somente com "true"

**QtdNotas**: Somente disciplina diferente de null

**StFaseInformada**: Somente fase de nota como True, o que for False são fases de notas calculadas, que não tem importação de notas

  

**OBS:**

Os filtros são importantes para que a função retorna apenas os boletins necessários para a fase de notas selecionada.

  

---

## Sobre o Boletim do aluno

Esse é um exemplo de um boletim, e ele contém todas as informações necessárias para a integração de notas.

  

Quando a função `getBoletins` filtra os boletins de um aluno **(que geralmente vem um array com uns 200 boletins)**, sobram apenas 12-19 boletins e deles é possível extrair as disciplinas, fases de notas, cabeçalhos e valores de notas já lançados para o aluno em cada disciplina, os campos utilizados então em negrito.

  

{  
  
**"IdDisciplina": 1,**  
  
**"NomeDisciplina": "Língua Portuguesa",**  
"SiglaDisciplina": "PORT",  
  
**"TipoComposicao": "N",  
"RequerNota": true,  
**  
"StUsaNotaConceito": false,  
"CodigoSerie": "n11",  
  
**"IdFaseNota": 2431,**  
"NumeroFase": 1,  
  
**"CabecBoletim": "1ª UND",**  
"OrdemBoletim": 1,  
"StExibirFaltas": true,  
"StExibirNotaParcial": false,  
"StExibirNotaFase": true,  
"StExibirNotaFaseWeb": 1,  
"QtdNotas": 1,  
"NomeFormula": "1 composição",  
  
**"Cabec01": "Nota 1",  
"Cabec02": null,  
"Cabec03": null,  
"Cabec04": null,  
"Cabec05": null,  
"Cabec06": null,  
"Cabec07": null,  
"Cabec08": null,  
"Cabec09": null,  
"Cabec10": null,  
**  
"StExibirNota": true,  
  
**"NC01": 5.4,  
"NC02": null,  
"NC03": null,  
"NC04": null,  
"NC05": null,  
"NC06": null,  
"NC07": null,  
"NC08": null,  
"NC09": null,  
"NC10": null,  
**  
"Nota01": null,  
"Nota02": null,  
"Nota03": null,  
"Nota04": null,  
"Nota05": null,  
"Nota06": null,  
"Nota07": null,  
"Nota08": null,  
"Nota09": null,  
"Nota10": null,  
"MaxQtdNotas": 1,  
"NotaFase": " 5,4",  
"NotaFase_SemExibicao": 5.4,  
"Faltas": 0,  
"QuantAulasDadas": 17,  
"StNotaFaseExibirDisp": false,  
"SituacaoAtual": "Cursando",  
"SituacaoAlunoTurma": "Cursando",  
"SituacaoSistema": "A",  
"MediaTodasAsDisciplinasNaFase": 6.828571,  
"NumeroFaseClassificacao": null,  
"QtdFases": 7,  
"IdTurma": 1792,  
"IdSerie": 27,  
"IdPeriodo": 23,  
"IdAluno": 3108,  
"StDisciplinaDispensada": "A",  
"NumeroOrdemExibicao": 1,  
"MediaMinimaAprovacao": 0.0,  
"DataInicioExibicao": "2014-04-14T00:00:00",  
"StFaseInformada": true  
"NomeFase": "1º Trimestre",  
  
**"ValorNotaMaxima01": 10.0,  
"ValorNotaMaxima02": 10.0,  
"ValorNotaMaxima03": 10.0,  
"ValorNotaMaxima04": 10.0,  
"ValorNotaMaxima05": 10.0,  
"ValorNotaMaxima06": 10.0,  
"ValorNotaMaxima07": 10.0,  
"ValorNotaMaxima08": 10.0,  
"ValorNotaMaxima09": 10.0,  
"ValorNotaMaxima10": 10.0  
**  
}