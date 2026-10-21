# MAQ-001 — Piloto de Estoque de Segurança com Dados Sintéticos e Machine Learning

## Visão geral

Projeto acadêmico desenvolvido no contexto da disciplina MAQ-001 (UERJ), com o objetivo de analisar a variabilidade das revisões de um plano de produção e avaliar sua influência no dimensionamento de estoque de segurança de componentes (Part Numbers — PNs).

O estudo utiliza **dados integralmente sintéticos** para demonstrar um fluxo reproduzível de análise exploratória, avaliação histórica de cobertura e experimentação com modelos de Machine Learning. Os resultados **não representam indicadores reais de uma empresa** e não constituem recomendação de estoque para aplicação operacional.

## Objetivos

- Examinar a trajetória das revisões do plano de produção entre o horizonte de lead time e o plano final.
- Distinguir os **aumentos finais** dos **picos intermediários** observados após o horizonte de lead time.
- Simular alternativas de estoque de segurança e avaliar sua cobertura histórica.
- Traduzir o estoque de segurança de máquinas equivalentes em unidades de componentes, considerando a utilização de cada PN.
- Comparar um modelo de referência (*baseline*) com Ridge Regression e Random Forest em um experimento preditivo exploratório.

## Base de dados e premissas

| Característica | Configuração do piloto |
|---|---|
| Natureza dos dados | Sintéticos |
| Modelo estudado | MAQ-001 (identificação fictícia) |
| Histórico de origem | 24 meses |
| Revisões do plano na planilha | 240 registros (237 considerados na execução de 20/10/2026, após exclusão de 3 revisões futuras) |
| Meses consolidados na análise | 21 |
| Componentes | 13 PNs fictícios |
| Fornecedores | 3 fornecedores fictícios |
| Lead time | 5 meses |
| Consumo no cenário analisado | 1 unidade de cada PN por máquina |

A elegibilidade dos meses depende da consistência das revisões e da existência dos horizontes necessários à comparação. A quantidade de observações efetivamente usada em cada análise pode ser menor que o total bruto.

## Metodologia

### 1. Importação e validação

O notebook lê a planilha sintética, verifica a estrutura das abas e trata a série de revisões do plano de produção. O notebook definitivo `Piloto_Estoque_Seguranca_MAQ001_APRIMORADO_COM_ROTEIRO.ipynb` utiliza as abas `L'On`, `FORN-A`, `FORN-B` e `FORN-C`.

São necessárias datas e horizontes coerentes, quantidades inteiras não negativas e revisões válidas no horizonte de lead time e no horizonte final (0). Meses sem as observações exigidas não são elegíveis para determinadas comparações.

### 2. Variabilidade do plano

Para cada mês elegível, o estudo compara o plano observado no horizonte de lead time com o plano final e acompanha as revisões intermediárias posteriores. A análise distingue:

- **Aumento final:** diferença positiva entre o plano final e o plano no horizonte de lead time.
- **Pico intermediário pós-LT:** maior exposição positiva observada ao longo das revisões posteriores ao horizonte de lead time.

Essa distinção é essencial: uma política que cobre todos os aumentos finais observados não necessariamente cobre todos os picos intermediários.

### 3. Simulação de estoque de segurança

O projeto avalia empiricamente alternativas de estoque de segurança, em **máquinas equivalentes**, e apresenta a cobertura histórica das variações observadas. Para os PNs com consumo de uma unidade por máquina, duas máquinas equivalentes correspondem a **duas unidades por PN**.

A cobertura obtida é retrospectiva e depende da base e do recorte utilizados. Ela não equivale a uma probabilidade comprovada de atendimento futuro.

### 4. Experimento de Machine Learning

A etapa complementar compara três abordagens:

1. **Baseline:** referência simples para comparação.
2. **Ridge Regression:** modelo linear regularizado.
3. **Random Forest:** modelo baseado em árvores.

A avaliação preserva a ordem temporal das observações e utiliza métricas de erro absoluto médio (**MAE**) e raiz do erro quadrático médio (**RMSE**). No experimento apresentado, houve 15 observações elegíveis e três meses reservados para teste.

## Resultados da execução apresentada

### Estoque de segurança e cobertura

- **Estoque de segurança empírico analisado:** 2 máquinas equivalentes.
- **Equivalência para PNs com consumo 1:1:** 2 unidades por PN.
- **Cobertura histórica dos aumentos finais:** 100%.
- **Cobertura histórica dos picos intermediários pós-LT:** 90,5%.

### Comparação dos modelos

| Modelo | MAE | RMSE |
|---|---:|---:|
| Baseline | 0,333 | 0,577 |
| Ridge Regression | 0,424 | **0,503** |
| Random Forest | 0,518 | 0,567 |

**Interpretação:** o baseline apresentou o menor MAE, enquanto Ridge apresentou o menor RMSE. Não foi demonstrada superioridade consistente do Machine Learning em relação à referência simples. Os resultados correspondem à execução com data de corte de **20/10/2026**; datas de corte diferentes podem alterar a amostra e as métricas.

## Como executar no Google Colab

1. Abra no Google Colab o notebook [`Piloto_Estoque_Seguranca_MAQ001_APRIMORADO_COM_ROTEIRO.ipynb`](Notebook%20/Piloto_Estoque_Seguranca_MAQ001_APRIMORADO_COM_ROTEIRO.ipynb) publicado neste repositório.
2. Execute as células na ordem apresentada (se necessário, instale as dependências indicadas em `requirements.txt`).
3. Quando solicitado, carregue a [planilha sintética](dados/Planilha_Modelo_MAQ-001_ORIENTADA%281%29.xlsx), `Planilha_Modelo_MAQ-001_ORIENTADA(1).xlsx` (ou selecione o mesmo arquivo com nome diferente no diálogo de upload).
4. Confira os parâmetros `MODELO = 'MAQ-001'`, `LT_MESES = 5` e a `DATA_CORTE` (por padrão, a data da execução).
5. Preserve os nomes e a estrutura das abas esperados **pela versão do notebook escolhida**. No notebook definitivo, as abas são `L'On`, `FORN-A`, `FORN-B` e `FORN-C`.
6. Analise as tabelas, gráficos, matriz de cobertura, datasheet e resultados de Machine Learning gerados pelo notebook.
7. Quando disponíveis, confira os arquivos exportados e o relatório PDF executivo.

**Atenção:** publique o notebook e a planilha da **mesma versão**. Outras versões de desenvolvimento podem esperar nomes de abas distintos. A data de corte também pode alterar a amostra final e as métricas, dependendo da configuração do notebook.

### Ambiente de execução e reprodutibilidade

As métricas da apresentação foram reproduzidas no Google Colab com **scikit-learn 1.6.1, NumPy 2.1.3 e pandas 2.2.3**, data de corte de **20/10/2026**. Consulte `requirements.txt` para as versões de referência. O notebook utiliza também `matplotlib`, `reportlab` e `ipywidgets`.

**Atenção:** uma validação local com scikit-learn 1.8.0 apresentou pequena diferença nas métricas do Random Forest. Para comparar os resultados da apresentação, prefira o ambiente documentado. A diferença entre versões é uma hipótese plausível, ainda não isolada experimentalmente como causa única.

Para reproduzir a mesma janela de dados, ajuste temporariamente a linha `DATA_CORTE` na primeira célula de código para `pd.Timestamp('2026-10-20')`. Por padrão, o notebook usa a data atual e os resultados poderão mudar. Isso não altera a metodologia; restaure o padrão após a verificação.

## Organização do repositório

```text
-Projeto-Pr-tico-de-Machine-Learning/
├── README.md
├── requirements.txt
├── Notebook/
│   └── Piloto_Estoque_Seguranca_MAQ001_APRIMORADO_COM_ROTEIRO.ipynb
├── dados/
│   └── Planilha_Modelo_MAQ-001_ORIENTADA(1).xlsx
├── Documentação/
│   ├── GUIA_APLICACAO_EMPRESARIAL.md
│   └── CONFERENCIA_PUBLICACAO.md
└── Apresentação/
    └── INSTRUCOES.md
```

A pasta [`Apresentação/`](Apresenta%C3%A7%C3%A3o/INSTRUCOES.md) contém orientações para incluir o link do vídeo e os slides finais após a revisão. O [guia de aplicação empresarial](Documenta%C3%A7%C3%A3o/GUIA_APLICACAO_EMPRESARIAL.md) está em `Documentação/`.

## Limitações e próximos passos

- A base é sintética e representa um cenário controlado.
- A quantidade de observações temporais elegíveis para ML é pequena.
- A cobertura histórica não comprova nível de serviço futuro.
- A análise de picos intermediários não deve ser confundida com a análise de aumentos finais.
- A adoção operacional exigiria dados reais autorizados, validação temporal adicional, avaliação de custos, critérios de serviço e acompanhamento conjunto com as áreas responsáveis.

## Privacidade e uso acadêmico

Este repositório deve conter somente dados e identificadores fictícios. Não devem ser publicados dados empresariais confidenciais, informações internas de fornecedores ou registros reais sem autorização.

## Status

**Piloto acadêmico demonstrativo.** A metodologia de análise e simulação foi implementada sobre dados sintéticos; a etapa de Machine Learning é exploratória e ainda não foi validada para decisão operacional.
