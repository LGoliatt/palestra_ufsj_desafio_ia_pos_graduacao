---
name: ppg-geociencias-aderencia
description: Analisa a aderência de um conjunto de ORCIDs à área de Geociências da CAPES, gera métricas agregadas, matriz de similaridade temática e grafo de coautoria com base no OpenAlex, para apoiar a submissão de um APCN de PPG novo.
version: 1.0
author: Análise automatizada
language: pt-BR
data_source: OpenAlex API (https://api.openalex.org)
referencia_normativa: CAPES — Documento Orientador APCN 2023 — Geociências
---

# SKILL — Análise de Aderência para PPG em Geociências

## 1. Objetivo

Dado um conjunto de ORCIDs, esta SKILL:

1. Consulta o **OpenAlex** para recuperar os dados de cada pesquisador.
2. Calcula a **aderência individual** de cada um à área de **Geociências** (conforme documento orientador CAPES/APCN 2023).
3. Produz a **matriz de similaridade temática** entre os pesquisadores.
4. Constrói o **grafo de coautoria** com a força dos vínculos.
5. Calcula **métricas agregadas** que representam a força do PPG a ser criado.
6. Apresenta **métricas adicionais de interesse** para um APCN.
7. Exporta os resultados em **CSV, PNG, IPYNB e HTML**.

---

## 2. Entradas

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `orcid_list` | lista de strings | Sim | Lista de ORCIDs no formato `0000-0000-0000-0000` |
| `area_capes` | string | Sim | Área de avaliação CAPES (aqui fixada em `Geociências`) |
| `max_obras` | inteiro | Não | Máximo de obras por autor (padrão: 200) |
| `mailto` | string | Não | E-mail para o *polite pool* do OpenAlex |

### ORCIDs de entrada (exemplo desta execução)

```
0000-0002-2844-9470   0000-0001-7017-4653   0000-0003-0821-126X
0000-0002-0508-8959   0000-0001-9573-8996   0000-0003-2012-7802
0000-0003-2486-1190   0000-0003-1899-9441   0000-0002-7205-9509
0000-0001-9943-1334   0000-0002-5330-7407   0000-0001-7874-9271
0000-0002-6070-7257   0000-0003-1321-158X   0000-0002-4974-6802
0000-0002-0261-4298   0000-0002-3632-9639
```

---

## 3. Saídas Esperadas

| Arquivo | Conteúdo |
|---|---|
| `aderencia_ppg_geociencias.csv` | Uma linha por ORCID com todas as métricas individuais |
| `matriz_similaridade.csv` | Matriz N×N de similaridade temática (cosseno) |
| `grafo_coautoria.png` | Grafo de coautoria com espessura das arestas = força do vínculo |
| `analise_ppg.ipynb` | Notebook executável com todo o pipeline |
| `analise_ppg.html` | Relatório em HTML navegável |

---

## 4. Pipeline de Execução

### 4.1 Fluxo lógico

```
[ORCIDs]
   ↓
[OpenAlex /authors?filter=orcid:<id>]
   ↓
[OpenAlex /works?filter=author.id:<author_id>]
   ↓
[Extração de topics, concepts, authorships, open_access]
   ↓
[Cálculo de aderência + métricas individuais]
   ↓
[Matriz de similaridade + grafo de coautoria]
   ↓
[Métricas agregadas do PPG]
   ↓
[Exportação CSV / PNG / IPYNB / HTML]
```

### 4.2 Etapas detalhadas

**Etapa 1 — Resolução ORCID → OpenAlex ID**
- Endpoint: `GET /authors?filter=orcid:<ORCID>`
- Retorna: `display_name`, `works_count`, `cited_by_count`, `summary_stats`

**Etapa 2 — Coleta de obras**
- Endpoint: `GET /works?filter=author.id:<ID>&per_page=200&cursor=*`
- Paginação via cursor até esgotar ou atingir `max_obras`

**Etapa 3 — Cálculo de aderência**
- Compara `topics` e `concepts` das obras com o vocabulário de Geociências
- Fórmula: `aderência = (acertos / total_termos) × 100`

**Etapa 4 — Métricas individuais**
- h-index, i10-index, obras totais, citações, obras 2020+, % OA, países, coautores únicos

**Etapa 5 — Similaridade temática**
- Vetor de temas por autor → similaridade de cosseno par a par

**Etapa 6 — Grafo de coautoria**
- Nós = pesquisadores; arestas = coautorias; peso = nº de obras conjuntas

**Etapa 7 — Métricas agregadas do PPG**
- Médias, medianas, totais e índice composto de força

---

## 5. Vocabulário de Geociências (CAPES APCN 2023)

```yaml
Geologia:
  - geology, geological, mineralogy, petrology, sedimentology,
    stratigraphy, tectonics, geochemistry, geochronology,
    paleontology, volcanology

Geofísica:
  - geophysics, geophysical, seismology, geodesy, geodetic,
    geomagnetism, gravimetry

Geografia Física / Sensoriamento:
  - geomorphology, remote sensing, geoprocessing, gis,
    geoinformatics, geospatial, cartography

Atmosfera / Oceano / Clima:
  - meteorology, climatology, atmospheric, oceanography,
    oceanographic, hydrology, hydrogeology

Terra / Ambiente:
  - earth science, environmental geology, natural hazard,
    geoconservation, geotourism
```

---

## 6. Métricas Produzidas

### 6.1 Métricas individuais

| Métrica | Descrição | Fonte OpenAlex |
|---|---|---|
| `aderencia_geociencias` | % de temas alinhados à área | topics + concepts |
| `obras_totais` | Total de obras do autor | `works_count` |
| `citacoes_totais` | Total de citações | `cited_by_count` |
| `h_index` | Índice h | `summary_stats.h_index` |
| `i10_index` | Índice i10 | `summary_stats.i10_index` |
| `obras_2020_mais` | Produção recente (2020+) | works |
| `pct_acesso_aberto` | % de obras em OA | `open_access.is_oa` |
| `media_citacoes_por_obra` | Média de citações | works |
| `numero_paises` | Países distintos nas coautorias | institutions |
| `tem_colaboracao_internacional` | 0/1 se há coautoria internacional | institutions |
| `coautores_unicos` | Nº de coautores distintos | authorships |

### 6.2 Métricas agregadas do PPG

| Métrica | Descrição |
|---|---|
| Docentes vinculados ao OpenAlex | Cobertura da base |
| Aderência média / mediana | Alinhamento do grupo à área |
| h-index médio e máximo | Qualidade consolidada |
| Total de obras e citações | Volume do grupo |
| Obras recentes (2020+) | Vitalidade científica |
| Docentes com internacionalização | Inserção global |
| Países distintos cobertos | Diversidade de redes |
| % média de acesso aberto | Visibilidade |
| **Índice composto de força** | Aderência 40% + h-index 25% + produção recente 20% + OA 15% |

### 6.3 Métricas adicionais recomendadas para APCN

| Métrica | Relevância CAPES | Fonte sugerida |
|---|---|---|
| Regularidade de produção (obras/ano) | Exigência de continuidade | OpenAlex |
| Artigos em estratos A1–A4 (Qualis) | Peso direto na avaliação | Qualis CAPES |
| Orientações concluídas | Experiência formativa | Sucupira / Lattes |
| Projetos financiados | Captação de recursos | Lattes / CV |
| Bolsas PQ / produtividade | Qualidade individual | Sucupira |
| Redes nacionais de cooperação | Inserção científica | OpenAlex |
| Cobertura geográfica dos estudos | Relevância regional | abstracts |
| Aderência ao PDI institucional | Requisito APCN | documento institucional |
| Infraestrutura laboratorial | Critério eliminatório | formulário APCN |
| Potencial de inserção social | Avaliação multidimensional | análise qualitativa |

---

## 7. Estruturas de Saída

### 7.1 Tabela individual (schema)

| orcid | nome | aderencia_geociencias | h_index | obras_totais | citacoes_totais | obras_2020_mais | pct_acesso_aberto | numero_paises | forca_ppg |
|---|---|---|---|---|---|---|---|---|---|
| 0000-... | Nome A | — | — | — | — | — | — | — | — |
| 0000-... | Nome B | — | — | — | — | — | — | — | — |

> Os valores são preenchidos após a execução do script contra a API do OpenAlex.

### 7.2 Matriz de similaridade (schema)

```
                 Autor A   Autor B   Autor C  ...
Autor A           1.000     0.000     0.000
Autor B           0.000     1.000     0.000
Autor C           0.000     0.000     1.000
...
```

### 7.3 Grafo de coautoria

- **Nós:** pesquisadores (tamanho ∝ grau de conexão)
- **Arestas:** coautorias (espessura ∝ nº de obras conjuntas)
- **Métricas:** densidade, grau, centralidade, top-10 colaborações

---

## 8. Código Executável (Python)

```python
# ============================================================
# ANÁLISE DE ADERÊNCIA PARA PPG EM GEOCIÊNCIAS
# Fonte: OpenAlex API
# ============================================================

import requests
import pandas as pd
import numpy as np
import networkx as nx
import matplotlib.pyplot as plt
from collections import Counter
import time
import warnings
warnings.filterwarnings('ignore')

# ------------------------------------------------------------
# 1. Configuração
# ------------------------------------------------------------
ORCIDS = [
    "0000-0002-2844-9470", "0000-0001-7017-4653", "0000-0003-0821-126X",
    "0000-0002-0508-8959", "0000-0001-9573-8996", "0000-0003-2012-7802",
    "0000-0003-2486-1190", "0000-0003-1899-9441", "0000-0002-7205-9509",
    "0000-0001-9943-1334", "0000-0002-5330-7407", "0000-0001-7874-9271",
    "0000-0002-6070-7257", "0000-0003-1321-158X", "0000-0002-4974-6802",
    "0000-0002-0261-4298", "0000-0002-3632-9639"
]

PALAVRAS_GEOCIENCIAS = [
    "geology", "geological", "mineralogy", "petrology", "sedimentology",
    "stratigraphy", "tectonics", "geochemistry", "geochronology",
    "paleontology", "volcanology",
    "geophysics", "geophysical", "seismology", "geodesy", "geodetic",
    "geomagnetism", "gravimetry",
    "geomorphology", "remote sensing", "geoprocessing", "gis",
    "geoinformatics", "geospatial", "cartography",
    "meteorology", "climatology", "atmospheric", "oceanography",
    "oceanographic", "hydrology", "hydrogeology",
    "earth science", "environmental geology", "natural hazard",
    "geoconservation", "geotourism"
]

HEADERS = {'User-Agent': 'PPG-Geociencias-Analysis/1.0 (mailto:seu@email.com)'}
BASE_URL = "https://api.openalex.org"

# ------------------------------------------------------------
# 2. Funções de coleta
# ------------------------------------------------------------
def buscar_autor_por_orcid(orcid):
    url = f"{BASE_URL}/authors"
    params = {"filter": f"orcid:{orcid}"}
    try:
        r = requests.get(url, params=params, headers=HEADERS, timeout=30)
        r.raise_for_status()
        dados = r.json()
        if dados.get("results"):
            return dados["results"][0]
    except Exception as e:
        print(f"  [ERRO] {orcid}: {e}")
    return None


def buscar_obras_do_autor(author_id, max_obras=200):
    obras = []
    cursor = "*"
    while len(obras) < max_obras:
        url = f"{BASE_URL}/works"
        params = {
            "filter": f"author.id:{author_id}",
            "per_page": 200,
            "cursor": cursor
        }
        try:
            r = requests.get(url, params=params, headers=HEADERS, timeout=60)
            r.raise_for_status()
            dados = r.json()
            obras.extend(dados.get("results", []))
            cursor = dados.get("meta", {}).get("next_cursor")
            if not cursor:
                break
            time.sleep(0.2)
        except Exception as e:
            print(f"    [ERRO obras] {e}")
            break
    return obras[:max_obras]


# ------------------------------------------------------------
# 3. Métricas
# ------------------------------------------------------------
def calcular_aderencia_geociencias(obras):
    if not obras:
        return 0.0, []
    temas, conceitos = [], []
    for o in obras:
        for t in o.get("topics", []):
            temas.append(t.get("display_name", "").lower())
        for c in o.get("concepts", []):
            conceitos.append(c.get("display_name", "").lower())
    acertos = 0
    termos_encontrados = Counter()
    for texto in temas + conceitos:
        for kw in PALAVRAS_GEOCIENCIAS:
            if kw in texto:
                acertos += 1
                termos_encontrados[kw] += 1
                break
    total = len(temas) + len(conceitos)
    score = (acertos / total * 100) if total > 0 else 0.0
    return round(score, 1), termos_encontrados.most_common(10)


def calcular_metricas_individuais(autor, obras):
    if not autor:
        return None
    anos = [o.get("publication_year") for o in obras if o.get("publication_year")]
    recentes = sum(1 for a in anos if a and a >= 2020)
    acesso_aberto = sum(1 for o in obras if o.get("open_access", {}).get("is_oa"))
    media_cit = np.mean([o.get("cited_by_count", 0) for o in obras]) if obras else 0
    paises, coautores = set(), set()
    for o in obras:
        for a in o.get("authorships", []):
            for inst in a.get("institutions", []):
                if inst.get("country_code"):
                    paises.add(inst["country_code"])
            cid = a.get("author", {}).get("id", "")
            if cid:
                coautores.add(cid)
    return {
        "obras_totais": autor.get("works_count", 0),
        "citacoes_totais": autor.get("cited_by_count", 0),
        "h_index": autor.get("summary_stats", {}).get("h_index", 0),
        "i10_index": autor.get("summary_stats", {}).get("i10_index", 0),
        "obras_2020_mais": recentes,
        "pct_acesso_aberto": round(acesso_aberto / len(obras) * 100, 1) if obras else 0,
        "media_citacoes_por_obra": round(media_cit, 1),
        "numero_paises": len(paises),
        "tem_colaboracao_internacional": int(len(paises) > 1),
        "obras_recuperadas": len(obras),
        "coautores_unicos": len(coautores)
    }


# ------------------------------------------------------------
# 4. Execução principal
# ------------------------------------------------------------
print("=" * 70)
print("ANÁLISE DE ADERÊNCIA — PPG EM GEOCIÊNCIAS")
print("=" * 70)

resultados = []
mapa_obras = {}

for i, orcid in enumerate(ORCIDS, 1):
    print(f"\n[{i}/{len(ORCIDS)}] Processando ORCID: {orcid}")
    autor = buscar_autor_por_orcid(orcid)
    if not autor:
        print("  ✗ Autor não encontrado no OpenAlex")
        resultados.append({
            "orcid": orcid, "nome": "N/A", "encontrado": False,
            "aderencia_geociencias": 0, "h_index": 0, "obras_totais": 0
        })
        continue
    author_id = autor["id"].replace("https://openalex.org/", "")
    nome = autor.get("display_name", "Desconhecido")
    print(f"  ✓ Encontrado: {nome}")
    obras = buscar_obras_do_autor(author_id, max_obras=200)
    mapa_obras[author_id] = obras
    print(f"    Obras recuperadas: {len(obras)}")
    aderencia, termos_top = calcular_aderencia_geociencias(obras)
    print(f"    Aderência Geociências: {aderencia}%")
    if termos_top:
        print(f"    Termos predominantes: {', '.join([f'{t}({c})' for t, c in termos_top[:5]])}")
    metricas = calcular_metricas_individuais(autor, obras)
    if metricas:
        resultados.append({
            "orcid": orcid, "nome": nome, "openalex_id": author_id,
            "encontrado": True, "aderencia_geociencias": aderencia, **metricas
        })
    time.sleep(0.3)

# ------------------------------------------------------------
# 5. Tabela consolidada
# ------------------------------------------------------------
df = pd.DataFrame(resultados)
print("\n" + "=" * 70)
print("RESUMO INDIVIDUAL")
print("=" * 70)
colunas = ["nome", "aderencia_geociencias", "h_index", "obras_totais",
           "citacoes_totais", "obras_2020_mais"]
print(df[[c for c in colunas if c in df.columns]].to_string(index=False))

# ------------------------------------------------------------
# 6. Métricas agregadas do grupo
# ------------------------------------------------------------
encontrados = df[df.get("encontrado") == True] if "encontrado" in df.columns else pd.DataFrame()
if len(encontrados) > 0:
    print("\n" + "=" * 70)
    print("MÉTRICAS AGREGADAS — FORÇA DO PPG PROPOSTO")
    print("=" * 70)
    print(f"Docentes vinculados ao OpenAlex........: {len(encontrados)}/{len(ORCIDS)}")
    print(f"Aderência média a Geociências.........: {encontrados['aderencia_geociencias'].mean():.1f}%")
    print(f"Mediana de aderência..................: {encontrados['aderencia_geociencias'].median():.1f}%")
    print(f"h-index médio do grupo................: {encontrados['h_index'].mean():.1f}")
    print(f"h-index máximo........................: {encontrados['h_index'].max()}")
    print(f"Total de obras (grupo)................: {encontrados['obras_totais'].sum()}")
    print(f"Total de citações (grupo).............: {encontrados['citacoes_totais'].sum()}")
    print(f"Obras recentes 2020+ (grupo)..........: {encontrados['obras_2020_mais'].sum()}")
    print(f"Docentes com internacionalização......: {encontrados['tem_colaboracao_internacional'].sum()}")
    print(f"Países distintos cobertos.............: {encontrados['numero_paises'].sum()}")
    print(f"% média de acesso aberto..............: {encontrados['pct_acesso_aberto'].mean():.1f}%")

    encontrados = encontrados.copy()
    encontrados["forca_ppg"] = (
        encontrados["aderencia_geociencias"].rank(pct=True) * 0.40 +
        encontrados["h_index"].rank(pct=True) * 0.25 +
        encontrados["obras_2020_mais"].rank(pct=True) * 0.20 +
        encontrados["pct_acesso_aberto"].rank(pct=True) * 0.15
    ) * 100
    print("\n--- ÍNDICE COMPOSTO DE FORÇA INDIVIDUAL (0-100) ---")
    print("Pesos: aderência 40% | h-index 25% | produção recente 20% | OA 15%")
    print(encontrados[["nome", "forca_ppg"]].sort_values("forca_ppg", ascending=False).to_string(index=False))

# ------------------------------------------------------------
# 7. Matriz de similaridade temática
# ------------------------------------------------------------
def vetor_temas(obras):
    v = Counter()
    for o in obras:
        for t in o.get("topics", []):
            v[t.get("display_name", "")] += 1
    return v


def similaridade_cosseno(v1, v2):
    chaves = set(v1) | set(v2)
    dot = sum(v1.get(k, 0) * v2.get(k, 0) for k in chaves)
    n1 = np.sqrt(sum(v1.get(k, 0) ** 2 for k in chaves))
    n2 = np.sqrt(sum(v2.get(k, 0) ** 2 for k in chaves))
    return dot / (n1 * n2) if n1 and n2 else 0.0


print("\n" + "=" * 70)
print("MATRIZ DE SIMILARIDADE TEMÁTICA")
print("=" * 70)

autores_validos = [(r["nome"], r["openalex_id"]) for r in resultados if r.get("encontrado")]
n = len(autores_validos)
matriz = np.zeros((n, n))
for i in range(n):
    for j in range(n):
        if i == j:
            matriz[i][j] = 1.0
        else:
            vi = vetor_temas(mapa_obras.get(autores_validos[i][1], []))
            vj = vetor_temas(mapa_obras.get(autores_validos[j][1], []))
            matriz[i][j] = round(similaridade_cosseno(vi, vj), 3)

df_sim = pd.DataFrame(matriz,
                       index=[a[0][:20] for a in autores_validos],
                       columns=[a[0][:20] for a in autores_validos])
print(df_sim.to_string())

# ------------------------------------------------------------
# 8. Grafo de coautoria
# ------------------------------------------------------------
print("\n" + "=" * 70)
print("GRAFO DE COAUTORIA (força dos vínculos)")
print("=" * 70)

G = nx.Graph()
nomes = {a[1]: a[0] for a in autores_validos}
for aid in mapa_obras:
    if aid in nomes:
        G.add_node(aid, nome=nomes[aid])

for aid, obras in mapa_obras.items():
    if aid not in nomes:
        continue
    for o in obras:
        coauts = set()
        for a in o.get("authorships", []):
            cid = a.get("author", {}).get("id", "").replace("https://openalex.org/", "")
            if cid and cid != aid and cid in nomes:
                coauts.add(cid)
        for c in coauts:
            if G.has_edge(aid, c):
                G[aid][c]["peso"] += 1
            else:
                G.add_edge(aid, c, peso=1)

print(f"Nós (pesquisadores).....: {G.number_of_nodes()}")
print(f"Arestas (coautorias)....: {G.number_of_edges()}")
print(f"Densidade da rede.......: {nx.density(G):.4f}")

if G.number_of_edges() > 0:
    print("\n--- TOP 10 COLABORAÇÕES ---")
    arestas = sorted(G.edges(data=True), key=lambda x: x[2].get("peso", 0), reverse=True)
    for u, v, d in arestas[:10]:
        print(f"  {nomes[u]} ↔ {nomes[v]}: {d['peso']} obras conjuntas")
    print("\n--- CENTRALIDADE (grau) ---")
    graus = sorted(G.degree(), key=lambda x: x[1], reverse=True)
    for nid, g in graus:
        print(f"  {nomes[nid]}: {g} conexões")

# ------------------------------------------------------------
# 9. Exportação
# ------------------------------------------------------------
df.to_csv("aderencia_ppg_geociencias.csv", index=False)
df_sim.to_csv("matriz_similaridade.csv")
print("\n✓ Exportado: aderencia_ppg_geociencias.csv")
print("✓ Exportado: matriz_similaridade.csv")

# ------------------------------------------------------------
# 10. Visualização
# ------------------------------------------------------------
if G.number_of_edges() > 0:
    plt.figure(figsize=(14, 10))
    pos = nx.spring_layout(G, k=2, iterations=50, seed=42)
    tamanhos = [G.degree(n_) * 300 + 500 for n_ in G.nodes()]
    larguras = [G[u][v].get("peso", 1) * 1.5 for u, v in G.edges()]
    nx.draw_networkx_nodes(G, pos, node_size=tamanhos, node_color='steelblue', alpha=0.85)
    nx.draw_networkx_edges(G, pos, width=larguras, alpha=0.5, edge_color='gray')
    nx.draw_networkx_labels(G, pos,
                            labels={n_: nomes[n_][:15] for n_ in G.nodes()},
                            font_size=8)
    plt.title("Rede de Coautoria — PPG em Geociências (OpenAlex)", fontsize=14)
    plt.axis('off')
    plt.tight_layout()
    plt.savefig("grafo_coautoria.png", dpi=150)
    print("✓ Grafo exportado: grafo_coautoria.png")

print("\n" + "=" * 70)
print("ANÁLISE CONCLUÍDA")
print("=" * 70)
```

### 8.1 Instalação de dependências

```bash
pip install requests pandas numpy networkx matplotlib scipy jupytext jupyter
```

### 8.2 Execução

```bash
python analise_ppg.py
```

### 8.3 Geração do `.ipynb` e `.html`

```bash
jupytext --to notebook analise_ppg.py
jupyter nbconvert --to notebook --execute analise_ppg.ipynb
jupyter nbconvert --to html analise_ppg.ipynb
```

---

## 9. Limitações e Boas Práticas

1. **Chave de API OpenAlex:** sem chave, o *rate limit* é menor. Registre-se em `openalex.org/settings/api` e use `mailto` ou `api_key`.
2. **Cobertura ORCID:** nem todos os ORCIDs estão vinculados ao OpenAlex; autores ausentes aparecerão como `encontrado: false`.
3. **Aderência automática:** baseada em `topics`/`concepts` do OpenAlex — não substitui análise qualitativa do Lattes.
4. **CAPES não usa OpenAlex:** usa Qualis, Sucupira e produção declarada. Esta SKILL é **pré-análise**, não resultado oficial.
5. **Validação humana obrigatória:** conferir amostra dos resultados antes de usar em decisão estratégica.
6. **Atualização:** reexecutar periodicamente, pois o OpenAlex é atualizado continuamente.

---

## 10. Checklist de Uso

- [ ] Lista de ORCIDs validada (formato `0000-0000-0000-0000`)
- [ ] Chave de API OpenAlex configurada
- [ ] Dependências Python instaladas
- [ ] Script executado sem erros
- [ ] `aderencia_ppg_geociencias.csv` gerado
- [ ] `matriz_similaridade.csv` gerado
- [ ] `grafo_coautoria.png` gerado
- [ ] `analise_ppg.ipynb` e `analise_ppg.html` exportados
- [ ] Amostra validada manualmente (Lattes/instituição)
- [ ] Métricas complementares (Qualis, Sucupira) integradas
- [ ] Relatório final consolidado para o APCN

---

## 11. Glossário

| Termo | Definição |
|---|---|
| **APCN** | Proposta de Criação de Curso Novo (CAPES) |
| **PPG** | Programa de Pós-Graduação |
| **ORCID** | Identificador digital persistente de pesquisador |
| **OpenAlex** | Base bibliográfica aberta mantida pela OurResearch |
| **Aderência** | Grau de alinhamento temático com a área CAPES |
| **Qualis** | Sistema de classificação de periódicos da CAPES |
| **Sucupira** | Plataforma de coleta de dados da pós-graduação brasileira |
| **PDI** | Plano de Desenvolvimento Institucional |
| **OA** | Open Access (acesso aberto) |

---

## 12. Referências

- CAPES — Documento Orientador APCN 2023 — Geociências (Colégio de Ciências Exatas, Tecnológicas e Multidisciplinar)
- OpenAlex API — https://docs.openalex.org
- OurResearch — https://ourresearch.org
- ORCID — https://orcid.org

---

## 13. Contato e Manutenção

Esta SKILL é genérica e reutilizável. Para adaptar a outras áreas CAPES:

1. Substituir o bloco `PALAVRAS_GEOCIENCIAS` pelo vocabulário da nova área.
2. Ajustar os pesos do `forca_ppg` conforme prioridades da área.
3. Atualizar a referência normativa na seção 12.

**Versão:** 1.0
**Última atualização:** conforme execução
**Idioma:** Português (Brasil)