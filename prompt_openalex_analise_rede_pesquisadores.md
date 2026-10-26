---
name: openalex-rede-pesquisadores
description: Gera um grafo interativo de similaridade científica entre um pesquisador (via ORCID) e os principais autores de um tema, usando OpenAlex, com tabela de métricas e centralidades, em .html ou .ipynb. Use quando o usuário pedir rede de pesquisadores, potenciais colaboradores, ou chamar sem argumentos.
---

# Prompt: Grafo de similaridade científica (OpenAlex, ORCID + tema)

Saída: `.html` (abre em qualquer navegador) ou `.ipynb` (Jupyter, Colab, VS Code).
Todo o texto ao usuário e dentro dos arquivos deve estar em português (pt-BR).

## 0. Compatibilidade (leia primeiro)

Este prompt deve funcionar em qualquer IA. Descubra o que você consegue fazer:

- **Nível A, executa código, tem internet e cria arquivos:** gere o arquivo, rode, valide e entregue.
- **Nível B, não executa código ou não acessa a internet:** entregue o código completo em UM único bloco ```python``` (sem fragmentar), pronto para colar no Google Colab (colab.research.google.com). Se o formato escolhido foi `.html`, o código grava o `.html` ao final. Se foi `.ipynb`, entregue as células numeradas ("Célula 1, Célula 2...") para colar uma a uma.

Nunca afirme que gerou ou validou algo que você não executou. Nunca invente dados de pesquisadores.

## 1. Se o usuário não digitou nada (ou só "oi", "olá", "teste", "ajuda", ou faltou ORCID e tema)

Responda SOMENTE com o bloco abaixo, exatamente assim, sem código e sem explicação extra:

````
🔬 GRAFO DE SIMILARIDADE CIENTÍFICA (OpenAlex)

Descubra quem pesquisa temas parecidos com o seu, quem é mais
influente na área e quem pode ser um bom colaborador. Você recebe
um grafo interativo + tabela com métricas.

COMO USAR (escolha um jeito)

1) Mais rápido: copie, troque o ORCID e o tema, e envie
   Gere uma rede de pesquisadores com base no ORCID
   0000-0002-2844-9470 para o tema "pyrolysis machine learning"
   em html

2) Resumido
   ORCID 0000-0002-2844-9470, tema: pyrolysis ML, html

3) Controle total: copie, ajuste e envie
   ORCID_USUARIO     = "0000-0002-2844-9470"
   TERMO_BUSCA       = "pyrolysis machine learning"
   MAX_WORKS_TEMA    = 200
   MAX_AUTORES       = 50
   MIN_SIMILARIDADE  = 0.1
   FORMATO_SAIDA     = "html"        # ou "ipynb"
   EMAIL_CONTATO     = "seu-email@exemplo.com"

FORMATO DE SAÍDA (você escolhe)
   html   abre em qualquer navegador, fácil de enviar por e-mail
   ipynb  caderno editável no Jupyter / Google Colab / VS Code
   Não sabe qual? Escolha html.

O QUE VOCÊ PRECISA TER
   • Seu ORCID (formato 0000-0000-0000-0000, veja em orcid.org)
   • Um tema de pesquisa (em inglês costuma dar melhores resultados)
   • Um e-mail real (a OpenAlex pede, é grátis e sem cadastro).
     Eu pergunto na hora de gerar. Não invento e-mail.

O QUE CADA PARÂMETRO FAZ
   MAX_WORKS_TEMA    quantos artigos do tema são analisados
   MAX_AUTORES       quantos autores entram no grafo (mais = mais lento)
   MIN_SIMILARIDADE  corte para desenhar uma ligação (0 a 1);
                     suba para um grafo mais limpo, desça para mais ligações
````

Depois do bloco, termine com UMA pergunta curta: "Me envie seu ORCID, o tema e o formato (html ou ipynb)."

## 2. Se o usuário deu algo (fluxo mínimo de perguntas)

1. Extraia: ORCID, tema, formato, e-mail, e os parâmetros opcionais.
2. Padrões para o que faltar: `MAX_WORKS_TEMA=200`, `MAX_AUTORES=50`, `MIN_SIMILARIDADE=0.1`.
3. ORCID e tema são obrigatórios. Valide o ORCID com a regex `^\d{4}-\d{4}-\d{4}-\d{3}[\dX]$`; se estiver errado, diga o que está errado e peça de novo.
4. Se faltar **formato** ou **e-mail**, faça UMA pergunta juntando os dois. Se o usuário não quiser informar e-mail, não bloqueie: gere com `EMAIL_CONTATO = ""` e o código pede o e-mail na execução.
5. Não faça mais de uma rodada de perguntas antes de gerar.

## 3. Parâmetros

```python
API_URL          = "https://api.openalex.org"
EMAIL_CONTATO    = ""                         # vazio = o código pergunta ao executar
ORCID_USUARIO    = "0000-0002-2844-9470"
TERMO_BUSCA      = "pyrolysis machine learning"
MAX_WORKS_TEMA   = 200
MAX_AUTORES      = 50
MIN_SIMILARIDADE = 0.1
FORMATO_SAIDA    = "html"                     # "html" ou "ipynb"
```

Código: se `EMAIL_CONTATO` estiver vazio ou for `seu-email@exemplo.com`, pedir com `input("Digite seu e-mail para a API OpenAlex: ")`. Sem `input` disponível, seguir sem `mailto` e avisar. Use `mailto` em TODAS as requisições. Nunca invente e-mail.

## 4. O que o relatório deve conter

1. Autor encontrado pelo ORCID e seus tópicos principais
2. Tabela dos autores do tema, ordenada por similaridade com o usuário
3. Grafo interativo (você em vermelho e maior)
4. Heatmap de correlação entre métricas
5. Metodologia e limitações

Estrutura em ordem: título e descrição, objetivos, dependências, importações, parâmetros, funções, coleta, tabela, grafo, heatmap, observações.

**Funções:** `buscar_autor_por_orcid`, `buscar_works_do_tema`, `extrair_autores_e_topicos`, `calcular_fwci_autores`, `calcular_similaridade`, `construir_grafo`, `calcular_metricas_rede`, `montar_dataframe_final`, `visualizar_grafo`.

**Dependências:** `requests pandas numpy matplotlib seaborn networkx pyvis`. Python 3.9+. No `.ipynb`, a primeira célula é `%pip install -q requests pandas numpy matplotlib seaborn networkx pyvis`.

## 5. Dados da OpenAlex

Use sempre `select=` para reduzir o tamanho das respostas e `timeout=30`. Se um nome de campo falhar, consulte a resposta bruta e ajuste em vez de inventar.

1. Autor pelo ORCID: `GET /authors?filter=orcid:{ORCID}` e guarde `id`, `display_name`, `works_count`, `cited_by_count`, `summary_stats`, `topics`. ORCID não encontrado: mensagem clara e parar.
2. Works do tema: `GET /works?search={TERMO}&per-page=200&select=id,fwci,cited_by_count,authorships,topics`. Para mais de 200, paginar com `cursor`.
3. Autores do tema: contar aparições em `authorships[].author` e escolher os `MAX_AUTORES` mais frequentes (mais o usuário).
4. Detalhes em lote: `GET /authors?filter=openalex:A1|A2|...&per-page=50` (até 50 ids por chamada). Não buscar um autor por vez.
5. Autor sem `topics`: pular. Tema sem resultado: mensagem clara e parar.

## 6. Métricas

**FWCI.** Os works da OpenAlex têm o campo `fwci`. Calcule `fwci_medio` do autor como a média do `fwci` dos works do tema em que ele aparece (ignorar nulos) e deixe claro que é a média **dentro dos artigos coletados**, não o FWCI total da carreira. Se `fwci` vier ausente na API, use proxy `(cited_by_count / works_count)` e documente como PROXY.

**Similaridade.** Para cada autor, vetor de tópicos (`topics[].id` → `count`). Similaridade do cosseno entre usuário e cada autor (0 a 1). Para as arestas entre autores, o mesmo cosseno entre os dois vetores. Só cria aresta se `similaridade >= MIN_SIMILARIDADE`. Bônus opcional: co-autoria direta nos works coletados.

```python
import numpy as np

def cosseno(a: dict, b: dict) -> float:
    chaves = set(a) | set(b)
    va = np.array([a.get(k, 0) for k in chaves], dtype=float)
    vb = np.array([b.get(k, 0) for k in chaves], dtype=float)
    den = np.linalg.norm(va) * np.linalg.norm(vb)
    return float(va @ vb / den) if den else 0.0
```

**Rede.** `networkx.Graph` com peso = similaridade. Calcular `degree_centrality`, `betweenness_centrality(weight=None)`, `closeness_centrality`. Nós: tamanho ∝ `fwci_medio` (com piso mínimo) ou citações; cor ∝ betweenness; usuário vermelho e maior. Aresta: espessura ∝ peso.

**Grafo autocontido (importante).** O pyvis padrão carrega bibliotecas de CDN. Para o HTML funcionar offline e em qualquer lugar:

```python
from pyvis.network import Network
net = Network(height="750px", width="100%", bgcolor="#222222",
              font_color="white", cdn_resources="in_line")
# ... adicionar nós/arestas com title= (tooltip com todas as métricas) ...
html_grafo = net.generate_html()   # string; não use notebook=True fora do Jupyter
```
Embuta `html_grafo` no relatório com `<iframe srcdoc="...">` (escapando aspas com `html.escape`) ou salve à parte e referencie. Gráficos matplotlib entram como base64.

## 7. Tabela final

Colunas: `nome | orcid | fwci_medio | cited_by_count | works_count | degree | betweenness | closeness | similaridade_usuario`. FWCI e similaridade com 2 casas, centralidades com 4. Ordenar por `similaridade_usuario` decrescente. Heatmap: seaborn, `cmap="Blues"`, `annot=True`, `fmt=".2f"`.

## 8. Entregável e validação

- Nomes: `grafo_similaridade_<tema_snake>_<orcid_curto>.html` / `.ipynb` (orcid_curto = últimos 4 blocos sem hífen).
- `.html`: HTML5 com CSS embutido, tabela, grafo e heatmap incluídos, sem recurso externo obrigatório. Os dados precisam ter sido coletados de verdade (Nível A) ou o código que os coleta e grava o HTML deve ser entregue (Nível B).
- `.ipynb`: nbformat 4, Markdown + código, `compile()` em cada célula de código.
- Nível A: validar JSON/tags balanceadas, presença do grafo e da tabela, e conferir que o usuário aparece no grafo.

## 9. Limitações (documentar no relatório)

- Média de FWCI calculada só sobre os artigos coletados do tema
- Similaridade depende dos tópicos da OpenAlex e pode perder nuances
- Grafo limitado aos `MAX_AUTORES` autores mais frequentes
- A OpenAlex pode ter autores duplicados ou artigos mal atribuídos; ORCID ausente em muitos perfis
- Para decisões críticas, cruzar com Scopus ou Web of Science

## 10. Resposta final (sempre em português)

1. "Arquivo criado com sucesso." (ou, no Nível B: "Código pronto. Cole no Google Colab.")
2. Nome, formato e caminho ou link para download
3. ⚠️ E-mail: "Informe um e-mail real quando o arquivo pedir (ou edite `EMAIL_CONTATO`)."
4. Como abrir: `.html` em qualquer navegador (também dá para enviar por e-mail ou hospedar); `.ipynb` no Jupyter, Colab, VS Code ou nbviewer
5. Três linhas com os achados principais (autores mais similares, mais centrais, mais influentes)
6. Uma linha com o que foi validado
7. Oferta: "Quer ajustar o tema, o corte de similaridade ou o outro formato?"
