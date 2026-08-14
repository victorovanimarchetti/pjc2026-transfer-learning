# O que transferir entre hospitais: dado, parâmetro ou modelo pronto

Código de apoio ao estudo sobre estratégias de transferência de modelos
preditivos entre unidades de pronto socorro, submetido ao 32º Prêmio Jovem
Cientista (CNPq, 2026), tema "Inteligência Artificial para o Bem Comum".

**Autor:** Victor Hugo Ovani Marchetti
**Orientação:** Prof. Dr. Alexandre Dias Porto Chiavegatto Filho, LabDaps, Faculdade de Saúde Pública da USP
**Coorientação:** Dr. José Arnaldo Shiomi da Cruz
**Aprovação ética:** CEP do Hospital IGESP, parecer 8.426.290, CAAE 96939326.0.0000.5450

---

## Os dados não estão aqui, e não estarão

Este repositório contém **método, não dado**. Os registros são assistenciais,
de pacientes, dado pessoal sensível sob o art. 11 da LGPD. Não são
distribuídos, nem em amostra, nem agregados a nível individual. As consultas de
extração também não são publicadas. O pipeline parte de um arquivo tabular cujo
contrato está descrito no notebook; quem tiver dados equivalentes reproduz a
análise inteira. As unidades aparecem anonimizadas.

## O estudo em uma frase

Numa rede com 2,4 milhões de altas de pronto socorro em 15 unidades de quatro
estados, comparamos oito estratégias de transferência (local, modelo externo
recalibrado, agrupamento de registros, continuação de boosting, e quatro
variantes com o modelo fundacional TabPFN) sob divisão temporal estrita
(treino 2024, teste 2025), com região de equivalência prática em vez de valor
de p. O resultado firme: **na faixa de 39 a 770 eventos anuais de treinamento,
tudo que usa dado local supera o modelo emprestado**; entre as estratégias com
dado local, o desempenho não decide, e a escolha cabe à privacidade.

## Resultados principais (AUROC média das 15 unidades)

| estratégia | AUROC | registros da origem cruzam a fronteira? |
|---|---|---|
| Agrupada | 0,7529 | sim, uma vez, no treinamento |
| Continuação de boosting | 0,7413 | não (só parâmetros) |
| Local | 0,7400 | não |
| Externa + Platt | 0,7156 | só a origem |

Continuação e local são **praticamente equivalentes** (P = 0,923 com
δ = 0,01). A superioridade da agrupada sobre o local não se estabelece
(P = 0,671). Em curva de dose-resposta, 25% dos registros da origem entregam
62% do ganho: informação que satura, não volume. No mecanismo de contexto, o
contexto misto empata com a agrupada (+0,0006), ao custo de exigir registros da
origem **a cada predição**; o ajuste de pesos na origem (transferência por
priori) não supera o fundacional sem ajuste.

## O achado mais reutilizável

Com hiperparâmetros fixos, o modelo local da menor unidade rendeu **AUROC de
0,500 exato**: trezentas árvores sem uma única divisão. O XGBoost estima
`base_score` pela taxa base (0,42%), o hessiano por linha vale p(1−p) ≈ 0,0042,
e `min_child_weight = 20` passa a exigir ~4.700 linhas por folha num treino de
9.199. Nenhuma divisão satisfaz, o modelo vira constante, e a comparação inteira
enviesa contra o braço com menos dados, fabricando uma "lei" de que a
transferência ajuda mais onde há menos dado (ρ = −0,743, p = 0,002).

Com seleção por unidade **e por estratégia**, a magnitude não sobrevive: a
correlação varia de −0,614 (LightGBM, p = 0,015) a +0,061 (floresta aleatória)
conforme o algoritmo. E o colapso pode ser silencioso: no LightGBM, o modelo de
zero árvores lança exceção ao consultar a importância de variáveis, em vez de
reportar zero.

**Recomendações para qualquer comparação entre estratégias de transferência:**
ajuste hiperparâmetros por unidade e por estratégia; verifique colapso antes de
interpretar AUROC ≈ 0,5; e trate exceção de software como possível colapso.

## Conteúdo

| arquivo | descrição |
|---|---|
| `Quando_transferir_modelo_clinico.ipynb` | pipeline documentado: validação do desfecho, partições sem vazamento, estratégias, métricas, correção de priori |

O notebook é documentação executável: as funções são as usadas no estudo, mas
não rodam sem um arquivo de dados no formato descrito.

```python
# requer: pandas, numpy, scikit-learn, xgboost, optuna
jupyter notebook Quando_transferir_modelo_clinico.ipynb
```

## Licença

Código sob licença MIT (ver `LICENSE`). Dados não licenciados porque não
distribuídos.

## Citação

Ao usar este material, cite o trabalho associado.
