## Sobre o projeto

Este projeto aplica árvores de decisão para prever o comportamento de reajuste de valor em contratos públicos de compras no Brasil, utilizando dados abertos do Portal Nacional de Contratações Públicas (PNCP).

A base foi construída a partir de 31.884 contratos assinados entre janeiro e junho de 2023, extraídos diretamente da API pública do PNCP.

## Problema e abordagem

A variável de interesse é a variação percentual entre o valor global atual do contrato e o valor inicial pactuado na assinatura. Como mais de 96% dos contratos não apresentam alteração de valor no momento da observação, o problema foi dividido em dois estágios:

**Estágio 1 - Ocorrência de reajuste**
Classificador binário que prevê se um contrato terá ou não alteração de valor, treinado sobre a base completa com balanceamento de classes (`class_weight='balanced'`).

**Estágio 2 - Magnitude do reajuste**
Condicional ao Estágio 1: entre os contratos que de fato sofreram alteração, classifica a magnitude em quatro faixas (Redução de Valor, Baixo, Médio/Alto, Extremo).

## Correção de censuramento temporal

Contratos cuja vigência ainda não havia terminado no momento da última atualização do registro no PNCP foram excluídos da base. Sem esse filtro, contratos jovens demais para terem tido chance de sofrer reajuste eram contabilizados como "sem alteração", distorcendo o desbalanceamento das classes.

## Variáveis preditoras

| Variável | Descrição |
|---|---|
| `tipoContrato.nome` | Tipo de instrumento jurídico da contratação (Empenho, Contrato, Carta Contrato, Comodato, Convênio, Concessão, entre outros) |
| `unidadeOrgao.ufSigla` | UF do órgão público contratante |
| `tipoPessoa` | Natureza jurídica do fornecedor (PJ, PF ou PE) |
| `valorInicial` | Valor do contrato no momento da assinatura, em reais |
| `meses_decorridos` | Duração total da vigência do contrato, em meses, calculada a partir das datas de assinatura e de fim de vigência |

## Modelagem

Árvores de decisão (`DecisionTreeClassifier`, critério de entropia) treinadas com `scikit-learn`, com `random_state` fixo para garantir reprodutibilidade dos resultados entre execuções. Avaliação feita via `train_test_split` estratificado e `classification_report`.

## Limitações conhecidas

- A base de contratos que efetivamente sofreram alteração de valor é pequena (dezenas a poucas centenas de registros, dependendo do estágio), o que limita a confiabilidade das métricas para as classes minoritárias.
- O filtro de censuramento temporal reduz o viés de contratos jovens, mas também diminui o volume total de dados disponível, especialmente para a classe "Redução de Valor".
- Os dados refletem uma janela de seis meses (jan-jun/2023) e podem não generalizar para outros períodos ou contextos econômicos.