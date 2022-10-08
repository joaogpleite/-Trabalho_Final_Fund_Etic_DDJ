# Projeto Final - Disciplina Fundamentos e Ética no Jornalismo de Dados - Insper 2022 

:pencil: Passo-a-passo do projeto final da disciplina de Fundamentos e Ética no Jornalismo de Dados, do Master em Jornalismo de Dados, Automação e Data Storytelling do Insper. Escolhemos um banco de dados do TSE, tratamos os dados, fizemos perguntas e analisamos as respostas. Além das análises, a proposta foi a de indicar caminhos para uma possível pauta. 

**Integrantes:**
- Ingrid Fernandes - [@indgris](https://github.com/indgris)
- João Leite - [@joaogpleite](https://github.com/joaogpleite)
- Karina Ferreira - [@karina-ferreira](https://github.com/karina-ferreira)
- Will Araújo - [@colocaOuserAqui](https://github.com/link)

:unlock: Vamos lá:

## :put_litter_in_its_place: Limpeza

**Em cand_22 existem 29.261 linhas. Cada uma delas se refere a um candidato, ou ao menos deveria. Contudo, foram encontrados os seguintes problemas:**
- A coluna CPF tem registros duplicados (cand_22)
- A coluna SQ_CANDIDATO tem registros únicos dos candidatos duplicados para cargos diferentes e nomes diferentes
- A coluna NOMES tem registros duplicados
- VICES E SUPLENTES não são inseridos na contagem para receber doações
- A coluna receitas_candidatos_22 para doadores não possuia SQ_CANDIDATO_DOADOR para todos cadastrados
- Há registros que apontam para transferências recursos sem identificação do doador (foi colocado a sigla partidária para justificar a transferência);
- Muitos candidatos doadores estão sem siglas
- Em rápida averiguação, esses candidatos estão inseridos em alguma das situações acima ou então não relataram terem recebido receita, ficando com valor menor do que R$ 0,01. Por isso, não aparecem.

**Resultado**
- ORGANIZAÇÃO DOS CANDIDATOS QUE RECEBERAM:

 `SELECT
	receitas_candidatos_22.TP_PRESTACAO_CONTAS,
	receitas_candidatos_22.SQ_CANDIDATO,
	receitas_candidatos_22.NR_CPF_CANDIDATO,
	receitas_candidatos_22.NM_CANDIDATO,
	receitas_candidatos_22.DS_CARGO,
	receitas_candidatos_22.NR_CANDIDATO,
	receitas_candidatos_22.SG_PARTIDO,
	receitas_candidatos_22.SG_UE,
	receitas_candidatos_22.SQ_CANDIDATO_DOADOR,
	receitas_candidatos_22.DS_ORIGEM_RECEITA,
	receitas_candidatos_22.DS_FONTE_RECEITA,
	receitas_candidatos_22.VR_RECEITA,
	receitas_candidatos_22.DS_ESPECIE_RECEITA,
	receitas_candidatos_22.DS_RECEITA,
	receitas_candidatos_22.SQ_CANDIDATO_DOADOR,
	receitas_candidatos_22.NR_CPF_CNPJ_DOADOR,
	receitas_candidatos_22.NM_DOADOR,
	receitas_candidatos_22.DS_CARGO_CANDIDATO_DOADOR,
	receitas_candidatos_22.NR_CANDIDATO_DOADOR,
	receitas_candidatos_22.SG_UF_DOADOR,
	receitas_candidatos_22.SG_PARTIDO_DOADOR`
  
  - ORGANIZAÇÃO DOS CANDIDATOS DOADORES:

`SELECT
	receitas_candidatos_22.SQ_CANDIDATO_DOADOR AS SQ_DOADOR,
	receitas_candidatos_22.NR_CPF_CNPJ_DOADOR,
	receitas_candidatos_22.NM_DOADOR AS NM_DOADOR_REGISTRADO,
	receitas_candidatos_22.DS_CARGO_CANDIDATO_DOADOR,
	receitas_candidatos_22.NR_CANDIDATO_DOADOR,
	receitas_candidatos_22.SQ_CANDIDATO,
	FROM receitas_candidatos_22
LEFT JOIN cand_22 ON cand_22.SQ_CANDIDATO = receitas_candidatos_22.SQ_CANDIDATO_DOADOR
WHERE
	receitas_candidatos_22.DS_FONTE_RECEITA = "FUNDO ESPECIAL" AND
	receitas_candidatos_22.DS_ORIGEM_RECEITA = "Recursos de outros candidatos"
GROUP BY receitas_candidatos_22.SQ_CANDIDATO_DOADOR`

- AGREGAMENTO DO FEFC RECEBIDO PELO CANDIDATO POR MEIO DO PARTIDO x VALOR DOADO EM UMA TABELA COMPLETA:

`SELECT
	fefc_candidatos.SQ_CANDIDATO AS SQ_CAND_FEFC,
	fefc_candidatos.NM_CANDIDATO AS NM_CAND_FEFC,
	fefc_candidatos.VR_RECEITA AS VALOR_RECEBIDO,
	/*sum(fefc_candidatos.VR_RECEITA) as VALOR_FEFC,
	sum(cor_gen_receita_fefc.VR_RECEITA) as SOMA_DOADO*/
FROM
	fefc_candidatos
/*GROUP BY
	fefc_candidatos.SQ_CANDIDATO`
  
## :point_up: PERGUNTAS 

1. Qual é o total de candidatos que receberam dinheiro do Fundo Especial de Financiamento de Campanha (FEFC)? **18 mil candidatos**

`SELECT
	receitas_candidatos_22.TP_PRESTACAO_CONTAS,
	receitas_candidatos_22.SQ_CANDIDATO,
	receitas_candidatos_22.NR_CPF_CANDIDATO,
	receitas_candidatos_22.NM_CANDIDATO,
	receitas_candidatos_22.DS_CARGO,
	receitas_candidatos_22.NR_CANDIDATO,
	receitas_candidatos_22.SG_PARTIDO,
	receitas_candidatos_22.SG_UE,
	receitas_candidatos_22.SQ_CANDIDATO_DOADOR,
	receitas_candidatos_22.DS_ORIGEM_RECEITA,
	receitas_candidatos_22.DS_FONTE_RECEITA,
	receitas_candidatos_22.VR_RECEITA,
	receitas_candidatos_22.DS_ESPECIE_RECEITA,
	receitas_candidatos_22.DS_RECEITA,
	receitas_candidatos_22.SQ_CANDIDATO_DOADOR,
	receitas_candidatos_22.NR_CPF_CNPJ_DOADOR,
	receitas_candidatos_22.NM_DOADOR,
	receitas_candidatos_22.DS_CARGO_CANDIDATO_DOADOR,
	receitas_candidatos_22.NR_CANDIDATO_DOADOR,
	receitas_candidatos_22.SG_UF_DOADOR,
	receitas_candidatos_22.SG_PARTIDO_DOADOR
FROM receitas_candidatos_22
INNER JOIN cand_22 
	WHERE
	cand_22.SQ_CANDIDATO = receitas_candidatos_22.SQ_CANDIDATO AND
	receitas_candidatos_22.DS_FONTE_RECEITA = "FUNDO ESPECIAL" AND
	receitas_candidatos_22.DS_ORIGEM_RECEITA = "Recursos de partido político"
GROUP BY
	receitas_candidatos_22.SQ_CANDIDATO`
	
2. Qual é a porcentagem geral do FEFC doado às candidaturas? **R: xxxx**

3. Lista geral das candidaturas que mais doaram a partir do FEFC.


## :bar_chart: Análise
As queries resultaram em uma seleção de registros organizados que permitem o monitoramento do fluxo de doação de campanhas, de maneira a identificar doador, recebedor, origem do recurso e o destino e aplicação desses recursos nas campanhas que os receberam.

## :loudspeaker: Sugestão de pauta
Levantamento de candidatos que foram contemplados com o Fundo Eleitoral e doaram parte dele para outras candidaturas. Fazer um recorte dos 10 maiores doadores e recebedores (em porcentagem) e investigar as relações entre eles, bem como os gastos de campanha de quem recebeu a doação e como foram aplicados até agora. Quem recebeu também fez doação advinda do Fundo Eleitoral? Para quem? Estabelecer ligações.
