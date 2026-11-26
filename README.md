# Dashboard Dengue × Clima — Petrolina-PE (v11.4)

**Análise exploratória de dados (AED) da relação entre variáveis climáticas e a incidência de dengue em Petrolina-PE (2010–2025), com dashboard interativo e materiais didáticos.**
Projeto de Iniciação Científica Júnior (PIBIC Jr) — IFSertãoPE, Campus Petrolina · Curso Técnico em Informática.

---

## Sobre o projeto

Este repositório reúne o código, os dados e a documentação de um estudo de caso de **ciência de dados em saúde** conduzido com estudantes do ensino médio integrado. Sobre **809 observações semanais** (2010–2025) integradas de três fontes públicas, aplicam-se **dez técnicas de análise exploratória de séries temporais** para caracterizar a relação, defasada e macroclimática, entre clima e dengue no semiárido pernambucano. Todo o fluxo é operacionalizado em um **dashboard interativo** (Streamlit) usado como instrumento de exploração e comunicação, e é acompanhado de uma **cartilha didática** que traduz o ferramental estatístico para o nível do ensino médio.

### Principais achados (resumo honesto)

- **Uma onda por ano, com pico no outono:** o ciclo anual multiplica a incidência por ≈ **6×** entre o vale (nov–dez) e o pico (**abr–mai**, SE 16–17) — de ~5 para ~40–50 casos/semana. O pico de outono é a chuva de jan–abr defasada de 6–8 semanas: confirmação independente do lag.
- **A sazonalidade é o maior componente *sistemático*, não o maior da variância:** ela responde por 24% da variância dos casos e a tendência por ~2%; os outros **74% ficam no resíduo** (magnitude e momento dos surtos). É por isso que o clima sozinho não prevê dengue.
- **Chuva e umidade são os sinais robustos:** precedem os casos (Granger, lags ≈ 6 e 4 semanas) e **permanecem significativas mesmo após a dessazonalização** das séries. A **umidade é a mais forte das duas** em todos os testes com a defasagem correta.
- **CCF e Granger concordam quando a sazonalidade sai:** na série bruta a CCF é um *platô* (argmax indeterminado, IC 80% = [4; 12] para a chuva); sobre resíduos STL os picos vão para **lag 6** (chuva) e **lag 4** (umidade) — exatamente os lags do Granger.
- **Pressão e vento são sinais fracos/sazonais:** significativos apenas na defasagem de 1 semana e **colapsam** quando se remove a sazonalidade — não devem ser tratados como preditores.
- **Temperatura não é informativa localmente** (baixa amplitude térmica em Petrolina).
- **A relação clima–dengue é real, porém fraca e instável** (rolling Spearman com medianas < 0,3; ρ de pico cai para ≈0,08 após pré-branqueamento): o clima **isolado não basta** para previsão de curto prazo.
- **ENSO — sem sustentação estatística nesta janela:** descritivamente a ordem é **El Niño > La Niña > Neutro**, e o Kruskal-Wallis nominal dá p<0,001 — mas esse p é **inválido**: as 809 semanas se agrupam em apenas **24 episódios contíguos**. Sob três nulos válidos o **p empírico fica entre 0,14 e 0,58**; com o episódio como unidade, p = 0,079. O "paradoxo do El Niño" segue registrado como **hipótese a testar**, não como achado.
- **Contribuição metodológica:** rodamos o teste padrão, obtivemos p<0,001 e então demonstramos, construindo a distribuição nula à mão, que o teste padrão estava **errado para estes dados**. O valor crítico correto do KW aqui é ~36–120, não 5,99.

> Os detalhes metodológicos, as fórmulas e a interpretação de cada teste estão na **cartilha** (`docs/`).

---

## Estrutura do repositório

```
Dengue_dashboard/
├── app.py                      # Ponto de entrada do dashboard (Streamlit)
├── config.py                   # Configurações centrais (caminhos, cores, ONI, parâmetros)
├── gerar_figuras_connepi.py    # Script autônomo: gera as figuras do artigo (300 dpi)
├── requirements.txt
├── CHANGELOG.md                # Histórico de versões (ver v11)
│
├── content/                    # [v11] Fonte única de textos interpretativos
│   └── achados.py              # Achados/Como ler/Cuidado destilados (manuscrito+cartilha)
│
├── data/
│   └── dataset_semanal_petrolina.xlsx   # 809 registros semanais (2010–2025)
│
├── utils/                      # Motor analítico
│   ├── data_loader.py          # Carga e filtragem dos dados
│   ├── eda_avancada.py         # ★ As 10 técnicas estatísticas (coração do projeto)
│   ├── metricas.py             # Indicadores-resumo (KPIs)
│   └── export.py               # Exportação (CSV/Excel/relatórios)
│
├── components/
│   ├── layout.py               # CSS, cabeçalho e barra lateral (filtros)
│   └── charts.py               # Construção dos gráficos (Plotly)
│
├── pages/                      # As 5 abas do dashboard
│   ├── situacao_atual.py
│   ├── panorama_historico.py
│   ├── clima_dengue.py
│   ├── enso_dengue.py
│   ├── analise_estatistica.py
│   └── educacional_exportacao.py
│
└── docs/
    ├── guia_interpretacao_clima_dengue_petrolina.md   # Cartilha didática
    ├── roteiro_scripts_trilha_aprendizagem.md         # Trilha de estudo dos scripts
    ├── PLANO_DASHBOARD_v11.3.md                       # Revisão crítica e plano da v11.3
    ├── PLANO_CORRECAO_v11.2.md                        # Correções científicas da v11.2
    └── EXECUCAO_lotes_C_D.md                          # O que a v11.3 entregou e o que falta
```

O arquivo **`utils/eda_avancada.py` é o motor**: tanto o dashboard (via `pages/`) quanto o script de figuras chamam a mesma lógica de cálculo. Entender esse arquivo é entender o projeto inteiro. Ele **nunca é alterado**: as verificações de robustez acrescentadas depois da validação vivem em **`utils/robustez.py`**, que é aditivo — um padrão deliberado de governança do código científico.

---

## Instalação

Requer **Python 3.11+**.

```bash
# 1. Clone o repositório
git clone <URL-DO-REPOSITORIO>
cd Dengue_dashboard

# 2. (Recomendado) crie um ambiente virtual
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Instale as dependências
pip install -r requirements.txt
```

---

## Como usar

**Rodar o dashboard interativo:**
```bash
streamlit run app.py
```
Abre no navegador, com cinco abas (Situação Atual, Panorama Histórico, Clima × Dengue, ENSO e Dengue, Análise Estatística) e filtros por período, variável e fase do ENSO.

**Gerar todas as figuras analíticas do artigo (300 dpi):**
```bash
python gerar_figuras_connepi.py
```
As figuras são salvas em `figuras_out/`.

**Gerar apenas uma figura (ex.: a de número 3):**
```bash
python gerar_figuras_connepi.py 3
```

---

## Dados e fontes

O arquivo `data/dataset_semanal_petrolina.xlsx` (809 semanas, 03/01/2010 – 06/07/2025) integra:

- **InfoDengue/SINAN** (Sistema de Informação de Agravos de Notificação) — `casos_est`, níveis de alerta, probabilidade de Rt > 1.
  > ⚠️ **Precisão terminológica:** `casos_est` é o *nowcasting* do InfoDengue, que corrige o **atraso de notificação** — **não** a subnotificação. Em semanas já consolidadas (praticamente toda a janela) ele coincide com o notificado: neste arquivo, `casos_est == casos` nas 809 linhas, e `casos_est_min == casos_est_max`. Casos que nunca chegaram ao SINAN permanecem invisíveis.
  > ⚠️ **Proveniência:** o arquivo tem 809 linhas, mas a grade semanal completa do período tem **810 semanas** — falta **08/04/2018**, interpolada no pipeline (`.asfreq("W-SUN").interpolate("time")`).
- **INMET** (Instituto Nacional de Meteorologia) — temperatura, umidade, precipitação, pressão, vento, nebulosidade, insolação, agregados por semana.
- **NOAA/CPC** (National Oceanic and Atmospheric Administration / Climate Prediction Center) — Índice Oceânico Niño (ONI), para classificação das fases do ENSO.

Por se tratar de dados secundários, públicos e agregados, o estudo é isento de apreciação por Comitê de Ética (Resolução CNS 510/2016, art. 1º, III).

---

## Documentação

- **`docs/guia_interpretacao_clima_dengue_petrolina.md`** — cartilha didática: cada técnica no formato Intuição → Formalização (equação com cada termo explicado) → No pipeline → Resultado → Como ler → Cuidado, além de fundamentos de testes de hipóteses, leitura das figuras e exercícios com gabarito.
- **`docs/roteiro_scripts_trilha_aprendizagem.md`** — trilha de estudo dos scripts: por onde começar, o que cada arquivo faz e como ligar cada função ao método estatístico e à figura correspondente.
- **`docs/PLANO_DASHBOARD_v11.3.md`** — a revisão crítica que originou esta versão: 13 observações do coordenador, três bugs não notados e o plano de correção em quatro lotes.
- **`docs/EXECUCAO_lotes_C_D.md`** — o que os Lotes C e D entregaram, a tabela de verificação de cada número contra o dataset real e os reflexos ainda pendentes no manuscrito.

Sugestão de percurso para quem chega ao projeto: **README → roteiro de scripts → cartilha (módulo a módulo, com o código ao lado) → reproduzir as figuras → explorar o dashboard.**

---

## Reprodutibilidade

- Todas as sementes de aleatoriedade estão fixadas (`seed = 42`); resultados de bootstrap e informação mútua são idênticos a cada execução.
- Todos os parâmetros de análise (número de lags, janela do rolling, número de permutações, período sazonal) estão centralizados em `config.py`.
- Os dados do ONI (fases ENSO) estão versionados em `config.py` (`ONI_DATA`), com a data de consulta ao NOAA/CPC documentada no próprio arquivo.

---

## Nota metodológica

As análises são **exploratórias**: a causalidade de Granger mede precedência preditiva, não causa. Como clima e casos são séries fortemente sazonais, conduziu-se uma verificação de robustez por dessazonalização (resíduos da STL), que preservou apenas chuva e umidade como sinais robustos. Para trabalhos futuros, o estado da arte aponta modelos aditivos com defasagens distribuídas (DLNM/GAM) e modelos híbridos que combinam clima, histórico de casos e ENSO, com validação prospectiva.

---

## Equipe

Projeto PIBIC Jr — IFSertãoPE, Campus Petrolina (Curso Técnico em Informática). Estudantes bolsistas, professores colaboradores e orientação docente. 
Orientador: Ednaldo Gomes da Silva (Campus Petrolina)
Bolsista: Carlos Eduardo Lopes Felix
Voluntaria: Maria Fernanda Silva Santos



