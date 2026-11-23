# Dicionário de dados: base_modelagem

Grão: uma linha por domicílio e corte. Gerado por `notebooks/02_rotulagem_e_variaveis.ipynb`.

| Coluna | Tipo | Definição | Ausentes |
|---|---|---|---|
| `corte` | str | Nome do corte temporal (treino, teste ou score). | 0 |
| `household_key` | int64 | Identificador do domicílio. | 0 |
| `frequencia` | int64 | Cestas distintas com pelo menos um item válido na observação. | 0 |
| `monetario` | float64 | Soma de sales_value dos itens válidos na observação. | 0 |
| `semanas_ativas` | int64 | Semanas distintas com compra na observação. | 0 |
| `recencia_dias` | int64 | Dias entre a última compra e o último dia da observação (0 = comprou no último dia). | 0 |
| `ticket_medio` | float64 | monetario / frequencia. | 0 |
| `n_departamentos` | int64 | Departamentos distintos comprados (diversidade de compra). | 0 |
| `tendencia_gasto` | float64 | log((gasto 2a metade + 1) / (gasto 1a metade + 1)); positivo = gasto crescendo. | 0 |
| `n_campanhas` | int64 | Campanhas recebidas cuja vigência cruza a observação. | 0 |
| `n_resgates` | int64 | Cupons resgatados com dia dentro da observação. | 0 |
| `recebeu_campanha` | int64 | 1 se n_campanhas > 0 (variável de H1). | 0 |
| `resgatou_cupom` | int64 | 1 se n_resgates > 0 (variável de H2). | 0 |
| `inativo_4s` | float64 | 1 se não houve compra válida nas 4 semanas após a observação. | 2432 |
| `inativo_8s` | float64 | 1 se não houve compra válida nas 8 semanas após a observação (rótulo principal). | 2432 |
| `inativo_12s` | float64 | 1 se não houve compra válida nas 12 semanas após a observação; vazio se a janela passa do fim da base. | 4849 |
| `renda` | str | Faixa de renda (INCOME_DESC); vazio sem demografia. | 4855 |
| `tamanho_domicilio` | str | Tamanho do domicílio (HOUSEHOLD_SIZE_DESC); vazio sem demografia. | 4855 |
| `tem_demografia` | int64 | 1 se o domicílio está em hh_demographic. | 0 |
