# Predição de HbA1c Alterada com Machine Learning a partir dos Dados da PNS 2013

Projeto desenvolvido como trabalho final da disciplina de Aprendizado de Máquina do curso de Tecnologia em Análise e Desenvolvimento de Sistemas (TADS).

## Integrantes

- Gabriela Tucunduva
- Luisa de Matos
- Maria Luisa Latta

## Sobre o projeto

Este projeto investiga o uso de técnicas de aprendizado de máquina para identificar participantes com resultado de hemoglobina glicada (HbA1c) igual ou superior a 6,5%, utilizando dados da Pesquisa Nacional de Saúde (PNS) de 2013.

O objetivo é explorar a relação entre características sociodemográficas, condições de saúde e hábitos de vida com a alteração da HbA1c, avaliando o potencial de modelos preditivos como ferramenta de apoio à identificação de pessoas que podem necessitar de avaliação clínica adicional.

**Importante:** um resultado de HbA1c igual ou superior a 6,5% é um critério laboratorial relevante, mas, isoladamente, não confirma um diagnóstico clínico de diabetes. Os resultados deste projeto têm finalidade acadêmica e exploratória.

## Ficha da proposta

| Campo | Preenchimento |
|---|---|
| **Problema** | Dado o perfil (idade, sexo, peso, altura, hipertensão e hábitos de alimentação e exercício) de um adulto sem diagnóstico prévio de diabetes, prever se a hemoglobina glicada (HbA1c) está elevada (≥ 6,5%), para que a pessoa possa ser orientada a procurar avaliação e confirmação por um profissional de saúde. |
| **Usuário** | Um adulto que preenche o formulário. Com a resposta, decide se procura avaliação e exame de confirmação. |
| **Tipo e alvo** | Classificação binária. Alvo: `hba1c_alterada`. |
| **Entradas** | 10 campos (ver "Variáveis utilizadas"), todos informáveis sem fazer exame. |
| **Dataset** | PNS 2013, exames laboratoriais: 8.952 linhas × 509 colunas. |
| **Métrica e baseline** | Recall da classe positiva. O baseline Dummy ("sempre não") tem recall 0. |
| **Riscos** | Rótulo de rastreamento (não é diagnóstico), classe positiva rara (3,8%), peso e altura com ausentes, sexo e hipertensão como dados pessoais e de saúde. |

## Fonte dos dados

Os dados utilizados são provenientes da Pesquisa Nacional de Saúde (PNS) de 2013, incluindo a base de exames laboratoriais e seu dicionário de variáveis.

Arquivos na pasta `data/`:

- `exames-PNS-2013-final.xlsx`
- `dicionario-variaveis-exames-pns-2013.xlsx`

- **Fonte:** PNS 2013, base dos exames laboratoriais, realizada pelo IBGE em parceria com o Ministério da Saúde e a Fiocruz.
- **Link (microdados):** https://www.ibge.gov.br/estatisticas/sociais/saude/9160-pesquisa-nacional-de-saude.html?=&t=microdados
- **Nota técnica dos exames:** https://ftp.ibge.gov.br/PNS/2013/Divulgacoes/Outros/Exames/Nota_tecnica_Exames_Laboratoriais_PNS.pdf
- **Licença:** não identificamos uma licença formal (como Creative Commons) para esta base. O artigo metodológico dos exames (Szwarcwald et al., *Rev Bras Epidemiol* 2019; 22 Supl 2: e190004) informa que ela foi disponibilizada sem necessidade de autorização prévia para uso. O uso neste projeto é acadêmico, sempre citando a fonte.

A base original utilizada possui 8.952 registros e 509 variáveis.

## Definição da variável-alvo

A variável-alvo `hba1c_alterada` foi construída a partir da variável `Z034`, referente ao resultado de hemoglobina glicada:

- `1`: HbA1c maior ou igual a 6,5%;
- `0`: HbA1c inferior a 6,5%.

Para a definição da população analítica, foram considerados participantes que não relataram diagnóstico prévio de diabetes (`Q030 == 3`) e foram excluídas as participantes que estavam grávidas no momento da pesquisa (`P005 != 1`). Após esses critérios, a população analisada ficou com 6.832 participantes.

A distribuição da variável-alvo foi:

| Classe | Quantidade | Proporção |
|---|---:|---:|
| HbA1c abaixo de 6,5% | 6.573 | 96,21% |
| HbA1c igual ou acima de 6,5% | 259 | 3,79% |

A distribuição evidencia um forte desbalanceamento entre as classes, aspecto considerado na avaliação dos modelos.

## Métrica de avaliação

O projeto tem finalidade de rastreamento: deixar de sinalizar alguém com HbA1c elevada (falso negativo) é o erro mais caro. Por isso, a métrica principal, definida na Entrega 1, é o **recall da classe positiva**. Também são acompanhados precisão, F1 da classe positiva, F1-macro e acurácia balanceada. A acurácia é apenas informação complementar, pois chega a 96% mesmo quando o modelo não identifica nenhum caso positivo.

## Variáveis utilizadas

Foram selecionadas variáveis relacionadas a características demográficas, saúde e hábitos de vida:

| Código | Descrição |
|---|---|
| `Z002` | Idade |
| `Z001` | Sexo |
| `P00101` | Peso |
| `P00401` | Altura |
| `Q002` | Diagnóstico de hipertensão arterial |
| `P006` | Frequência semanal de consumo de feijão |
| `P018` | Frequência semanal de consumo de frutas |
| `P020` | Frequência semanal de consumo de refrigerante ou suco artificial |
| `P025` | Frequência semanal de consumo de doces |
| `P034` | Prática de atividade física |

## Esboço do fluxo da aplicação

Formulário → API → resposta. A interface nunca carrega o modelo: ela só conversa com a API por HTTP.

1. **Formulário:** o usuário informa idade, sexo, peso, altura, hipertensão, dias por semana de quatro alimentos e se praticou exercício.
2. **API:** recebe os dados por `POST /predict`, valida tipos e faixas, aplica o pipeline salvo (`pipeline.joblib`) e responde em JSON. Peso e altura podem vir em branco, pois o pré-processamento imputa esses valores.
3. **Resposta na tela:** um sinal de rastreamento, com o aviso de que não é diagnóstico e de que é preciso procurar avaliação profissional quando houver alerta.

Exemplo ilustrativo, com valores inventados só para mostrar o formato:

```json
POST /predict
{"idade": 45, "sexo": "feminino", "peso_kg": 78, "altura_cm": 165, "hipertensao": "nao",
 "dias_feijao": 7, "dias_frutas": 3, "dias_refrigerante": 1, "dias_doces": 2, "exercicio": "sim"}

200 OK
{"previsao": "sinal de alerta", "probabilidade": 0.64, "modelo": "pipeline-v1",
 "aviso": "apoio à triagem, não é diagnóstico"}
```

## Organização do projeto

```text
 diabetes_risk_prediction/
 ├── data/
 │   ├── exames-PNS-2013-final.xlsx
 │   └── dicionario-variaveis-exames-pns-2013.xlsx
 ├── notebooks/
 │   ├── 00_primeiro_contato.ipynb
 │   └── 01_eda_entrega2.ipynb
 ├── requirements.txt
 └── README.md
```

## Etapas do projeto

### Entrega 1 — Primeiro contato com os dados

- Carregamento e inspeção inicial da base;
- Conhecimento da estrutura dos dados e do dicionário de variáveis;
- Identificação inicial das variáveis relevantes para o problema.

### Entrega 2 — Análise exploratória e preparação dos dados

- Aplicação dos critérios de seleção da população analítica;
- Construção da variável-alvo `hba1c_alterada`;
- Análise exploratória e avaliação da distribuição das classes;
- Separação dos dados em conjuntos de treino e teste, com estratificação;
- Preparação das variáveis numéricas e categóricas por meio de pipelines;
- Tratamento de valores ausentes;
- Padronização das variáveis numéricas e codificação das categóricas;
- Avaliação inicial utilizando DummyClassifier e Regressão Logística.

A separação entre treino e teste foi realizada na proporção de 80% e 20%, respectivamente, com estratificação e `random_state=42`. O conjunto de treino contém 5.465 registros e o conjunto de teste contém 1.367 registros.

### Principais achados da EDA

A EDA usa apenas o conjunto de treino (`X_treino` e `y_treino`); o teste permanece reservado. Cada gráfico do notebook termina com uma observação e uma decisão.

- **Alvo:** 5.258 participantes (96,2%) com HbA1c abaixo de 6,5% e 207 (3,8%) com HbA1c igual ou acima, no treino. Decisão: manter a divisão estratificada e usar o recall da classe positiva como métrica principal.
- **Ausentes:** altura com 26,9%, peso com 14,5% e hipertensão com 1,6%. As demais variáveis não têm ausentes no treino. Decisão: mediana com indicador de ausência para peso e altura, moda para hipertensão e nenhuma linha removida.
- **Distribuições:** não há valores impossíveis nem códigos como 99 ou 999 nas variáveis escolhidas. Idade (18 a 104 anos), peso (32 a 211 kg) e altura (120 a 202 cm) têm valores altos plausíveis, que foram mantidos.
- **Relação com o alvo:** quem tem HbA1c alterada é mais velho (mediana de 53 anos contra 45) e mais pesado (74,5 kg contra 68). A taxa de HbA1c alterada é de 5,8% entre quem tem hipertensão diagnosticada, contra 3,2% entre quem não tem, e de 2,4% entre quem pratica exercício, contra 4,4% entre quem não pratica. As associações não indicam causa.
- **Redundância:** a maior correlação é entre peso e altura (r = 0,50). Nenhum par passa de |r| = 0,9 e nenhuma coluna foi removida.

### Tabela de decisões

| Coluna | O que é | Tipo | Achado na EDA | Decisão | Onde |
|---|---|---|---|---|---|
| `Z002` | idade | numérica | 18 a 104 anos, sem ausentes | mediana + `StandardScaler` | pipeline |
| `P00101` | peso (kg) | numérica | 14,5% ausente; cauda à direita (até 211 kg), valores plausíveis | mediana + indicador + `StandardScaler` | pipeline |
| `P00401` | altura (cm) | numérica | 26,9% ausente; 120 a 202 cm, plausíveis | mediana + indicador + `StandardScaler` | pipeline |
| `P006`, `P018`, `P020`, `P025` | dias por semana (feijão, frutas, refrigerante, doces) | numérica (0 a 7) | sem ausentes, sem códigos especiais | mediana + `StandardScaler` | pipeline |
| `Z001` | sexo | categórica | 2 categorias, sem ausentes | moda + one-hot (`ignore`) | pipeline |
| `Q002` | hipertensão diagnosticada | categórica | 3 categorias; 1,6% ausente; "só na gravidez" com 93 pessoas | moda + one-hot (`ignore`) | pipeline |
| `P034` | exercício nos últimos 3 meses | categórica | 2 categorias, sem ausentes | moda + one-hot (`ignore`) | pipeline |
| `Q030` | diagnóstico prévio de diabetes | filtro | só quem respondeu "Não" (3) entra | filtro; não vai para X | `limpar(df)` |
| `P005` | gravidez atual | filtro | quem respondeu "Sim" (1) sai | filtro; não vai para X | `limpar(df)` |
| `Z034` | HbA1c (%) | define o alvo | vaza o alvo (é o próprio alvo) | vira `hba1c_alterada`; fora de X | `limpar(df)` + seleção de X |
| demais 496 colunas | outros módulos da PNS | não selecionadas | fora do escopo desta entrega | ficam de fora | seleção de X |

### Avaliação inicial

A avaliação usa validação cruzada estratificada com cinco divisões (`shuffle=True`, `random_state=42`), no conjunto de treino e com o pipeline completo, com as mesmas métricas do baseline da Entrega 1. A terceira linha (regressão logística com `class_weight="balanced"`) serve apenas para mostrar que o pipeline consegue aprender; a escolha formal de modelo e de balanceamento fica para a Entrega 3.

| Modelo | Acurácia | Recall (classe 1) | Precisão (classe 1) | F1 (classe 1) | F1-macro | Acurácia balanceada |
|---|---:|---:|---:|---:|---:|---:|
| Dummy (sempre "não") | 0,9621 | 0,0000 | 0,0000 | 0,0000 | 0,4903 | 0,5000 |
| Regressão logística | 0,9619 | 0,0000 | 0,0000 | 0,0000 | 0,4903 | 0,4999 |
| Regressão logística (`class_weight="balanced"`) | 0,6337 | 0,6479 | 0,0648 | 0,1178 | 0,4432 | 0,6405 |

O classificador Dummy e a regressão logística com configuração padrão apresentaram F1-macro de aproximadamente 0,4903 e recall da classe positiva igual a zero: com o forte desbalanceamento, a regressão logística comum aprende a responder sempre "não". Com `class_weight="balanced"`, o recall sobe para 0,65, acima do baseline na métrica principal. O custo é a precisão (0,06): a maior parte dos alertas é falso positivo, o que derruba o F1-macro para 0,44. Equilibrar recall e falsos positivos é o principal desafio da próxima etapa.

## Pré-processamento

O pré-processamento foi organizado com `ColumnTransformer` e pipelines:

- **Variáveis numéricas:** imputação de valores ausentes pela mediana, criação de indicadores de ausência e padronização com `StandardScaler`.
- **Variáveis categóricas:** imputação pela categoria mais frequente e transformação com `OneHotEncoder`, utilizando `handle_unknown="ignore"`.

O pré-processamento é ajustado dentro do pipeline para evitar vazamento de dados entre os conjuntos de treino e validação.

Teste de sanidade: o pré-processador transforma as 5.465 linhas do treino em 16 colunas (7 numéricas, 2 indicadores de ausência e 7 colunas de one-hot), sem nenhum valor ausente. Colunas fora das listas ficam de fora (`remainder="drop"`).

## Tecnologias e bibliotecas

- Python
- Jupyter Notebook
- pandas
- NumPy
- Matplotlib
- Seaborn
- scikit-learn

## Como executar

1. Clone este repositório.
2. Os notebooks leem os dois arquivos da PNS 2013 (pasta `data/`) direto do GitHub, pela URL Raw, então precisam de internet. Para usar os arquivos locais, troque o caminho na célula de carregamento.
3. Abra os notebooks na pasta `notebooks/` em um ambiente com Python e Jupyter instalados.
4. Execute as células em ordem, começando pelo notebook `00_primeiro_contato.ipynb` e, em seguida, `01_eda_entrega2.ipynb`.

Instale as bibliotecas necessárias, caso ainda não estejam disponíveis no ambiente:

```bash
pip install -r requirements.txt
```

## Limitações e próximos passos

- A base utilizada é da PNS de 2013 e, portanto, não representa necessariamente o cenário epidemiológico atual.
- A variável-alvo é uma classificação baseada no resultado laboratorial de HbA1c e não equivale a um diagnóstico médico confirmado.
- A proporção reduzida de exemplos positivos pode prejudicar o desempenho dos modelos na identificação dessa classe.
- Nas próximas etapas, serão investigadas estratégias para lidar com o desbalanceamento, além de outros modelos e métricas de avaliação, com atenção especial à capacidade de identificar a classe positiva.
- Sexo e hipertensão são dados pessoais e de saúde. O formulário não pede nome nem identificação, o sexo fica como entrada demográfica básica (seu efeito será reavaliado na Entrega 3) e a resposta sempre traz o aviso de uso responsável.
- Peso e altura são autorreferidos e têm ausentes (14,5% e 26,9% no treino). O peso amostral (`peso_lab`) ainda não foi utilizado e precisa ser revisado.
- O treino tem apenas 207 casos positivos, então as diferenças observadas entre as classes devem ser lidas com cautela.

---

Projeto acadêmico desenvolvido para fins de estudo e experimentação em aprendizado de máquina.
