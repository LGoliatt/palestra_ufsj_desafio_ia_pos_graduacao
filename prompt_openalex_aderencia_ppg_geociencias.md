---
name: openalex-aderencia-ppg
description: A partir de uma lista de ORCIDs de possíveis docentes, gera um arquivo pronto para compartilhar (notebook Jupyter .ipynb e/ou página HTML) que consulta o OpenAlex e mede a aderência de cada pesquisador a uma área de avaliação da CAPES (padrão: Geociências), calcula a matriz de similaridade entre eles, o grafo de coautoria, as métricas agregadas do grupo e um índice de força para um novo programa de pós-graduação (APCN), com checklist dos critérios do Documento Orientador. Use quando o usuário quiser avaliar um corpo docente para propor um novo PPG, mapear linhas de pesquisa, medir coesão e produção do grupo ou comparar aderência de pesquisadores a uma área.
---

# Prompt: aderência de pesquisadores e força de um novo PPG com OpenAlex

Versão 4.0 | Funciona em qualquer assistente de IA (ChatGPT, Claude, Gemini, Copilot, Mistral, etc.)

## COMO USAR (para a pessoa)

1. Copie TODO o conteúdo deste arquivo, a partir da linha "INSTRUÇÕES PARA A IA".
2. Cole no chat da IA e confira o bloco ENTRADAS no final (a lista de ORCIDs já está preenchida).
3. Escolha o formato: `ipynb`, `html` ou `ambos`.
4. Receba os arquivos, baixe e abra. O HTML abre com duplo clique no navegador. O notebook abre no Google Colab, JupyterLab ou VS Code.

Importante: se a IA não conseguir abrir o link do Documento Orientador (links longos do gov.br costumam falhar), anexe o PDF no chat.

---

## INSTRUÇÕES PARA A IA

Atue como cientista de dados, bibliometrista e analista de redes, especialista em Python, JavaScript, networkx e na avaliação de propostas de cursos novos (APCN) da CAPES. Escreva todo texto visível ao usuário em português.

### Objetivo

Criar um arquivo real e funcional que, para uma lista de ORCIDs:

1. Resolve cada ORCID no OpenAlex e coleta os trabalhos da janela de análise.
2. Mede a **aderência de cada pesquisador à área-alvo** (padrão: Geociências, Área 05 da CAPES).
3. Calcula a **matriz de similaridade** temática entre os pesquisadores.
4. Constrói o **grafo de conexão** (coautoria e similaridade) com métricas de rede.
5. Apresenta as **métricas agregadas do grupo** extraídas do OpenAlex.
6. Calcula o **índice de força do PPG novo** (0 a 100), em dimensões, com checklist dos critérios do Documento Orientador de APCN.
7. Sugere **linhas de pesquisa candidatas**, a contribuição de cada docente e um **núcleo docente recomendado**.

### Entradas (use os padrões se não informado)

| Variável | Padrão |
|---|---|
| ORCIDS | obrigatória (lista, formato 0000-0000-0000-0000, aceitar também a URL completa) |
| NOME_PPG | "Novo PPG" |
| AREA_ALVO | "Geociências" (CAPES, Área 05) |
| NIVEL | "mestrado" ("mestrado", "doutorado" ou "mestrado e doutorado") |
| MODALIDADE | "acadêmico" ou "profissional" |
| JANELA_ANOS | 5 (últimos anos completos mais o ano corrente, como no Documento Orientador) |
| URL_DOCUMENTO_APCN | https://www.gov.br/capes/pt-br/acesso-a-informacao/acoes-e-programas/avaliacao/sobre-a-avaliacao/areas-avaliacao/sobre-as-areas-de-avaliacao/colegio-de-ciencias-exatas-tecnologicas-e-multidisciplinar/ciencias-exatas-e-da-terra/Geociencias_Documento_Orientador_APCN_2023.pdf/@@display-file/file |
| EMAIL_CONTATO | "seu-email@exemplo.com" (placeholder) |
| FORMATO_SAIDA | "ipynb", "html" ou "ambos". Se não informado, gere os dois |
| COMPLEMENTOS_CSV | opcional (veja "Dados complementares") |
| CHAVE_API | "" (opcional) |

Parâmetros de decisão, todos ajustáveis (no HTML por controles deslizantes; no notebook por variáveis):

| Parâmetro | Padrão |
|---|---|
| LIMIAR_ADERENCIA | 30 (índice 0 a 100 para o pesquisador contar como "elegível") |
| PESO_TAXONOMIA / PESO_PALAVRAS | 0.6 / 0.4 (composição do índice de aderência) |
| LIMIAR_SIMILARIDADE | 0.35 (cosseno para aresta de similaridade) |
| MAX_AUTORES_POR_TRABALHO | 50 (acima disso o trabalho não gera aresta de coautoria) |
| DENSIDADE_ALVO | 0.15 |
| META_INTERNACIONAL | 0.40 (fração de trabalhos com coautor de outro país) |
| PESOS_FORCA | massa crítica 15, aderência 20, produção e impacto 20, coesão 15, equilíbrio 10, coerência 10, internacionalização 10 |

### Parâmetros do Documento Orientador de APCN, Área 05 Geociências

Leia o documento em URL_DOCUMENTO_APCN (ou o PDF anexado) e extraia os valores vigentes. Se não conseguir acessá-lo, use os valores abaixo, que vieram da edição de 2021 do documento, e AVISE na interface e na resposta que precisam ser conferidos contra a edição de 2023:

- Corpo docente: apenas doutores; no mínimo 80% de permanentes entre permanentes e colaboradores.
- Mínimo de docentes permanentes: 8 para mestrado e 10 para doutorado (acadêmico e profissional).
- Pelo menos 70% dos permanentes (60% no mestrado profissional) em tempo integral na instituição e alocados em apenas um programa; os demais em até três programas; mínimo de 10 horas semanais no programa.
- Pelo menos 60% dos permanentes com ao menos uma orientação concluída de mestrado; orientar doutorado exige ao menos uma orientação de mestrado concluída.
- Produção: pelo menos 70% dos permanentes (60% no profissional) com um artigo por ano em periódicos indexados em bases internacionais, particularmente Web of Science e Scopus, nos últimos cinco anos. Produção per capita divide o artigo entre os docentes permanentes coautores. Só conta produção ligada às linhas de pesquisa.
- Produção distribuída de forma equilibrada entre docentes e linhas, com linhas produtivas. Linhas de baixa produção devem ser evitadas ou fundidas.
- Mestrado novo com corpo docente pouco experiente: 1 a 2 discentes por docente permanente até as primeiras defesas. Doutorado: máximo recomendado de 3 a 6 orientações por docente.
- Valorizados: bolsistas de produtividade do CNPq, pós-doutorado, liderança (editoria, comitês, redes), parcerias e mobilidade.
- A proposta deve ser inovadora e sem sobreposição com cursos da mesma instituição e região. Participação de docentes externos não pode caracterizar dependência.
- Doutorado: o corpo docente permanente do doutorado não deve ser recorte significativo do corpo do mestrado.

### Regras da API (valem para os dois formatos)

- Base: `https://api.openalex.org`. Sempre enviar `mailto`=EMAIL_CONTATO. Se CHAVE_API estiver preenchida, enviar como `api_key`. Em caso de 401, 403 ou 429, orientar a conferir as regras atuais em docs.openalex.org. Timeout de 30 s, pausa de 0,15 s, repetição com espera crescente no 429, mensagens claras.
- Bloquear a consulta se o e-mail ainda for o placeholder.
- **Autor por ORCID**: `GET /authors/orcid:<ORCID>`; se falhar, `GET /authors?filter=orcid:https://orcid.org/<ORCID>`. Campos usados: `id`, `display_name`, `orcid`, `works_count`, `cited_by_count`, `summary_stats` (h_index, i10_index, 2yr_mean_citedness), `last_known_institutions` (ou `affiliations`), `topics`, `counts_by_year`. ORCID não encontrado vai para uma lista de pendências e a análise continua com os demais.
- **Confirmação de identidade**: mostrar uma tabela com ORCID, nome, instituição, h-index e total de trabalhos para o usuário conferir antes de interpretar resultados.
- **Trabalhos na janela**: `GET /works?filter=author.id:<A_ID>,from_publication_date:<ano_inicial>-01-01&per-page=200&cursor=*&select=id,doi,display_name,publication_year,type,fwci,cited_by_count,citation_normalized_percentile,open_access,primary_location,primary_topic,topics,authorships` (paginar por `meta.next_cursor`). Descartar duplicatas por `id`.
- **Periódicos em lote**: reunir os `primary_location.source.id` e consultar `GET /sources?filter=openalex_id:S1|S2|...&per-page=50&select=id,display_name,type,is_in_doaj,is_core,summary_stats` em blocos de 50.
- **Taxonomia da área-alvo**: descobrir os identificadores pela API, sem fixar números de memória: `GET /fields?search=Earth and Planetary` (campo "Earth and Planetary Sciences"), depois `GET /subfields?filter=field.id:<id>&per-page=50`. Os campos afins padrão são "Environmental Science" (e, se o usuário pedir, outros). Para outra AREA_ALVO, derive os campos e subcampos equivalentes.
- **Texto da área**: por pesquisador, `GET /works?filter=author.id:<A_ID>,from_publication_date:<ano_inicial>-01-01,title_and_abstract.search:<EXPRESSAO_AREA>&per-page=1` e use `meta.count`. Se a API recusar o filtro, use `search=`. EXPRESSAO_AREA padrão (editável, em inglês): `(geology OR geophysics OR geochemistry OR petrology OR hydrogeology OR hydrology OR groundwater OR seismology OR meteorology OR climatology OR oceanography OR paleontology OR stratigraphy OR sedimentology OR remote sensing OR "earth system" OR geomorphology OR "soil science" OR mineral OR geotechnical)`.
- Sem tokens, sem credenciais, sem arquivos externos além do COMPLEMENTOS_CSV opcional.

### Dados complementares (opcional)

O OpenAlex não traz vínculo, regime de trabalho, orientações, bolsas ou pós-doutorado. Aceite um CSV opcional com colunas: `orcid`, `doutor` (s/n), `integral` (s/n), `n_outros_programas`, `orientacoes_mestrado_concluidas`, `bolsa_produtividade` (s/n), `pos_doc` (s/n), `externo` (s/n). No notebook, leia o arquivo se existir; no HTML, ofereça uma caixa para colar o CSV. Sem ele, o checklist marca esses itens como "não verificável com OpenAlex" e lista o que falta, e gere o modelo do CSV para download.

### Método (definições obrigatórias, iguais nos dois formatos)

**Métricas por pesquisador (janela):** trabalhos, artigos (`type`=article, em periódico), trabalhos por ano, citações, FWCI médio e mediano (ignorando nulos, informando quantos entraram), fração no top 10% (`citation_normalized_percentile.is_in_top_10_percent`), fração em acesso aberto, nº de periódicos distintos, proxy de veículo (mediana de `2yr_mean_citedness` das fontes), nº de países e de instituições coautoras, h-index e i10 do perfil, anos de carreira (do menor `publication_year` do perfil), artigos por ano e se teve pelo menos 1 artigo em todos os anos da janela.

**Aderência à área (por pesquisador):**
- `ad_tax_core`: fração dos trabalhos da janela cujo `primary_topic.field` está nos campos centrais da área-alvo.
- `ad_tax_afim`: fração nos campos afins.
- `ad_taxonomia = 100 x (ad_tax_core + 0.5 x ad_tax_afim)`, limitado a 100.
- `ad_palavras = 100 x (trabalhos que casam com EXPRESSAO_AREA / trabalhos da janela)`, limitado a 100.
- `indice_aderencia = PESO_TAXONOMIA x ad_taxonomia + PESO_PALAVRAS x ad_palavras`.
- Classe: Alta (índice >= 50), Média (30 a 49,9), Baixa (10 a 29,9), Nula (< 10).
- Por que dois componentes: a classificação de tópicos do OpenAlex é automática e pode subestimar quem aplica métodos computacionais a problemas de geociências. O componente textual capta esses casos. Mostre os dois lado a lado.

**Subáreas:** distribuição dos trabalhos de cada pesquisador pelos subcampos da área-alvo (via `primary_topic.subfield`), para o mapa de cobertura.

**Similaridade (matriz N x N):** vetor por pesquisador sobre tópicos (peso = soma do `score` de cada tópico nos trabalhos da janela), TF-IDF sobre os pesquisadores, cosseno. Diagonal = 1. Ordene a matriz por agrupamento hierárquico.

**Grafo:** nós = pesquisadores. Aresta de coautoria = trabalhos da janela com ambos, peso = nº de trabalhos (ignorar trabalhos acima de MAX_AUTORES_POR_TRABALHO). Aresta de similaridade (tracejada) quando o cosseno >= LIMIAR_SIMILARIDADE e não há coautoria: representa colaboração potencial. Métricas de rede: grau, intermediação, comunidades (modularidade gulosa), densidade, nº de componentes, fração no maior componente.

**Linhas de pesquisa candidatas:** comunidades de similaridade (agrupamento hierárquico do cosseno ou modularidade sobre o grafo de similaridade). Nomeie cada linha pelos 3 tópicos mais pesados. Reporte docentes por linha, produção por linha e alerte linhas com apenas 1 docente ou produção muito baixa (o documento recomenda fundir).

**Dimensões da força do PPG (D1 a D7, cada uma de 0 a 1):**
- D1 massa crítica = min(1, nº de elegíveis / mínimo de permanentes do nível). Elegível = índice de aderência >= LIMIAR_ADERENCIA. Para "mestrado e doutorado", use o mínimo do doutorado.
- D2 aderência = média do índice de aderência dos membros / 100.
- D3 produção e impacto = média de: min(1, fração de docentes com média >= 1 artigo por ano / 0,70), min(1, FWCI mediano do grupo / 1,0), min(1, fração top 10% do grupo / 0,10).
- D4 coesão = 0,5 x fração no maior componente de coautoria + 0,5 x min(1, densidade / DENSIDADE_ALVO).
- D5 equilíbrio = média de (1 - Gini dos artigos por docente) e a entropia normalizada da distribuição de docentes entre as linhas candidatas.
- D6 coerência temática = min(1, similaridade média par a par / 0,5).
- D7 internacionalização = min(1, fração de trabalhos com coautor de outro país / META_INTERNACIONAL).
- `FORCA = soma(PESOS_FORCA x D) / soma(PESOS_FORCA) x 100`.
- Classe: Forte (>= 75), Promissor (55 a 74), Frágil (35 a 54), Insuficiente (< 35).
- Isto é um apoio ao planejamento, NÃO a nota que a CAPES atribuirá. Diga isso na interface.

**Checklist do Documento Orientador (semáforo):** para cada critério, mostre Atende (verde), Atenção (amarelo), Não atende (vermelho) ou Não verificável (cinza), com o valor medido e a fonte:
- Nº de permanentes elegíveis frente ao mínimo (OpenAlex).
- Fração com pelo menos 1 artigo por ano na janela, meta 70% (60% profissional). Observação obrigatória: OpenAlex não informa indexação em Web of Science ou Scopus; use o artigo em periódico como proxy e peça confirmação nessas bases.
- Distribuição equilibrada da produção (Gini e participação dos 3 mais produtivos, alerta se acima de 50%).
- Linhas produtivas e docentes equilibrados entre linhas.
- Dependência de docentes externos (se COMPLEMENTOS_CSV informar).
- Itens apenas com complementos: doutorado, tempo integral (70% ou 60%), alocação em um só programa, 10 horas semanais, 60% com orientação concluída, bolsistas PQ, pós-doutorado. Sem o CSV, "Não verificável".
- Itens institucionais fora do alcance dos dados: infraestrutura, PDI, regimento, sobreposição com PPGs da instituição e da região (sugerir consulta à Plataforma Sucupira).

**Contribuição individual:** `delta_i` = FORCA do grupo menos FORCA do grupo sem o pesquisador i (método leave-one-out). Positivo = o pesquisador aumenta a força. Classes: Essencial (delta >= 3), Contribui (0 a 3), Neutro/Peso (delta < 0). Mostre também as dimensões que mais mudam.

**Núcleo recomendado:** seleção gulosa para formar um grupo do tamanho mínimo do nível (ou N_NUCLEO informado) que maximize FORCA, trocando membros enquanto houver melhora. Mostre quem entra, quem fica de fora e os ganhos por dimensão. Apresente como sugestão para discussão, nunca como julgamento de mérito individual.

**Métricas agregadas do grupo (painel):** nº de pesquisadores resolvidos, trabalhos e artigos na janela, trabalhos por ano, citações, h-index (média, mediana, máximo), FWCI médio e mediano, % top 10%, % acesso aberto, nº de periódicos distintos e principais, principais tópicos e subáreas, principais países e instituições coautoras, % de trabalhos com coautoria interna ao grupo, produção per capita (com divisão do artigo entre coautores do grupo, como no documento), anos de carreira médios, composição por classe de aderência.

**Outras métricas de interesse para criar o PPG (inclua todas):**
1. Cobertura de subáreas: quantos docentes com pelo menos 10% dos trabalhos em cada subárea da área-alvo, e quais subáreas ficam descobertas (informativo; o documento valoriza coerência, então não penalize a ausência de subáreas).
2. Pares de alto potencial: alta similaridade e nenhuma coautoria (convites naturais a colaborar).
3. Pontes: pesquisadores de alta intermediação que ligam comunidades.
4. Concentração de produção: Gini e participação dos 3 mais produtivos.
5. Tendência: trabalhos por ano do grupo (crescendo ou caindo) e citações por ano.
6. Maturidade: anos de carreira, trabalhos totais e distribuição de h-index.
7. Alcance internacional: países coautores e fração de trabalhos internacionais.
8. Qualidade do veículo (proxy): mediana de `2yr_mean_citedness` dos periódicos usados.
9. Aderência por nível: quantos pesquisadores seriam elegíveis em cada limiar (gráfico de sensibilidade de 10 a 60).
10. Sensibilidade da força aos pesos (tabela com 3 cenários: padrão, foco em produção, foco em aderência).
11. Lista de pendências de dados: ORCIDs não encontrados, perfis com poucos trabalhos na janela (possível perfil incompleto no OpenAlex), pesquisadores sem instituição.

### Visualizações (valem para os dois formatos)

1. **Painel agregado**: cartões com os números principais e barras de trabalhos por ano (azuis, valor no topo).
2. **Aderência por pesquisador**: barras horizontais empilhadas (campos centrais, afins, outros) com o índice ao lado e a classe.
3. **Matriz de similaridade**: mapa de calor ordenado por agrupamento, valores nas células, escala sequencial azul, nomes abreviados.
4. **Grafo de conexão**: layout por forças; cor = índice de aderência; tamanho = trabalhos na janela; contorno por comunidade (linha candidata); aresta cheia = coautoria (espessura = nº de trabalhos); aresta tracejada = similaridade sem coautoria. Legenda completa.
5. **Força do PPG**: indicador 0 a 100 com a classe, gráfico radar das 7 dimensões e tabela com o valor de cada dimensão.
6. **Checklist semáforo** do Documento Orientador.
7. **Cobertura de subáreas**: mapa de calor pesquisador x subárea.
8. **Contribuição individual** (barras de delta) e **núcleo recomendado**.
9. **Linhas candidatas**: tabela com docentes, tópicos principais e produção.
10. Aparência limpa e profissional. Não dependa só de cor para o significado (use rótulos, contorno e texto).

### Formato A: notebook `.ipynb`

Crie o arquivo de verdade em JSON nbformat 4 (nbformat_minor 5), compatível com Python 3.9+, executável célula por célula. Dependências: requests, pandas, numpy, networkx, matplotlib, seaborn, scikit-learn, scipy. Célula opcional: `%pip install -q requests pandas numpy networkx matplotlib seaborn scikit-learn scipy`.

Seções, cada uma com Markdown explicativo antes do código:

1. Título, objetivos e aviso de que o resultado é apoio ao planejamento
2. Instalação
3. Importações
4. Configuração (entradas, parâmetros de decisão, parâmetros do Documento Orientador com a nota sobre a edição consultada, aviso sobre o e-mail)
5. Funções da API (com docstring)
6. Resolução dos ORCIDs e confirmação de identidade (e pendências)
7. Coleta dos trabalhos, fontes e taxonomia
8. Painel de métricas agregadas
9. Aderência à área
10. Matriz de similaridade
11. Grafo de conexão e métricas de rede
12. Linhas de pesquisa candidatas
13. Força do PPG e checklist
14. Contribuição individual e núcleo recomendado
15. Outras métricas de interesse
16. Exportações (PNG 300 DPI, CSV de cada tabela, GEXF do grafo, modelo de COMPLEMENTOS_CSV)
17. Metodologia e limitações

Se plotly estiver disponível, ofereça versões interativas do grafo e da matriz em células opcionais.

Validação programática (se você executa código): arquivo existe; JSON válido; `nbformat` definido; células Markdown e de código presentes; contém OpenAlex, mailto, orcid, fwci, aderência, similaridade, networkx, Geociências; cada célula compila (`ast.parse`, ignorando linhas que começam com `%`). Teste as funções de aderência, similaridade, Gini, força e leave-one-out com dados sintéticos pequenos e com a API simulada.

### Formato B: página `.html`

Um único arquivo HTML autossuficiente, sem CDN e sem bibliotecas externas, que consulta o OpenAlex direto do navegador (CORS permitido) com `fetch` e `AbortController` (timeout 30 s). Implemente em JavaScript puro: TF-IDF e cosseno, agrupamento hierárquico, centralidades, comunidades, Gini, entropia, força e leave-one-out, e o grafo e os gráficos em SVG ou canvas.

- Cabeçalho com caixa para colar os ORCIDs (um por linha, aceitando "Nome; ORCID"), campos de nível, modalidade, janela, e-mail, EXPRESSAO_AREA editável e caixa opcional para colar o COMPLEMENTOS_CSV. Botão "Analisar" com barra de progresso e lista de pendências.
- Após resolver os ORCIDs, mostrar a tabela de confirmação de identidade e só então seguir.
- Abas: Painel, Aderência, Similaridade, Grafo, Força e checklist, Contribuição e núcleo, Metodologia e limitações.
- Controles deslizantes para LIMIAR_ADERENCIA, pesos da aderência, LIMIAR_SIMILARIDADE e PESOS_FORCA, com recálculo imediato sem nova consulta. Seletor para incluir ou excluir pesquisadores e ver o efeito na força.
- Grafo: arrastar nós, tooltip (nome, instituição, índice de aderência, FWCI, trabalhos), clique abre painel com links para o perfil OpenAlex e o ORCID.
- Exportar: PNG de cada gráfico (escala 3x), CSV das tabelas, GEXF do grafo, JSON completo da análise e uma aba "Relatório" imprimível (resumo em texto, força, checklist, núcleo recomendado).
- Bloquear a consulta se o e-mail for o placeholder. Envolver toda leitura da resposta em try/catch, tratar ORCID não encontrado e perfil sem trabalhos na janela.
- Layout responsivo, fonte do sistema, cores em tokens CSS, modo claro e escuro.

### Seção "Metodologia e limitações" (obrigatória nos dois formatos)

Inclua, em linguagem simples:
- A aderência usa a classificação automática de tópicos do OpenAlex e uma busca textual. Ambas erram. Confira os resultados de cada pessoa antes de usá-los.
- O vínculo entre ORCID e perfil no OpenAlex pode estar incompleto ou misturado com homônimos. Pesquisadores com poucos trabalhos na janela podem ter perfil incompleto.
- Produção que não está no OpenAlex (por exemplo, itens só no Lattes) não é contada.
- O OpenAlex não informa indexação em Web of Science ou Scopus nem a classificação de periódicos da CAPES. O critério de "um artigo por ano" é um proxy e deve ser confirmado nessas bases.
- Vínculo, regime de trabalho, outros programas, orientações concluídas, bolsas de produtividade, pós-doutorado, infraestrutura e sobreposição com outros PPGs não vêm do OpenAlex. Complete com Lattes, Plataforma Sucupira e dados da instituição.
- Os parâmetros do Documento Orientador vieram da edição indicada na interface. Confira contra o documento vigente antes de submeter.
- O índice de força é uma ferramenta de planejamento. Não representa a avaliação da CAPES nem o mérito de pessoas. Indicadores bibliométricos têm vieses (área, idade de carreira, idioma, tamanho das colaborações). Discuta os resultados com o grupo de forma transparente.
- Pesos e limiares são escolhas. Use a análise de sensibilidade antes de tomar decisões.

### Entrega

- Salve os arquivos e disponibilize para download. Se você não cria arquivos, entregue o conteúdo completo em UM único bloco de código por arquivo, com o nome sugerido acima dele e instruções de 2 linhas para salvar com a extensão correta.
- Se você não executa código ou não acessa a API, diga isso claramente e não afirme que validou ou testou o que não testou.
- Resposta final em português, curta: (1) o que foi criado, (2) nomes dos arquivos, (3) caminho absoluto ou link de download, (4) que o usuário precisa trocar `EMAIL_CONTATO = "seu-email@exemplo.com"` por um e-mail válido, (5) como abrir, (6) qual edição do Documento Orientador foi usada, (7) aviso de que a primeira execução deve usar poucos ORCIDs (3 a 5) para conferir identidades e termos.
- Nomes dos arquivos: `aderencia_ppg_<area>_openalex.ipynb` e `aderencia_ppg_<area>_openalex.html`.

### ENTRADAS DESTA EXECUÇÃO (confira e ajuste)

```
NOME_PPG = "Novo PPG em Geociências"
AREA_ALVO = "Geociências"
NIVEL = "mestrado"            # ou "doutorado" ou "mestrado e doutorado"
MODALIDADE = "acadêmico"      # ou "profissional"
JANELA_ANOS = 5
FORMATO_SAIDA = "ambos"       # "ipynb", "html" ou "ambos"
ORCIDS = [
  "0000-0002-2844-9470",
  "0000-0001-7017-4653",
  "0000-0003-0821-126X",
  "0000-0002-0508-8959",
  "0000-0001-9573-8996",
  "0000-0003-2012-7802",
  "0000-0003-2486-1190",
  "0000-0003-1899-9441",
  "0000-0002-7205-9509",
  "0000-0001-9943-1334",
  "0000-0002-5330-7407",
  "0000-0001-7874-9271",
  "0000-0002-6070-7257",
  "0000-0003-1321-158X",
  "0000-0002-4974-6802",
  "0000-0002-0261-4298",
  "0000-0002-3632-9639"
]
```
