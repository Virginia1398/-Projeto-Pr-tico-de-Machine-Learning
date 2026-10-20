# Guia de preparação dos dados — aplicação empresarial (versão piloto)

## Finalidade
Usar o mesmo método de análise de revisões de planos de produção para outros conjuntos de dados. **Nesta versão, a execução não é totalmente parametrizada**: o código fixa `MODELO = 'MAQ-001'`, `LT_MESES = 5`, a aba `L'On` e as abas `FORN-A`, `FORN-B`, `FORN-C`. Uma base diferente requer preservar essas abas e ajustar os dois parâmetros no início do notebook. Não há integração automática com SAP.

## Fluxo recomendado
1. Extraia o histórico de revisões de produção, preservando as datas originais de cada revisão e o mês de produção a que se refere.
2. Faça uma cópia da planilha-modelo. Atualize os **valores das linhas** nas abas de planejamento e de PNs; preserve os **nomes das colunas e das abas**.
3. Ajuste `MODELO` e `LT_MESES` na primeira célula de código para coincidir com a planilha; confira se todos os PNs têm o mesmo LT.
4. Execute as células do Colab em ordem. Confira as mensagens de validação, os meses elegíveis, o painel de indicadores, a matriz de cobertura e o relatório.
5. Valide a adequação do estoque com Compras/Planejamento/Logística antes de adotar qualquer recomendação. O ML está em fase de validação e não deve ser usado para decidir compras sem testes adicionais.

## Dicionário de entrada
**Aba `L'On` — revisões de produção**

| Coluna obrigatória | Pode alterar os valores? | Regra |
|---|---|---|
| `Modelo` | Sim | Deve coincidir com `MODELO` no código. |
| `Data da revisão` | Sim | Data real da revisão, válida como data do Excel. |
| `Mês de produção` | Sim | Mês-alvo da produção, data válida. |
| `Meses até produção` | Sim, com conferência | Número inteiro igual à diferença em meses entre mês-alvo e revisão. |
| `Plano de máquinas` | Sim | Inteiro não negativo. |

**Abas `FORN-A`, `FORN-B`, `FORN-C` — cadastro de PNs**

| Coluna obrigatória | Pode alterar os valores? | Regra |
|---|---|---|
| `PN sintético` | Sim | Identificador único entre todas as abas. O nome da coluna permanece mesmo com PNs reais. |
| `Modelo sintético` | Sim | Deve coincidir com `MODELO`. |
| `Qtd por máquina` | Sim | Número positivo; é multiplicador da demanda. |
| `Lead time (meses)` | Sim, com restrição | Todos devem ser iguais a `LT_MESES`; LT misto não é suportado. |
| `Fornecedor fictício` | Sim | Rótulo usado nas saídas; mantenha a coluna. |

**Não modificar sem adaptar o código:** nomes das abas, cabeçalhos, estrutura de uma linha por revisão/horizonte, critérios de cálculo e células do notebook. As abas `LEIA-ME` e `DICIONÁRIO` são documentais.

## Condições para um mês entrar na análise
É necessário existir **uma revisão exatamente `LT_MESES` antes** e **uma revisão no mês de produção (horizonte 0)**. Se faltar alguma, o mês é excluído; o notebook atual não escolhe automaticamente a última revisão anterior ao LT. Não pode haver duas linhas para o mesmo par `(Mês de produção, Meses até produção)`.

## Interpretação dos indicadores
- `Plano_LT`: quantidade planejada no momento da decisão de compra.
- `Plano_final`: quantidade no horizonte 0.
- `Aumento_final`: máximo entre 0 e (`Plano_final - Plano_LT`).
- `Pico_pos_LT`: maior quantidade observada nas revisões após LT.
- `Aumento_pico`: máximo entre 0 e (`Pico_pos_LT - Plano_LT`).
- `Min_pos_LT`, `Amplitude_pos_LT`: mínimo e diferença entre máximo e mínimo na trajetória pós-LT.
- **Matriz de cobertura:** ✓ quando o ES candidato cobre a necessidade adicional daquele mês; ✕ caso contrário. O critério de aumento final e o critério de pico pós-LT são distintos.
- Os indicadores por PN são multiplicados por `Qtd por máquina`. Os PNs de uma mesma máquina não constituem amostras independentes.

## Limitações e segurança
A cobertura é **histórica**, não garantia futura. O experimento de ML utiliza poucos meses independentes; comparar sempre com baseline e validar prospectivamente. Não carregar dados empresariais confidenciais em ambientes ou repositórios públicos sem autorização, revisão de segurança e regras internas de TI/LGPD. A planilha sintética é apenas demonstração. O professor solicitou dados reais e públicos; dados sintéticos não atendem literalmente esse requisito.


## Plano de validação futura com dados reais

**Situação:** o ML é um experimento de viabilidade, ainda não uma previsão validada para compras. A cobertura calculada é retrospectiva.

**Coletar:** histórico datado de cada revisão por mês de produção (incluindo revisão no LT e plano final), volume efetivamente produzido, PNs e consumo por máquina, lead time planejado e realizado por PN/fornecedor, pedidos e datas de entrega, estoque disponível, faltas, urgências e custo de manter estoque ou parar produção. Guardar a versão e a origem das extrações.

**Testar:** (1) integridade e ausência de vazamento temporal; (2) comparação com baseline e política empírica em múltiplas janelas temporais futuras (*walk-forward*); (3) MAE/RMSE, viés, taxa de cobertura, faltas, excesso de estoque e custo; (4) estabilidade por período, máquina, PN e fornecedor; (5) sensibilidade a revisões ausentes e LT variável; (6) piloto em paralelo (*shadow mode*) sem influenciar compras, seguido de aprovação conjunta de Planejamento, Compras, Logística e TI.

**Critério de liberação:** definir previamente metas de desempenho e risco com a empresa. Não ativar recomendações automáticas se o modelo não superar a referência simples ou se não houver evidência de cobertura e custo aceitáveis em dados reais fora da amostra.
