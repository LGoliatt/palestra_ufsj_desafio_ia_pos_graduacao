---
name: openalex-estado-da-arte-lacunas
description: Gera um arquivo pronto para compartilhar (notebook Jupyter .ipynb ou página HTML) que consulta a API do OpenAlex para mapear o estado da arte de um tema (publicações por ano, tópico e país) e cruzar dois eixos, por exemplo método x área de aplicação, para revelar onde há pouca literatura e, portanto, possíveis lacunas e oportunidades de contribuição. Use sempre que o usuário pedir estado da arte, mapeamento de literatura, lacunas de pesquisa, análise de tendências, cruzamento de métodos com aplicações ou justificativa de originalidade com OpenAlex.
---

# Prompt: estado da arte e lacunas de pesquisa com OpenAlex

Versão 3.0 | Funciona em qualquer assistente de IA (ChatGPT, Claude, Gemini, Copilot, Mistral, etc.)

## COMO USAR (para a pessoa)

1. Copie TODO o conteúdo deste arquivo, a partir da linha "INSTRUÇÕES PARA A IA".
2. Cole no chat da IA.
3. No bloco ENTRADAS, ajuste as duas listas: termos do eixo A (ex.: seus métodos) e do eixo B (ex.: suas áreas de aplicação).
4. Receba o `.ipynb` ou o `.html`, baixe e abra.

Para compartilhar: envie este `.md` (prompt) ou o arquivo gerado. O HTML abre com duplo clique em qualquer navegador. O notebook abre no Google Colab (Arquivo > Fazer upload), JupyterLab ou VS Code.

---

## INSTRUÇÕES PARA A IA

Atue como cientista de dados e bibliometrista, especialista em Python, JavaScript, APIs REST e visualização de dados científicos. Escreva todo texto visível ao usuário em português.

### Objetivo

Criar um arquivo real e funcional que consulta `https://api.openalex.org/works` e entrega:

1. **Panorama do estado da arte**: publicações por ano, principais tópicos e principais países.
2. **Evolução dos tópicos** (opcional): quais tópicos crescem ou encolhem ao longo dos anos.
3. **Matriz de cruzamento**: eixo A (métodos) x eixo B (aplicações), com a contagem de publicações em cada par.
4. **Mapa de lacunas**: ranking dos pares com menos literatura do que o esperado, dado o volume de cada eixo, com tendência recente.
5. **Leitura-base**: para qualquer par escolhido, os trabalhos mais citados, os mais recentes, os países e as instituições mais ativas.

Nunca baixe trabalhos um a um para contar. Use `group_by`.

### Entradas (use os padrões se não informado)

| Variável | Padrão |
|---|---|
| NOME_DO_ESTUDO | texto livre (ex.: "Métodos simbólicos e interpretáveis em ciências ambientais") |
| LISTA_METODOS (eixo A) | obrigatória. Cada item é um termo ou um grupo de sinônimos separados por " \| " |
| LISTA_APLICACOES (eixo B) | obrigatória, mesmo formato |
| ANO_INICIAL | 2018 |
| JANELA_RECENTE | 3 (anos completos usados para a tendência) |
| EXCLUIR_ANO_ATUAL | verdadeiro (o ano corrente é parcial e distorce a tendência) |
| TIPOS | vazio (todos). Opcional: "article\|review" |
| ESPERADO_MINIMO | 3 (só classifica lacuna se o esperado for pelo menos isso) |
| RAZAO_LACUNA | 0.5 (observado/esperado abaixo disso = lacuna) |
| LIMITE_POUCO_EXPLORADO | 5 (observado igual ou abaixo disso = pouco explorado) |
| EMAIL_CONTATO | "seu-email@exemplo.com" (placeholder) |
| FORMATO_SAIDA | "ipynb", "html" ou "ambos". Se não informado, pergunte em UMA linha; sem resposta, gere os dois |
| CHAVE_API | "" (opcional) |

Limite de segurança: eixo A x eixo B no máximo 10 x 10. Acima de 64 pares, avise o tempo estimado (cerca de 0,2 s por requisição) antes de executar.

### Regras da API (valem para os dois formatos)

- Base: `https://api.openalex.org/works`. Sempre enviar `mailto`=EMAIL_CONTATO. Se CHAVE_API estiver preenchida, enviar como `api_key`. Se a API responder 401, 403 ou 429, orientar a conferir as regras atuais em docs.openalex.org.
- Timeout de 30 s, tratamento de erro HTTP, pausa de 0,15 s entre requisições, repetição com espera crescente em caso de 429, mensagens de erro claras.
- Bloquear a consulta se o e-mail ainda for o placeholder.
- **Construção da expressão de busca**: cada termo entre aspas (frase exata); sinônimos unidos por `OR`; operadores em MAIÚSCULAS; grupos entre parênteses. Exemplo: `("symbolic regression" OR "genetic programming")`. Par A x B: `(<expr A>) AND (<expr B>)`.
- **Como consultar**: usar `filter=title_and_abstract.search:<expressão>,from_publication_date:<ANO_INICIAL>-01-01` (acrescentar `,type:<TIPOS>` se informado). Se a API recusar essa forma (HTTP 400), usar o parâmetro `search=<expressão>` com o mesmo filtro de data e registrar na interface que a busca ocorreu também no texto completo.
- **Contagem**: usar `group_by=publication_year`. O total é a SOMA das contagens dos grupos (não confiar em `meta.count`). Descartar o grupo "unknown". Uma única requisição por consulta fornece o total e a série anual.
- **Panorama**: `group_by=publication_year`, `group_by=primary_topic.id` (top 15 pelos dados retornados), `group_by=authorships.countries` (top 15, exibir `key_display_name`), sobre a expressão do universo (definido abaixo).
- **Evolução dos tópicos** (opcional): para os 10 principais tópicos, uma requisição por tópico com `filter=primary_topic.id:<id>,...` e `group_by=publication_year`.
- **Leitura-base** de um par: `sort=cited_by_count:desc&per-page=10` e `sort=publication_date:desc&per-page=10`, com `select=id,display_name,publication_year,cited_by_count,fwci,doi,primary_location`. Mais `group_by=authorships.countries` e `group_by=authorships.institutions.id` (top 10) para o par.
- Sem tokens, sem credenciais, sem arquivos externos.

### Método (definições obrigatórias, iguais nos dois formatos)

**Universo U.** Número de trabalhos que casam com `(<todos os termos do eixo A unidos por OR>) OR (<todos os termos do eixo B unidos por OR>)`, no período. Calculado por uma consulta.

**Marginais.** Para cada item i do eixo A: `A_i` (total) e série anual. Para cada item j do eixo B: `B_j` e série anual.

**Matriz.** Para cada par (i, j): `obs_ij` (total) e série anual, via expressão `(A_i) AND (B_j)`.

**Esperado sob independência.** `esp_ij = A_i x B_j / U`. Razão `razao_ij = obs_ij / esp_ij`. Trate o esperado como linha de base de comparação entre pares, não como verdade absoluta, porque métodos e aplicações nunca são totalmente independentes.

**Tendência.** Sobre anos completos (excluindo o ano corrente se EXCLUIR_ANO_ATUAL): `recente` = soma dos últimos JANELA_RECENTE anos, `anterior` = soma dos JANELA_RECENTE anos anteriores. `tendencia = (recente + 1) / (anterior + 1)`. Acima de 1 indica crescimento.

**Classificação de cada par** (nesta ordem de prioridade):
- **Lacuna**: `esp_ij >= ESPERADO_MINIMO` e `razao_ij < RAZAO_LACUNA`.
- **Pouco explorado**: `obs_ij <= LIMITE_POUCO_EXPLORADO` e não é lacuna.
- **Emergente**: `obs_ij` até 4x o LIMITE_POUCO_EXPLORADO e `tendencia >= 2`.
- **Consolidado**: `razao_ij >= 1.5` e `obs_ij` acima do mediano da matriz.
- **Intermediário**: demais.

**Ranking de lacunas.** Ordene as lacunas por `esp_ij - obs_ij` decrescente (quanto mais literatura "faltando" em relação ao esperado, mais alto). Em seguida, os pares emergentes por tendência.

### Visualizações (valem para os dois formatos)

1. **Publicações por ano**: barras verticais azuis, valor no topo, separador de milhares, grade suave só no eixo Y, título com NOME_DO_ESTUDO e o período.
2. **Principais tópicos** e **principais países**: barras horizontais ordenadas, com valor.
3. **Evolução dos tópicos** (se ativada): mapa de calor tópico x ano, escala sequencial azul.
4. **Matriz método x aplicação**: mapa de calor com o número em cada célula e escala de cor logarítmica. Linhas e colunas ordenadas pelos totais marginais. Células de lacuna com contorno tracejado, células emergentes com contorno sólido. Legenda clara. Não depender só de cor (use contorno e a tabela).
5. **Mapa de lacunas**: segundo mapa de calor com a `razao_ij` (divergente, centrado em 1) e a tabela ranqueada com colunas: método, aplicação, observado, esperado, razão, tendência, classe.
6. Aparência limpa e profissional, adequada para relatório científico.

### Formato A: notebook `.ipynb`

Crie o arquivo de verdade em JSON nbformat 4 (nbformat_minor 5), compatível com Python 3.9+, executável célula por célula. Dependências: requests, pandas, numpy, matplotlib, seaborn. Célula opcional de instalação: `%pip install -q requests pandas numpy matplotlib seaborn`.

Seções, cada uma com Markdown explicativo antes do código:

1. Título e descrição
2. Objetivos
3. Instalação das dependências
4. Importação das bibliotecas
5. Configuração (variáveis da tabela de entradas, aviso destacado sobre trocar o e-mail)
6. Funções com docstring: `montar_expressao`, `consultar_grupos`, `contar_por_ano`, `panorama`, `matriz_cruzamento`, `classificar_pares`, `leitura_base`
7. Verificação das entradas (tamanho da matriz, tempo estimado)
8. Panorama: ano, tópicos, países
9. Evolução dos tópicos (célula opcional)
10. Matriz método x aplicação
11. Mapa de lacunas e tabela ranqueada
12. Leitura-base para os 3 principais pares de lacuna e para um par escolhido pelo usuário
13. Exportação (PNG 300 DPI de cada gráfico e CSV da matriz e do ranking)
14. Metodologia e limitações

Se plotly estiver disponível, ofereça mapa de calor interativo em célula opcional.

Validação programática (se você executa código): arquivo existe; JSON válido; `nbformat` definido; há células Markdown e de código; contém OpenAlex, mailto, group_by, publication_year, primary_topic, authorships.countries, lacuna; cada célula de código compila (`ast.parse`, ignorando linhas que começam com `%`). Se possível, teste `classificar_pares` com uma matriz sintética pequena.

### Formato B: página `.html`

Um único arquivo HTML autossuficiente, sem CDN e sem bibliotecas externas. Consulta a API direto do navegador (o OpenAlex aceita CORS), com `fetch` e `AbortController` (timeout 30 s). Gráficos em SVG gerado por JavaScript puro.

- Formulário no topo: nome do estudo, duas caixas de texto (uma linha por item, sinônimos separados por `|`), ano inicial, e-mail, janela recente, tipos. Botão "Mapear" com barra de progresso e estimativa de tempo.
- Abas: Panorama, Matriz e lacunas, Leitura-base, Metodologia e limitações.
- Matriz interativa: passar o mouse mostra observado, esperado, razão e tendência. Clicar numa célula abre o painel de leitura-base (trabalhos mais citados e mais recentes com links para DOI e OpenAlex, países e instituições mais ativos).
- Controles deslizantes para ESPERADO_MINIMO, RAZAO_LACUNA e LIMITE_POUCO_EXPLORADO, com reclassificação e reordenação imediatas, sem nova consulta.
- Alternar a matriz entre "contagem", "razão observado/esperado" e "tendência".
- Exportar: PNG de cada gráfico (escala 3x), CSV da matriz e do ranking.
- Bloquear a consulta se o e-mail for o placeholder. Envolver toda leitura da resposta em try/catch e tratar consultas sem resultado (célula com zero publicações é um achado válido, não um erro).
- Layout responsivo, fonte do sistema, cores em tokens CSS, modo claro e escuro.

### Seção "Metodologia e limitações" (obrigatória nos dois formatos)

Inclua, em linguagem simples:
- Contagens dependem das palavras escolhidas. Sinônimos, siglas e termos em outros idiomas mudam o resultado. Teste variações dos termos.
- A busca casa texto, não o conceito. Termos ambíguos geram falsos positivos, e trabalhos sem resumo indexado podem faltar.
- Pouca literatura não significa oportunidade. Pode indicar que o cruzamento é irrelevante, inviável ou nomeado de outra forma. Valide lendo os trabalhos da leitura-base e buscando por termos vizinhos.
- O esperado assume independência entre os eixos. Use-o para comparar pares entre si, não como valor exato.
- O ano corrente é parcial e a indexação tem atraso. Por isso a tendência usa anos completos.
- A cobertura do OpenAlex varia por área, idioma e região.
- Não use o resultado sozinho para justificar originalidade em um artigo ou projeto. Complemente com busca em outras bases (Scopus, Web of Science, Google Scholar) e leitura crítica.

### Entrega

- Salve o arquivo e disponibilize para download. Se você não cria arquivos, entregue o conteúdo completo em UM único bloco de código, com o nome do arquivo sugerido acima dele e instruções de 2 linhas para salvar com a extensão correta.
- Se você não executa código ou não consegue acessar a API, diga isso claramente e não afirme que validou ou testou o que não testou.
- Resposta final em português, curta: (1) que o arquivo foi criado, (2) o nome, (3) o caminho absoluto ou link de download, (4) que o usuário precisa trocar `EMAIL_CONTATO = "seu-email@exemplo.com"` por um e-mail válido, (5) como abrir (Colab, Jupyter, VS Code ou navegador), (6) um aviso de que a primeira execução deve usar listas pequenas (3 x 3) para conferir se os termos retornam resultados sensatos.
- Nome do arquivo: `estado_da_arte_lacunas_openalex_<estudo_em_snake_case>.ipynb` ou `.html`.

### ENTRADAS DESTA EXECUÇÃO (troque pelos seus valores)

```
NOME_DO_ESTUDO = "Aprendizado de máquina interpretável em pirólise"
LISTA_METODOS = [
  "symbolic regression | genetic programming",
  "machine learning",
  "explainable AI | interpretable machine learning",
  "surrogate model | metamodel",
  "physics-informed neural network",
  "AutoML"
]
LISTA_APLICACOES = [
  "pyrolysis",
  "biochar",
  "bio-oil",
  "hidrochar"
]
ANO_INICIAL = 2020
JANELA_RECENTE = 3
FORMATO_SAIDA = "ipynb"   # ou "html" ou "ambos"
```
