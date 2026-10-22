---
name: openalex-analise-publicacoes
description: Conta publicações por ano (ou outro campo) no OpenAlex para um tema e gera gráfico + tabela em .html ou .ipynb. Use quando o usuário pedir análise bibliométrica, evolução de publicações de um tema, ou chamar a skill sem argumentos.
---

# SKILL: Evolução de publicações por tema (OpenAlex)

Saída: `.html` (abre em qualquer navegador) ou `.ipynb` (Jupyter, Colab, VS Code).
Todo o texto ao usuário e dentro dos arquivos deve estar em português (pt-BR).

## 0. Compatibilidade (leia primeiro)

Esta skill deve funcionar em qualquer IA. Antes de responder, descubra o que você consegue fazer:

- **Nível A, executa código e cria arquivos:** gere o arquivo, valide, entregue o link/caminho.
- **Nível B, não executa código nem cria arquivos:** entregue o código completo em UM único bloco ```python``` (sem fragmentar) e diga ao usuário para colar no Google Colab (colab.research.google.com) e executar. Se ele escolheu `.html`, o código deve gravar o arquivo `.html` ao final. Se escolheu `.ipynb`, entregue o código dividido nas seções abaixo, identificadas como "Célula 1, Célula 2...", para colar célula a célula.

Nunca diga que "gerou o arquivo" se você não conseguiu criá-lo de fato.

## 1. Se o usuário não digitou nada (ou só "oi", "olá", "teste", "ajuda")

Responda SOMENTE com o bloco abaixo, exatamente assim, sem código e sem explicação extra:

````
📊 ANÁLISE DE PUBLICAÇÕES POR TEMA (OpenAlex)

Esta ferramenta conta quantos artigos existem sobre um tema,
ano a ano, e entrega tabela + gráfico prontos para usar.

COMO USAR (escolha um jeito)

1) Mais rápido: copie e envie este exemplo
   Gere uma busca com o tema "symbolic regression hydrology"
   a partir de 01/01/2018 em html

2) Com o seu tema: troque só o que está entre aspas
   Gere uma busca com o tema "SEU TEMA AQUI"
   a partir de AAAA-MM-DD em html

3) Sem pensar: envie apenas a palavra: padrão
   (uso o tema "symbolic regression hydrology", desde 2018)

FORMATO DE SAÍDA (você escolhe)
   html   abre em qualquer navegador, fácil de enviar por e-mail
   ipynb  caderno editável no Jupyter / Google Colab / VS Code
   Não sabe qual? Escolha html.

O QUE VOCÊ PRECISA TER
   Um e-mail real (a OpenAlex pede, é grátis e sem cadastro).
   Eu pergunto na hora de gerar. Não invento e-mail.

PARÂMETROS (opcional, para quem quer controle total)
   TERMO_BUSCA       = "symbolic regression hydrology"
   DATA_INICIAL      = "2018-01-01"
   CAMPO_AGRUPAMENTO = "publication_year"
   FORMATO_SAIDA     = "html"        # ou "ipynb"
   EMAIL_CONTATO     = "seu-email@exemplo.com"

   O que significa cada um:
   TERMO_BUSCA       assunto pesquisado nos artigos (pode ser em inglês
                     ou português; inglês costuma dar mais resultados)
   DATA_INICIAL      só conta artigos publicados desde esta data
   CAMPO_AGRUPAMENTO como agrupar: publication_year (ano), type (tipo
                     de documento), open_access.is_oa (acesso aberto)
   FORMATO_SAIDA     html ou ipynb
   EMAIL_CONTATO     entra no parâmetro mailto da API (acesso mais estável)
````

Depois do bloco, termine com UMA pergunta curta: "Quer o exemplo padrão? Responda com o formato: html ou ipynb."

## 2. Se o usuário deu algo (fluxo mínimo de perguntas)

Regras para não cansar o usuário:

1. Extraia da mensagem: tema, data inicial, campo de agrupamento, formato, e-mail.
2. Use os padrões para o que faltar: data `2018-01-01`, agrupamento `publication_year`. Tema padrão só se ele disser "padrão" ou "tanto faz".
3. Se faltar **formato** ou **e-mail**, faça UMA única pergunta juntando os dois. Exemplo: "Qual formato (html ou ipynb) e qual e-mail uso na API? Se preferir, o arquivo pergunta o e-mail quando você executá-lo."
4. Se o usuário não quiser informar e-mail, não bloqueie. Gere o arquivo com `EMAIL_CONTATO = ""` e faça o código pedir o e-mail na execução (ver seção 4).
5. Se o tema vier em português, sugira em uma linha a versão em inglês, mas rode o que ele pediu.

## 3. Parâmetros

```python
API_URL            = "https://api.openalex.org/works"
EMAIL_CONTATO      = ""                              # vazio = o código pergunta ao executar
TERMO_BUSCA        = "symbolic regression hydrology"
DATA_INICIAL       = "2018-01-01"
CAMPO_AGRUPAMENTO  = "publication_year"
ARQUIVO_GRAFICO    = "publicacoes_<tema_snake_case>.png"
```

Se o tema mudar, adapte títulos, textos e nomes de arquivo (snake_case, sem acentos).

## 4. Requisitos do código gerado

**Chamada da API**
- `GET https://api.openalex.org/works` com `search`, `filter=from_publication_date:{DATA_INICIAL}`, `group_by`, `mailto`.
- `requests.get(..., timeout=30)` e `raise_for_status()`.
- Usar `group_by` nativo. Não baixar artigos um a um.
- Resposta vazia: mensagem clara em português, sem traceback.

**E-mail**
- Se `EMAIL_CONTATO` estiver vazio ou for o placeholder `seu-email@exemplo.com`, pedir com `input("Digite seu e-mail para a API OpenAlex: ")`. Em ambiente sem `input` (execução automática), seguir sem `mailto` e avisar que o acesso pode ficar mais lento.
- Validar o formato com uma checagem simples (`"@" in email and "." in email`).

**Função obrigatória**

```python
def agrupar_publicacoes(campo_agrupamento, termo_busca, data_inicial, email, timeout=30) -> pd.DataFrame:
    """Retorna DataFrame com colunas: grupo, quantidade."""
```
- Docstring com objetivo, parâmetros, retorno e exceções.
- Se `campo_agrupamento == "publication_year"`: converter para numérico, remover inválidos, ordenar crescente, converter para int.
- Resultado: `publicacoes_por_ano`. Imprimir "Quantidade de publicações por ano:" e a tabela.
- A API devolve `group_by` como lista de itens com `key`, `key_display_name` e `count`.

**Gráfico (matplotlib + seaborn)**
- paleta `Blues_d`, `figsize=(12, 7)`, anos em ordem no eixo X, valor no topo de cada barra, milhar com ponto (1.234).
- Título: `Publicações científicas sobre "<termo>" a partir de <data_inicial>`.
- Eixos: "Ano de publicação" e "Quantidade de publicações". Grade suave só no eixo Y.
- `plt.tight_layout()`, salvar PNG a 300 DPI, `plt.show()`.

**Dependências:** `requests pandas matplotlib seaborn`. Python 3.9+. Sem tokens, sem credenciais.

## 5. Estrutura do entregável (igual para html e ipynb)

1. Título e descrição  2. Objetivos  3. Instalação  4. Importações  5. Configuração da API  6. Função de consulta  7. Coleta  8. Tabela  9. Gráfico  10. Metodologia e limitações

- `.ipynb`: nbformat 4, cada seção com uma célula Markdown e uma ou mais de código. Primeira célula de código: `%pip install -q requests pandas matplotlib seaborn`.
- `.html`: HTML5 com CSS embutido, uma `<section>` por item, tabela em HTML e gráfico como `<img src="data:image/png;base64,...">`. Nenhum recurso externo. Para ser autocontido, o HTML precisa dos dados já coletados: se você não consegue rodar a consulta (Nível B), entregue o script Python que grava o HTML.
- Nomes: `analise_openalex_<tema_snake_case>.ipynb` / `.html`.

**Limitações a documentar:** a busca é textual e pode trazer artigos fora do tema; o ano mais recente pode estar incompleto; a OpenAlex pode ter duplicatas e metadados faltando; para decisões críticas, cruzar com Scopus ou Web of Science.

## 6. Validação (Nível A)

- `.ipynb`: JSON válido, `nbformat` definido, células Markdown e de código, `compile()` em cada célula de código sem erro de sintaxe.
- `.html`: tags balanceadas, imagem base64 presente, nenhum `src`/`href` externo.
- Palavras-chave presentes: OpenAlex, mailto, publication_year, o termo de busca, seaborn, matplotlib, Blues_d.

## 7. Resposta final (sempre em português)

1. "Arquivo criado com sucesso." (ou, no Nível B: "Código pronto. Cole no Google Colab.")
2. Nome do arquivo e formato
3. Caminho ou link para download
4. ⚠️ E-mail: "Informe um e-mail real quando o arquivo pedir (ou edite `EMAIL_CONTATO`)."
5. Como abrir: `.html` em qualquer navegador (também dá para enviar por e-mail ou hospedar no GitHub Pages); `.ipynb` no Jupyter, Colab ou VS Code
6. Uma linha com o que foi validado
7. Oferta: "Quer outro tema, outra data ou o outro formato?"
