# Análise de Teste A/B

Projeto que avalia, por diferentes abordagens, se a inclusão de um **CTA (call-to-action)** em uma página altera a **taxa de conversão (compra)** dos usuários.

São comparados dois grupos:

- **Controle**: usuários que viram a versão atual da página (`pagina_sem_cta`);
- **Tratamento**: usuários que viram a versão modificada (`pagina_com_cta`).

A análise segue dois caminhos complementares:

1. **Testes estatísticos** (paramétricos e não paramétricos), para verificar se a diferença de conversão entre os grupos é estatisticamente significativa;
2. **Regressão logística**, para quantificar o tamanho dessa diferença por meio do coeficiente (logit) e do odds-ratio.

Todo o desenvolvimento está em [notebook.ipynb](notebook.ipynb).

## Dados

O arquivo [dataset.csv](dataset.csv) possui 294.460 registros e 5 colunas:

| Coluna       | Descrição                                          |
|--------------|----------------------------------------------------|
| `id_usuario` | Identificador do usuário                           |
| `timestamp`  | Data e hora do acesso (janeiro de 2024)            |
| `grupo`      | `controle` ou `tratamento`                         |
| `pagina`     | `pagina_sem_cta` ou `pagina_com_cta`               |
| `converteu`  | `1` se o usuário realizou a compra, `0` caso contrário |

## Etapas

### 1. Análise exploratória e limpeza

- Não há valores nulos e os grupos estão balanceados (147.270 no controle e 147.190 no tratamento).
- Foi identificada uma **inconsistência**: 1.965 usuários do controle viram a página com CTA e 1.928 do tratamento viram a página sem CTA. Esses registros foram removidos, restando **290.567 observações**.
- Taxa de conversão após a limpeza:

| Grupo      | Taxa de conversão |
|------------|-------------------|
| Controle   | 11,88%            |
| Tratamento | 12,04%            |

### 2. Teste A/B com métodos estatísticos

Hipóteses:

- **H0**: não há associação entre o grupo e a conversão;
- **H1**: o grupo ao qual o usuário pertence influencia a probabilidade de conversão.

Nível de significância adotado: **α = 0,05**.

| Teste | Tipo | Estatística | p-valor | Conclusão |
|-------|------|-------------|---------|-----------|
| Qui-quadrado (independência) | Não paramétrico | χ² = 1,716 | 0,190 | Não rejeita H0 |
| T de Student (amostras independentes) | Paramétrico | t = -1,316 | 0,188 | Não rejeita H0 |
| Mann-Whitney U (bicaudal) | Não paramétrico | U ≈ 1,054 × 10¹⁰ | 0,188 | Não rejeita H0 |

- **Qui-quadrado**: testa a independência entre `grupo` e `converteu` a partir da tabela de contingência.
- **T de Student**: compara as médias de conversão dos grupos. Para validar a suposição de normalidade foi aplicado o teste de **Shapiro-Wilk**, que a rejeitou para ambos os grupos (p ≈ 10⁻¹⁷⁸) — esperado, já que a variável é binária. Por isso, o resultado do teste T não foi usado como base para decisão.
- **Mann-Whitney U**: alternativa não paramétrica ao teste T, que não exige normalidade.

Todos os testes concordam: **a diferença de conversão entre os grupos não é estatisticamente significativa**.

### 3. Teste A/B com regressão logística

Foi ajustado um modelo de regressão logística (`scikit-learn`) tendo `converteu` como variável dependente e uma variável dummy de página (`1` = com CTA, `0` = sem CTA) como independente.

| Métrica | Valor |
|---------|-------|
| Coeficiente (logit) | 0,0156 |
| Odds-ratio (e^coef) | 1,0157 |

Interpretação do odds-ratio:

- **OR = 1**: a variável não tem efeito sobre a conversão;
- **OR > 1**: efeito positivo;
- **OR < 1**: efeito negativo.

O odds-ratio de aproximadamente 1,016 indica que a presença do CTA multiplica as chances (odds) de conversão por ~1,016, ou seja, **odds cerca de 1,6% maiores** em relação à página sem CTA.

## Conclusão

A página com CTA apresenta um efeito ligeiramente positivo sobre a conversão (12,04% contra 11,88%, odds-ratio ≈ 1,016), mas essa diferença **não é estatisticamente significativa** segundo os testes Qui-quadrado e Mann-Whitney U (p ≈ 0,19). Com os dados disponíveis, não há evidência suficiente para afirmar que o CTA aumenta a conversão.

## Como executar

1. Crie e ative um ambiente virtual:

   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # Linux/macOS
   source .venv/bin/activate
   ```

2. Instale as dependências:

   ```bash
   pip install numpy pandas matplotlib seaborn scipy statsmodels scikit-learn jupyter
   ```

3. Abra o notebook:

   ```bash
   jupyter notebook notebook.ipynb
   ```

## Tecnologias

- Python
- pandas e NumPy (manipulação de dados)
- Matplotlib e Seaborn (visualização)
- SciPy (Qui-quadrado, T de Student, Shapiro-Wilk, Mann-Whitney U)
- scikit-learn (regressão logística)
