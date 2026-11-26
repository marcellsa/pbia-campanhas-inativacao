# Campanhas de marketing direto e inativação de clientes no varejo alimentar

Projeto da disciplina **Projeto em Business Intelligence e Analytics** (PUCRS Online), Fase 2.
Aluno: **Marcel Luís dos Santos Albuquerque**

**Dashboard publicado (Power BI):**
https://app.powerbi.com/view?r=eyJrIjoiMWQ0MmRjZTItOGFkNC00YmY3LTkzZTEtNmNlMGVhYTQyZDYxIiwidCI6ImIwYTdhYzliLWJlOTgtNDhhMC05N2FjLTU4ZWVlNjAwYTU1MyJ9&pageName=f93588c81137330dc612

---

## 1. Problema de negócio

Um varejista alimentar investe em campanhas de marketing direto e cupons. A área comercial precisa responder três perguntas:

1. Quantos clientes estão deixando de comprar e quanto isso custa?
2. Campanhas e cupons ajudam a reter clientes?
3. Quais clientes devem ser abordados primeiro?

**Definição de inativo (Fase 1):** domicílio que comprou na janela de observação (26 semanas) e não fez nenhuma compra válida nas **8 semanas seguintes**. As janelas de 4 e 12 semanas entram como análise de sensibilidade.

## 2. Hipóteses

| Hipótese | Enunciado | Critério de refutação |
|---|---|---|
| H1 | Domicílios que receberam campanha têm menor inativação, controlando por RFM | Sem diferença significativa a 5% |
| H2 | Domicílios que resgataram cupom têm menor inativação, controlando por RFM | Sem diferença significativa a 5% |
| H3 | A regressão logística completa (RFM + campanhas + cupons + diversidade + tendência) tem AUC maior que a logística só com RFM e que a regra de recência | IC 95% da diferença de AUC (bootstrap) inclui zero |

## 3. Principais resultados

Todos os números abaixo saem dos arquivos JSON em `reports/`, gerados pelos notebooks.

| Tema | Resultado |
|---|---|
| Base | 2.500 domicílios, 2.595.732 itens de venda, 276.484 cestas, 102 semanas |
| Inativação em 8 semanas | 8,8% no treino (213 de 2.409) e 10,1% no teste (244 de 2.417) |
| Comparação bruta (treino) | Quem recebeu campanha inativou menos: 2,8% contra 17,0% |
| **H1** | **Refutada.** Com controle de RFM: OR 0,717 (IC 95% 0,457 a 1,126; p = 0,148) |
| **H2** | **Refutada.** Com controle de RFM: OR 0,192 (IC 95% 0,026 a 1,410; p = 0,105) |
| Sensibilidade | Nenhuma das 10 combinações (hipótese × corte × janela) é significativa a 5% |
| **H3** | **Aceita parcialmente.** AUC no teste: regra de recência 0,815, logística RFM 0,862, logística completa 0,857. A completa supera a regra de recência (+0,042; IC 0,017 a 0,066), mas não a logística RFM (−0,005; IC −0,011 a 0,002) |
| Uso comercial | Nos 10% de maior risco, 43,0% de fato inativaram, contra 10,1% na média (lift de 4,26x) |
| Lista de risco atual | 244 domicílios em risco alto; receita esperada em risco de R$ 75.298,65 |

**Leitura:** a diferença bruta entre quem recebe e quem não recebe campanha desaparece quando se compara clientes de perfil de compra parecido (RFM). As campanhas não foram sorteadas: chegam a clientes que já compram mais. Os resultados são associações controladas, não efeitos causais.

## 4. KPIs do dashboard

| KPI | Definição | Valor |
|---|---|---|
| Taxa de inativação (8 semanas) | Inativos ÷ domicílios com rótulo no corte | 10,1% (teste) |
| Receita esperada em risco | Soma de probabilidade de inativação × gasto de 26 semanas | R$ 75.298,65 |
| Domicílios em risco alto | 10% do topo do score atual | 244 |
| Recência média | Dias desde a última compra (corte atual) | 18,0 |
| Frequência média | Cestas em 26 semanas (corte atual) | 31,0 |
| Ticket por cesta | Receita ÷ cestas (base toda) | R$ 29,24 |
| Taxa de resgate entre expostos | Expostos que resgataram cupom (corte atual) | 20,9% |

## 5. Arquitetura

```
Kaggle (CSV)  ->  data/raw
     |
     v
01 Perfil e limpeza ............. data/interim      (qualidade, flags, conferência da Fase 1)
02 Rotulagem e variáveis ........ data/processed    (cortes treino/teste/score, RFM, rótulo)
03 Testes H1 e H2 ............... reports/hipoteses (qui-quadrado, logística com controle RFM)
04 Modelo preditivo H3 .......... reports/modelo    (3 modelos, bootstrap de AUC, score de risco)
05 Exportação para o BI ......... data/processed/bi (esquema estrela + totais de conferência)
     |
     v
Power BI Desktop (modelo estrela, medidas DAX)  ->  Power BI Service (Publicar na Web)
```

**Cortes temporais (validação temporal):**

| Corte | Observação | Rótulo | Uso |
|---|---|---|---|
| treino | semanas 53 a 78 | 79 a 86 | treinar modelos e testar H1 e H2 |
| teste | semanas 69 a 94 | 95 a 102 | avaliar os modelos (H3) |
| score | semanas 77 a 102 | não há | lista de risco atual |

**Modelo dimensional (esquema estrela):**

| Tabela | Tipo | Grão |
|---|---|---|
| `fato_vendas` | fato | item válido da cesta |
| `fato_resgate` | fato | cupom resgatado |
| `fato_exposicao` | fato | domicílio × campanha recebida |
| `fato_domicilio_corte` | fato | domicílio × corte, com RFM, rótulo e score |
| `dim_domicilio`, `dim_produto`, `dim_campanha`, `dim_semana` | dimensões | domicílio, produto, campanha, semana |
| `resultado_hipoteses`, `resultado_modelos` | apoio | saídas dos notebooks 03 e 04 |

**Ferramentas:** Python 3.12 (pandas, numpy, scipy, statsmodels, scikit-learn, matplotlib), Jupyter no VS Code, Git e GitHub, Power BI Desktop e Power BI Service.

## 6. Dashboard

Quatro páginas, cada uma respondendo a uma pergunta de decisão:

| Página | Pergunta | Conteúdo |
|---|---|---|
| 1 | Como está a retenção de clientes? | KPIs, receita semanal por período, inativação por corte |
| 2 | Campanhas e cupons retêm clientes? | Comparação bruta, tabela de H1 e H2 com IC 95%, exposição e resgates por campanha |
| 3 | Quem a área comercial deve abordar primeiro? | Domicílios em risco alto, dispersão risco × gasto, lista priorizada |
| 4 | Quão confiável é o modelo? | AUC por modelo e métricas de validação |

Os prints das páginas estão em `reports/figuras/` (`pbi_p1_retencao.png` a `pbi_p4_modelo.png`).
O modelo do relatório, sem dados, está em `powerbi/pbia_campanhas_inativacao.pbit`. As medidas DAX ficam na tabela `_Medidas`, organizadas em pastas (Vendas, Inativação, Campanhas e Priorização).

**Conferência:** antes da montagem dos visuais, as medidas foram comparadas com os totais de `reports/bi/conferencia_bi.json` e com as saídas dos notebooks. Todos os valores bateram (por exemplo, receita de itens válidos de 8.057.443,80, 275.539 cestas e receita esperada em risco de 75.298,65).

## 7. Estrutura do repositório

```
pbia-campanhas-inativacao/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/               CSVs originais (não versionados)
│   ├── interim/           dados tratados (não versionados)
│   └── processed/         bases finais e esquema estrela (não versionados)
├── notebooks/
│   ├── 01_perfil_e_limpeza.ipynb
│   ├── 02_rotulagem_e_variaveis.ipynb
│   ├── 03_testes_h1_h2.ipynb
│   ├── 04_modelo_preditivo_h3.ipynb
│   └── 05_exportacao_bi.ipynb
├── reports/               números (JSON), tabelas (CSV) e figuras de cada etapa
├── docs/
│   └── dicionario_dados.md  definição de cada coluna da base de modelagem
└── powerbi/               modelo do relatório Power BI (.pbit, sem dados)
```

## 8. Como reproduzir

1. Clone o repositório e crie o ambiente:
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```
2. Baixe os dados em https://www.kaggle.com/datasets/frtgnn/dunnhumby-the-complete-journey e extraia os CSVs em `data/raw/` (o `causal_data.csv` não é usado).
3. Rode os notebooks 01 a 05, em ordem.
4. Confira os totais em `reports/bi/conferencia_bi.json`: receita de itens válidos 8.057.443,80 e 275.539 cestas.
5. Abra `powerbi/pbia_campanhas_inativacao.pbit` no Power BI Desktop e aponte a origem para os CSVs de `data/processed/bi/`.

Cada notebook termina com uma conferência dos seus números e grava um JSON em `reports/`.

## 9. Dados e licença

- Fonte: dunnhumby "The Complete Journey", cópia publicada no Kaggle.
- A licença (ODbL / DbCL) foi definida por quem publicou no Kaggle, e não pela dunnhumby. Por isso os CSVs **não são redistribuídos** neste repositório: cada pessoa baixa da fonte original.
- A base é anonimizada: os domicílios são identificados apenas por `household_key`.

## 10. Limitações

- As campanhas não foram distribuídas ao acaso; H1 e H2 medem associações controladas por RFM, não efeitos causais.
- Só 801 dos 2.500 domicílios (32%) têm dados demográficos, e no corte de treino nenhum deles ficou inativo. A moderação por renda e tamanho do domicílio foi feita no corte de teste e é exploratória.
- A base não tem datas de calendário: dias e semanas são números sequenciais.
- O Power Query do relatório aponta para caminhos locais; para atualizar os dados, ajuste a origem dos CSVs.

## 11. Mudanças em relação à Fase 1

O desenho da Fase 1 foi mantido. Foram definidos detalhes que estavam em aberto: janela de observação de 26 semanas, a base ter 102 semanas (e não 104) e a moderação feita só no corte de teste.
