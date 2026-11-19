# \# Previsão de vendas da drogaria Rossmann

# 

# \[!\[Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/joao1824/projeto-ml-rossmann/blob/main/Projeto\_ML\_Rossmann.ipynb)

# 

# Projeto 2 da disciplina de \*\*Machine Learning Clássico\*\* do curso de Engenharia de Computação da \*\*UNISATC\*\*.

# 

# \*\*Pergunta:\*\* quais fatores influenciam a quantidade de vendas da drogaria Rossmann?

# 

# \*\*Objetivo:\*\* prever a venda diária de cada loja e identificar o que mais pesa nessa venda.

# 

# | | |

# |---|---|

# | \*\*Equipe\*\* | Bruno Pagani Rampinelli, João Henrique Camilo Fogaça e Vinícius dos Santos Nascimento |

# | \*\*Professor\*\* | Rodrigo Ramos Silva |

# | \*\*Dados\*\* | \[Rossmann Store Sales (Kaggle, 2015)](https://www.kaggle.com/competitions/rossmann-store-sales) |

# | \*\*Tipo de problema\*\* | Regressão (supervisionado) + agrupamento com K-means (não supervisionado) |

# 

# \---

# 

# \## Resultado em uma frase

# 

# O \*\*Gradient Boosting\*\* prevê a venda diária de cada loja com erro médio de \*\*688 por dia\*\* e \*\*R² de 0,89\*\* num período que ele nunca viu (janeiro a julho de 2015), \*\*12,4% melhor\*\* que a média histórica de cada loja.

# 

# \---

# 

# \## Arquivos do repositório

# 

# | Arquivo | O que é |

# |---|---|

# | `Projeto\_ML\_Rossmann.ipynb` | Notebook completo, executado do início ao fim, com as explicações em células de texto |

# | `train.csv` | Vendas diárias por loja (1.017.209 linhas, 2013 a 2015), arquivo original do Kaggle |

# | `store.csv` | Cadastro das 1.115 lojas, arquivo original do Kaggle |

# | `Relatorio\_Rossmann.docx` | Relatório no modelo ABNT, com os mesmos números do notebook |

# 

# \---

# 

# \## Como reproduzir

# 

# 1\. Abra o notebook no Google Colab (botão no topo desta página).

# 2\. Faça o upload de `train.csv` e `store.csv` na pasta do Colab (`/content`).

# 3\. Execute todas as células (\*\*Ambiente de execução → Executar tudo\*\*).

# 4\. Se a primeira célula pedir, reinicie a sessão e execute tudo de novo. Ela instala as versões das bibliotecas usadas no projeto.

# 

# A execução completa leva de \*\*25 a 30 minutos\*\*, quase todo esse tempo na busca de hiperparâmetros.

# 

# \*\*Versões fixadas na primeira célula:\*\* pandas 3.0.5, scikit-learn 1.9.1, matplotlib 3.11.2 e numpy 2.5.3. Toda a aleatoriedade usa a mesma semente, para os números se repetirem.

# 

# \---

# 

# \## Metodologia

# 

# \### Dados e higiene

# \- `train.csv` e `store.csv` unidos pelo código da loja.

# \- 172.817 dias com a loja fechada foram removidos (venda zero por definição).

# \- \*\*`Customers` ficou fora do modelo:\*\* o número de clientes só é conhecido depois que o dia termina, então usá-lo seria vazamento de dados.

# 

# \### Divisão por data

# | Conjunto | Período | Linhas | Uso |

# |---|---|---|---|

# | Treino | 2013 e 2014 | 648.309 | Aprender e escolher (validação cruzada) |

# | Teste | Jan a jul de 2015 | 196.029 | Medir o desempenho final, \*\*uma única vez\*\* |

# 

# A validação cruzada é \*\*temporal\*\*: 5 dobras de 6 semanas no fim do treino. Em cada uma, o modelo aprende com o passado e é validado no futuro.

# 

# \### Pré-processamento

# Tudo que aprende com os dados fica \*\*dentro de um pipeline\*\*, ajustado só no treino de cada dobra:

# \- mediana do treino nos valores ausentes, com indicador de ausência;

# \- One-Hot Encoding nas categorias;

# \- padronização só no SVR e no K-means;

# \- histórico de vendas de cada loja (por dia da semana e com e sem promoção);

# \- alvo em \*\*logaritmo\*\*, porque a venda tem cauda longa. A previsão volta para a escala original antes de calcular o erro.

# 

# Atributos criados: ano, mês, dia, semana do ano, meses de concorrência, Promo2 ativa, véspera e dia seguinte a feriado.

# 

# \### Métricas

# Definidas \*\*antes\*\* de treinar os modelos.

# \- \*\*MAE\*\* (critério de escolha): erro médio em unidades de venda.

# \- \*\*RMSE\*\* e \*\*R²\*\*: confirmação.

# \- \*\*RMSPE\*\* (métrica da competição): secundária, porque um único dia com venda muito baixa distorce o resultado.

# 

# \*\*Baseline:\*\* média histórica de cada loja por dia da semana e com ou sem promoção.

# 

# \### Modelos

# | Modelo | Pressuposto |

# |---|---|

# | Árvore de Decisão | A venda muda em degraus, conforme combinações de fatores |

# | Random Forest (bagging) | A média de muitas árvores independentes cancela os erros de cada uma |

# | Gradient Boosting (boosting) | Árvores em sequência, cada uma corrigindo o erro da anterior |

# | SVR (kernel RBF) | A venda varia de forma suave |

# | K-means | As lojas formam perfis de comportamento |

# 

# Hiperparâmetros escolhidos com `GridSearchCV` sobre as 5 dobras temporais, pelo menor MAE de validação.

# 

# \---

# 

# \## Resultados

# 

# \### Validação cruzada (média das 5 dobras)

# | Modelo | MAE | RMSE | R² |

# |---|---|---|---|

# | \*\*Gradient Boosting\*\* | \*\*653\*\* | \*\*958\*\* | \*\*0,91\*\* |

# | Random Forest | 693 | 1.038 | 0,89 |

# | Árvore de Decisão | 794 | 1.170 | 0,86 |

# | Baseline | 920 | 1.327 | 0,82 |

# | SVR | 938 | 1.382 | 0,80 |

# 

# \### Teste (janeiro a julho de 2015, uso único)

# | Modelo | MAE | RMSE | R² | RMSPE |

# |---|---|---|---|---|

# | \*\*Gradient Boosting\*\* | \*\*688\*\* | \*\*993\*\* | \*\*0,89\*\* | \*\*0,15\*\* |

# | Random Forest | 697 | 1.022 | 0,89 | 0,15 |

# | Árvore de Decisão | 767 | 1.108 | 0,87 | 0,16 |

# | Baseline | 785 | 1.163 | 0,85 | 0,17 |

# | SVR | 921 | 1.380 | 0,79 | 0,21 |

# 

# A ordem dos modelos no teste é a mesma da validação, o que mostra que a validação cruzada temporal estimou bem o desempenho no futuro.

# 

# \### Fatores que mais influenciam a venda

# 1\. \*\*Qual é a loja\*\* (porte e histórico)

# 2\. \*\*Promoção do dia\*\*: +39% em média

# 3\. \*\*Dia da semana\*\*

# 4\. \*\*Época do ano\*\*: dezembro vende 27% a mais

# 5\. \*\*Proximidade de feriados\*\*

# 

# Sem promoção, feriado e dia da semana, o R² da validação cai de \*\*0,91 para 0,57\*\*.

# 

# \### Perfis de loja (K-means, K = 3)

# | Perfil | Lojas | Característica |

# |---|---|---|

# | A · Dependentes de promoção | 399 | A promoção aumenta a venda em 56%; sábado fraco |

# | B · Movimentadas de ticket baixo | 326 | Mais clientes por dia, menor gasto por cliente |

# | C · Ticket alto e pouco movimento | 390 | Poucos clientes, maior gasto por cliente; sábado forte |

# 

# Os perfis são estáveis (ARI de 0,82 a 0,98) e úteis: o perfil A formado só com dados de 2013 antecipou a reação à promoção em 2014.

# 

# \---

# 

# \## Limitações

# \- O teste vai de janeiro a julho e não inclui dezembro, o mês mais difícil de prever.

# \- O SVR foi treinado com uma amostra de 6 mil linhas, por custo, e as grades de hiperparâmetros foram pequenas.

# \- Os perfis do K-means têm fronteiras difusas (silhueta de 0,29). As ações sugeridas para cada perfil são hipóteses.

# \- A importância das variáveis mostra associação, não causa.

# 

# \---

# 

# \## Referências

# \- ALBASSAM, S.; ALQAHTANI, A.; ALAZBA, A. Retail sales forecasting using tree-based machine learning models: an empirical study of preprocessing configurations and feature ablation. \*Applied Sciences\*, v. 16, n. 15, 7556, 2026.

# \- BOJER, C. S.; MELDGAARD, J. P. Kaggle forecasting competitions: an overlooked learning opportunity. \*International Journal of Forecasting\*, v. 37, n. 2, p. 587-603, 2021.

# \- GUO, C.; BERKHAHN, F. Entity embeddings of categorical variables. arXiv:1604.06737, 2016.

# \- KAGGLE. Rossmann Store Sales. 2015. Disponível em: https://www.kaggle.com/competitions/rossmann-store-sales

# \- MOHD SHAHAR, N. A. et al. Sales forecasting using machine learning methods for online store. In: InvENT 2024. Atlantis Press, 2024.

# \- PAVLYSHENKO, B. M. Machine-learning models for sales time series forecasting. \*Data\*, v. 4, n. 1, 15, 2019.

# \- PEDREGOSA, F. et al. Scikit-learn: machine learning in Python. \*Journal of Machine Learning Research\*, v. 12, p. 2825-2830, 2011.

# 

# A lista completa, no formato ABNT, está no relatório.

