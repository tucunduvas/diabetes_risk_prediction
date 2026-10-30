# Predição de diabetes com dados do Vigitel

Projeto Final de Machine Learning · IFMS Três Lagoas · Grupo: _Gabriela Tucunduva, Luísa Matos e Maria Luisa Latta_

## Ficha da proposta (Entrega 1)

| Campo | Preenchimento |
|---|---|
| **Problema** | Dado o perfil demográfico e alguns hábitos/condições de saúde informados pela pessoa, prever se ela tem diagnóstico referido de diabetes para que um profissional de saúde (ou a própria pessoa) busque triagem e acompanhamento cedo. |
| **Usuário** | _(quem preenche o formulário e o que decide com a resposta)_ |
| **Tipo e alvo** | Classificação binária · alvo: `diab` (1 = diagnóstico referido de diabetes) |
| **Entradas** | idade, sexo, anos de estudo, peso, altura, hipertensão referida, fumante, consumo abusivo de álcool (todas passam no teste do formulário; ver notebook seção 5) |
| **Dataset** | Vigitel, Ministério da Saúde · https://svs.aids.gov.br/download/Vigitel/ · arquivo 2006–2024 (486 colunas), recorte do grupo: edições de **2020 a 2024** (2020, 2021, 2023 e 2024; não existe 2022) · 102.908 linhas × 9 colunas usadas (8 entradas + alvo) ·|
| **Métrica** | Recall da classe diabetes + F1 da classe diabetes: deixar passar uma pessoa com diabetes custa mais que um alarme falso (que só leva a um exame de confirmação). |
| **Baseline** | `DummyClassifier(most_frequent)`: recall = 0, F1 = 0, acurácia = 0,868 (alta só porque a classe positiva é minoria: 13,2% dos casos). Números em `notebooks/00_primeiro_contato.ipynb`. |
| **Riscos** | Alvo autorreferido (ruído no rótulo); prevalência varia entre as edições (12–15%) e 2020–2021 são anos de pandemia; não há edição de 2022; amostra telefônica só das capitais; colunas sensíveis (sexo, escolaridade); vazamento (`db`, `insulina`, `med_db` ficam de fora: em branco = sem diabetes); uso apenas como apoio à triagem. |

## Fluxo da aplicação

Formulário → `POST /predict` (JSON) → API valida e consulta o `pipeline.joblib` → resposta com classe, probabilidade, versão do modelo e aviso.

## Como rodar

_(preencher nas próximas entregas)_
