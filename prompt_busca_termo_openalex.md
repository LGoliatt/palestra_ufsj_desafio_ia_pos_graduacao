======================================================================
TEMA PADRÃO (personalizável pelo usuário)
======================================================================
Termo de busca: "symbolic regression hydrology"
Data inicial: 2018-01-01
Agrupamento: publication_year
Fonte: OpenAlex (https://api.openalex.org/works)

Se o usuário trocar o tema, adapte TODOS os textos, títulos de gráfico e
nomes de arquivo proporcionalmente (mantendo o padrão snake_case em português).

======================================================================
ENTREGÁVEIS OBRIGATÓRIOS
======================================================================
1. Arquivo físico real:
- Se `.ipynb`: JSON válido no formato nbformat 4.
  Nome padrão: analise_openalex_<tema_snake_case>.ipynb
- Se `.html`: HTML5 autocontido, com CSS embutido, gráfico em base64
  (PNG embutido) OU com Plotly via CDN.
  Nome padrão: analise_openalex_<tema_snake_case>.html
2. Validação programática do arquivo gerado.
3. Disponibilização para download (link `data:` ou caminho de arquivo).
4. Informe o caminho absoluto do arquivo.
5. Aviso destacado: o usuário DEVE substituir EMAIL_CONTATO antes de executar.

======================================================================
ESTRUTURA OBRIGATÓRIA (mesma para .ipynb e .html)
======================================================================
1.  Título e descrição da análise
2.  Objetivos
3.  Instalação das dependências
4.  Importação das bibliotecas
5.  Configuração da API
6.  Função de consulta e agrupamento
7.  Coleta dos dados
8.  Exibição da tabela de resultados
9.  Criação do gráfico
10. Observações sobre metodologia e limitações

Em `.ipynb`: cada seção = célula Markdown + célula(s) de código.
Em `.html`: cada seção = `<section>` com `<h2>` + bloco `<pre><code>` (se
mostrar código) e/ou resultado renderizado (tabela HTML + gráfico base64).

Todo o texto deve estar em PORTUGUÊS (pt-BR).

======================================================================
DEPENDÊNCIAS
======================================================================
- requests
- pandas
- matplotlib
- seaborn
- (opcional) nbformat, nbclient para validação do .ipynb

Célula inicial opcional (no .ipynb):
 %pip install -q requests pandas matplotlib seaborn

No .html, embuta o gráfico como base64 para não depender de rede.

======================================================================
CONFIGURAÇÃO DA API (variáveis exatas)
======================================================================
explique o texto abaixo para o usuario caso euel não digite nada emostro o seguitne: Modo de usar: gere yuma busca com o tema "symbolic regression hydrology" a partir de "01/01/2018"


API_URL            = "https://api.openalex.org/works"
EMAIL_CONTATO      = "seu-email@exemplo.com"   # PLACEHOLDER — substituir
TERMO_BUSCA        = "symbolic regression hydrology"
DATA_INICIAL       = "2018-01-01"
CAMPO_AGRUPAMENTO  = "publication_year"
ARQUIVO_GRAFICO    = "publicacoes_<tema_snake_case>.png"

Use SEMPRE o parâmetro `mailto` na requisição para acessar o polite pool.
NÃO invente e-mail real. Deixe o placeholder e explique isso no notebook.

======================================================================
REQUISIÇÃO À API
======================================================================
- Endpoint: https://api.openalex.org/works
- Parâmetros:
 search  = TERMO_BUSCA
 filter  = f"from_publication_date:{DATA_INICIAL}"
 group_by = CAMPO_AGRUPAMENTO
 mailto  = EMAIL_CONTATO
- requests.get(..., timeout=30)
- .raise_for_status() para capturar erros HTTP
- Converter para JSON
- Extrair dados de `group_by`
- Tratar respostas vazias com mensagem clara
- Não baixar publicações individualmente (usar group_by nativo)

======================================================================
FUNÇÃO OBRIGATÓRIA: agrupar_publicacoes()
======================================================================
Assinatura mínima:
 def agrupar_publicacoes(campo_agrupamento, termo_busca,
                         data_inicial, email, timeout=30) -> pd.DataFrame

Retorno: DataFrame com colunas:
 - grupo       (str ou int conforme o campo)
 - quantidade  (int)

Comportamento:
- Reutilizável para outros campos de agrupamento do OpenAlex
- Se campo == "publication_year":
 * converter grupo para numérico
 * remover valores inválidos
 * ordenar cronologicamente crescente
 * converter para inteiro quando possível
- Docstring completa: objetivo, parâmetros, retorno, exceções.

======================================================================
EXIBIÇÃO
======================================================================
Variável obrigatória: publicacoes_por_ano
Imprimir:
 "Quantidade de publicações por ano:"
e em seguida o DataFrame de forma legível.
Se vazio, imprimir mensagem clara (não lançar erro confuso).

======================================================================
GRÁFICO (matplotlib + seaborn)
======================================================================
Requisitos:
1.  Paleta "Blues_d"
2.  Anos no eixo X em ordem cronológica
3.  Quantidade no eixo Y
4.  Valor exato no topo de cada barra
5.  Separador de milhares (ex.: 1.234)
6.  Título:
     Publicações científicas sobre "<termo>"
     a partir de <data_inicial>
7.  Rótulos:
     X: "Ano de publicação"
     Y: "Quantidade de publicações"
8.  Grade suave apenas no eixo Y
9.  Sem grade vertical
10. Rotacionar rótulos do X se necessário
11. figsize=(12, 7)
12. plt.tight_layout()
13. Salvar PNG em 300 DPI: ARQUIVO_GRAFICO
14. plt.show()

No .html, embutir o PNG como base64 (<img src="data:image/png;base64,...">).

======================================================================
ROBUSTEZ E BOAS PRÁTICAS
======================================================================
- Funções bem organizadas e comentadas
- Nomes de variáveis em português ou claramente compreensíveis
- Timeout configurado
- Tratamento de erros HTTP e respostas vazias
- Verificação se EMAIL_CONTATO ainda é o placeholder ANTES da consulta
(emitir aviso, não travar)
- Compatível com Python 3.9+
- Sem arquivos externos, sem tokens, sem credenciais reais
- Executável célula por célula (no .ipynb)

======================================================================
VALIDAÇÃO PROGRAMÁTICA (após gerar o arquivo)
======================================================================
Para .ipynb, verifique:
- arquivo existe
- JSON válido
- "nbformat" definido
- há células Markdown
- há células de código
- palavras-chave presentes no conteúdo:
 OpenAlex, mailto, publication_year,
 "symbolic regression hydrology", seaborn,
 matplotlib, agrupamento, Blues_d
- (se possível) compilar cada célula de código com `compile()` para
garantir ausência de erros de sintaxe

Para .html, verifique:
- arquivo existe
- HTML bem formado (tags balanceadas)
- presença das mesmas palavras-chave
- gráfico embutido (base64 ou referência Plotly)
- CSS embutido (nenhum recurso externo obrigatório)

======================================================================
FORMATO DA RESPOSTA FINAL (sempre em português)
======================================================================
1.  Confirmação: "Arquivo criado com sucesso."
2.  Nome do arquivo.
3.  Caminho absoluto.
4.  Link para download (data: URI ou caminho clicável).
5.  Aviso destacado:
     ⚠️ Substitua EMAIL_CONTATO = "seu-email@exemplo.com"
     por um e-mail válido antes de executar.
6.  Resumo do que foi validado.

======================================================================
COMO COMPARTILHAR
======================================================================
- .ipynb → abrir no Jupyter, JupyterLab, Google Colab, VS Code ou Binder.
- .html  → abrir em qualquer navegador, enviar por e-mail, hospedar em
        GitHub Pages, Netlify, etc. Já é autocontido.
- Sempre informe ao usuário final qual formato está recebendo e como usá-lo.


==================================================
Como funciona
==================================================
explqie em portugues por favor
Se você não digitar nada, eu mostro e explico o seguinte bloco padrão, que serve como ponto de partida:

Modo de usar: gere uma busca com o tema "symbolic regression hydrology" a partir de "01/01/2018".

text
API_URL            = "https://api.openalex.org/works"
EMAIL_CONTATO      = "seu-email@exemplo.com"   # PLACEHOLDER — substituir
TERMO_BUSCA        = "symbolic regression hydrology"
DATA_INICIAL       = "2018-01-01"
CAMPO_AGRUPAMENTO  = "publication_year"
ARQUIVO_GRAFICO    = "publicacoes_<tema_snake_case>.png"
O que cada variável significa:

API_URL — endereço base da API do OpenAlex, de onde os dados de publicações são obtidos.

EMAIL_CONTATO — placeholder que você deve substituir por um e-mail válido. Ele é enviado no parâmetro mailto para que a requisição entre no polite pool do OpenAlex (acesso mais estável e prioritário). Não invente e-mails reais.

TERMO_BUSCA — termo pesquisado nas publicações. O padrão é "symbolic regression hydrology", mas pode ser trocado por qualquer tema.

DATA_INICIAL — recorte temporal mínimo, no formato 