# Quando transferir um modelo clínico entre hospitais

Código de apoio ao estudo sobre estratégias de transferência de modelos
preditivos entre unidades de pronto socorro, submetido ao 32º Prêmio Jovem
Cientista (CNPq, 2026).

**Autor:** Victor Hugo Ovani Marchetti
**Orientação:** Prof. Dr. Alexandre Dias Porto Chiavegatto Filho — LabDaps, Faculdade de Saúde Pública da USP
**Coorientação:** Dr. José Arnaldo Shiomi da Cruz
**Aprovação ética:** CEP do Hospital IGESP, parecer 8.426.290, CAAE 96939326.0.0000.5450

---

## Os dados não estão aqui, e não estarão

Este repositório contém **método, não dado**. Os registros são assistenciais, de
pacientes, categoria de dado pessoal sensível sob o art. 11 da Lei Geral de
Proteção de Dados. Não são distribuídos, nem em amostra, nem agregados a nível
individual.

As consultas de extração ao repositório institucional **também não são
publicadas**, por descreverem a estrutura interna de sistemas assistenciais.

O que está aqui é o pipeline **a partir de um arquivo tabular**, com o contrato
de dados descrito no notebook. Quem tiver dados próprios com esse formato
reproduz a análise inteira.

As unidades aparecem anonimizadas.

## Conteúdo

| arquivo | descrição |
|---|---|
| `Quando_transferir_modelo_clinico.ipynb` | pipeline completo: validação do desfecho, partições sem vazamento, estratégias comparadas, métricas, correção de priori |

## O achado mais reutilizável

Durante o estudo, o modelo local de uma unidade produziu AUROC de **0,500
exato** — predição constante. A leitura natural seria "com 39 eventos não há
sinal suficiente". Estava errada.

O XGBoost estima `base_score` a partir da taxa base do desfecho (0,42% nesta
coorte). O hessiano por linha vale então `p(1−p) ≈ 0,0042`, e exigir
`min_child_weight = 20` equivale a exigir **4.738 linhas por folha** — num
treino de 9.199. Nenhuma divisão satisfaz a restrição e o modelo vira constante.

Com o parâmetro ajustado, a mesma unidade chega a **0,717**.

Isso importa para qualquer comparação entre estratégias de transferência:
hiperparâmetros fixos penalizam mais o braço com menos dados, que é justamente o
braço local nas unidades pequenas. A penalização fica **correlacionada com o
porte da unidade** e produz, sozinha, uma aparente lei de que "a transferência
ajuda mais onde há menos dado".

Neste estudo, com hiperparâmetros ajustados por unidade, a correlação entre
número de eventos e ganho da transferência caiu de **ρ = −0,74 (p = 0,002)**
para **ρ = −0,11 (p = 0,69)**.

**Recomendação:** ajuste hiperparâmetros por unidade e por estratégia, e
verifique se algum modelo colapsou antes de interpretar AUROC próxima de 0,5. O
notebook traz a função de diagnóstico.

## Como usar

```python
# requer: pandas, numpy, scikit-learn, xgboost, optuna
jupyter notebook Quando_transferir_modelo_clinico.ipynb
```

O notebook é documentação executável: as funções são as usadas no estudo, mas
não rodam sem um arquivo de dados no formato descrito.

## Licença

Código sob licença MIT (ver `LICENSE`). Dados não licenciados porque não
distribuídos.

## Citação

Ao usar este material, cite o trabalho associado.
