# 📄 Documentação: Status de pedido dos clientes 1031

Este sistema busca os status de cada pedido feito na PPL utilizando o programa base (5-192, feito em COBOL). Esses dados são enviados para uma API desenvolvida em Python, que grava todas as informações coletadas no banco de dados `api_teste`.

---
## ⚙️ Configuração necessária
### 1. **Arquivo `.env`**

Crie um arquivo chamado `.env` com as informações de conexão ao banco de dados MySQL. Estrutura sugerida:

```
USER=
PASSWORD=
HOST=
DATABASE=
```

O arquivo `.env.config` contém um exemplo de estrutura para referência.

### 2. **Criar um Ambiente Virtual**

No terminal, execute:

```
python -m venv .venv
```
### 3. **Ativar o Ambiente Virtual**

Ativar o ambiente virtual:

No Windows:
```
.venv/Scripts/activate
```

No  Linux/Mac:

```
source .venv/bin/activate
```

### 4. **Instalar Dependências**

Com o ambiente virtual ativo, instale as bibliotecas:

```
pip install -r requirements.txt
```

---

## 💻 **Como executar a API e Acessar o Site**

### 1. **API Principal** 

No terminal:

```
python routes.py
```

### 2. **Acessar o site**

No navegador, digite:

```
192.9.200.47:8080
```

---

# 👨‍💻 **Descrição dos Scripts feitos em python**

## routes.py

Está aplicação Flask fornece um painel web para exibir o status de pedidos e gerenciar imagens de campanhas promocionais (upload e remoção). Ela também integra uma função de recepção de dados de um sistema externo (COBOL) para gravação em banco de dados.

---

### 🗄️ Estrutura de Arquivos

- `app.py` - Arquivo principal da aplicação Flask.
- `grava_banco.py` - Módulo externo responsável por gravar dados vindos do COBOL no banco de dados.
- `concectar_mysql.py` - Módulo externo para realizar conexão com o MySQL.
- `templates/index.html` - Página principal para acompanhar os pedidos.
- `templates/painel.html` - Página de administração das campanhas.
- `static/img/campanha1`, `static/img/campanha2` - Pastas onde imagens são salvas.

### 📂 Estrutura de Diretórios Criada Automaticamente

Ao iniciar a aplicação, os diretórios:

- `static/img/campanha1`
- `statis/img/campanha2`

São automaticamente criados se não existirem.

### ✅ Tipos de Arquivos Permitidos para Upload

- `.png`
- `.jpg`
- `.jpeg`
- `.gif`

### 💻 Endpoints da Aplicação

1. `/` (GET)

**Descrição**:

Carrega a página com a lista de pedidos filtrados pela data e hora atual. Também carrega imagens de campanhas organizadas por pasta.

**Funcionalidade**:

- Consulta pedidos recentes do banco de dados com base na hora do pedido.
- Mostra apenas os pedios que ainda não foram faturados.
- Filtra os pedidos por prioridade.
- Mostra os pedidos em ordem decrescente respeitando a prioridade de cada pedido.
- Exibe imagens das campanhas.
- Reseta o campo `RETIRAR` de `sim` para `nao` apenas de um IP em especifico.

---

2. `/cobol` (POST)

**Descrição**:

Recebe dados JSON via POST de do sistema COBOL e grava no banco de dados.

**Entrada:**

```
{
    "chave1": "valor1",
    ...
}
```

**Saída**:

- `"ok"` com status 200 em caso de sucesso.
- JSON de erro com status `500` em caso de falha.

---

3. `/painel` (GET)

**Descrição**:

Carrega a interface de administração para adicionar novas campanhas e visualizar imagens enviadas por campanha.

---

4. `/upload` (POST)

**Descrição**:

Permite envio de múltiplas imagens via formulário. Alterna entre as pastas `campanha1` e `campanha2`.

**Validações**:

- Verifica se o arquivo existe e se é permitido.
- Alterna pasta de upload entre campanhas.
- Usa `secure_filename()` para evitar problemas de segurança.

**Mensagens Flask**:

- Sucesso ou erro por arquivo.
- Feedback agregado ao final.

---

5. `/remover_item` (GET)

**Descrição**:

Remove um arquivo de imagem com base no `filename` fornecido via query string.

**Parâmetros**:

- `filename` - Nome do arquivo a ser removido.

**Comportamento**:

- Busca em todas as pastas de campanha.
- Remove sse encontrar.
- Exibe mensagens de sucesso ou erro.

---

### 🔁 Funções Auxiliares

#### `allowed_file(filename)`

Verifique se as imagens estão na extensão correta.

---

#### `get_campaing_imagens()`

Retorna um dicionário contendo todas as imagens organizadas por pasta de campanha.

**Exemplo de retorno**:

```
{
    "campanha1": [{"filename": "img1.png", "path": "static/img/campanha1/img1.png"}, ...],
    "campanha2": [{"filename": "img2.png", "path": "static/img/campanha2/img2.png"}, ...]
}
```

---

### 🖥️ Lógica de Consulta de Pedidos

Consulta na tabela `status_venda` usando os seguintes critérios:

- Data atual.
- Pedidos não faturados ('X' significa que o pedido ainda não foi faturado).
- Pega os pedidos que foram conferidos a menos de 30 minutos da hora atual.
- Hora do pedido entre
    - `HORA_ATUAL - 2 horas` e `HORA_ATUAL + 1 hora`
- Se `STATUS_PEDIDO` for `RETIRAR`, aplica uma janela de tempo mais curta.
- Adicona uma prioridade ao status de cada pedido.
- Mostra cada pedido de forma decrescente referente a sua prioridade.

**Ordenação**:

- Prioriza status com maior urgência (`RETIRAR` antes de `CONFERINDO`, etc).
- Dentro do mesmo status, ordena por número de pedido decrescente.

---

## conectar_mysql.py

Script responsável por estabelecer a conexão com o banco de dados **MySQL**, utilizando as configurações definidas no `.env`.

## grava_banco.py

Este script foi desenvolvido para registrar, atualizar e remover informações sobre pedidos e seus respectivos status em tempo real no banco de dados MySQL. Ele interage com dados vindos de um sistema COBOL e assegura que o banco reflita corretamente o andamento do pedido - desde a separação até a retirada.

### 🗄️ Funções

`criar_tabela()`

Verifica se a tabela `status_venda` existe. Caso não exista, ela é criada com os seguintes campos:

- `DATA`: Data do pedido
- `CODIGO_VENDEDOR`: Identificador do vendedor
- `STATUS_PEDIDO`: Status atual do pedido
- `NUM_PEDIDO`: Número do pedido (Primary Key)
- `HORA_NOTA`: Hora em que o pedido foi emitido
- `RETIRA`: Indica se o pedido é retirado na loja
- `HORA_PEDIDO`: Hora do registro
- `NOME_CLIENTE`: Nome do cliente
- `CODIGO_CLIENTE`: Código do cliente
- `TIPO_PAGAMENTO`: Tipo de pagamento (`pix`, `boleto`, etc)
- `NOME_SEPARADOR`: Quem está separando o pedido e quem está conferindo, junto com a hora de que começou a separação ou conferencia
- `RETIRAR`: Se o pedido está pronto para retirada
- `FATURADO`: Se o pedido já foi conferido pelo cliente e teve a nota faturada
- `TEMPO`: Mostra quando por quanto tempo um pedido esta sendo separado

---

`deletar_dados()`
Remove registro da tabela `status_venda` **criados no dia atual** com hora superior a 30 minutos atrás **e** cujo status não seja "`RETIRAR NO CAIXA`".

---

`formatar_hora_banco(hora_valor)`

Converte o valor `HORA_PEDIDO` vindo do banco (em vários formatos possíveis) para o tipo `timedelta`, que facilita comparações entre horários.

---

`gravar_banco(...)`

Insere um novo registro na tabela `status_venda` como os dados processados do sistema COBOL, se ele ainda não existir.

---

`verificar_dados(...)`

Verifica se o registro com a combinação específica (`CODIGO_VENDEDOR`, `STATUS_PEDIDO`, `NUM_PEDIDO`, `CODIGO_CLIENTE`, `TEMPO`) já existente no banco de dados.
- se **existe** o `mudou` recebe `'nao'`;
- caso **não exista** o `mudou` recebe `'sim'`.

---

`atualizar_dados(...)`

Atualiza o registro existente no banco se o `STATUS_PEDIDO` foi alterado. Dependendo do novo status, também atualiza campos como `STATUS_PEDIDO` e `FATURADO`.

---

`deletar_info(...)`

Remove manualmente um registro específico da tabela com base em `CODIGO_VENDEDOR`, `NUM_PEDIDO` e `CODIGO_CLIENTE`. Isso é feito quando há uma descrição no pedido - indicando que o pedido foi cancelado ou vai devolver ao estoque.

---

`filtrar_dados(dados_cobol)`

Recebe um dicionário com os dados vindos do sistema COBOL e executa a lógica:

1. Ignora pedidos com `COD-VEND == 99999` ou `NUM-PEDIDO == 000000`.
2. Mapeia os status antigos(`ASEPARAR`, `SEPARACAO`, etc.) para os novos padronizados (`SEPARAR`, `SEPARACAO`, etc.).
3. Converte tipo de pagamento(`x` -> `pix`, `b` -> `boleto`).
4. Busca a `HORA-INI-SEPARACAO` e a `HORA-INI-CONFERENCIA`.
5. Verifica se a nota foi faturada ou não
6. Busca a data do pedido
7. Verifica se já existe o registro:
    - Se **não** existir: grava o pedido novo
    - Se existir **mas** o status mudou ou faturamento mudou faz a atualização no banco de dados
    - Se houver `DESCRICAO`, o pedido será deletado da tabela

---

`main(dados_cobol)`

Ponto de entrada do script:

1. Loga os dados recebidos
2. Cria a tabela se não existir
3. Processa os dados com `filtrar_dados()`
4. Deleta dados antigos com `deletar_dados()`

### 🖥️ Fluxo Geral
```
Dados COBOL ➜ main()
                └── criar_tabela()
                └── filtrar_dados()
                        ├── verifica_dados()
                        ├── atualizar_dados() ou gravar_banco()
                        └── deletar_info()
                └── deletar_dados()  (limpeza de dados antigos)

```

# 👨‍💻 **Descrição dos templates em HTML**

## index.html

Este template HTML é utilizado para exibir o **acompanhamento dos pedidos**, apresentando visualemnte o status de cada pedido e imagens promocionais, com destaque visual para facilitar a identificação por cores.

---

### 🏠 Estrutura Geral

- Head:
    - Link para:
        - Estilo CSS (style.css).
        - Ícone da pagina (fiveicon.png).
- Body:
    - Contém todo o conteúdo visível da aplicação.

---

### 💻 Elementos Principais

#### Logo

```
<img src="{{ url_for('static', filename='img/logo.png') }} ...">
```

Exibe o logotipo da PPL.

---

#### Legenda por Cores

```
<div class="legenda">
    <span class="verde"></span>
    <span class="azul"></span> 
    <span class="amarelo"></span>
    <span class="laranja"></span> 
    <span class="vermelho"></span> 
</div>
```

Indica visualmente o significado de cada cor usada nos pedidos (comportamento definido por CSS):

- `Verde`: Pedido pronto para retirada no caixa.
- `Azul`: Em Conferência.
- `Amarelo`: Separado.
- `Laranja`: Em Separação.
- `Vermelho`: A Separar.

--- 

#### Áudio de Notificação

```
<audio id="notificacao" src="caminho ate a notificação"></audio>
```

Som de notificação que e ativado via JavaScript para notificar quando um pedido pode ser retirado no caixa.

#### 📂 Tabela de Pedidos

```
<table class="tabela-pedidos" id="tabelaPedidos">...</table>
```

- Cabeçalho (`thead`):
    - PEDIDO, NOME CLIENTE, SEPARADOR/CONFERENTE, STATUS.
- Corpo(`tbody`):
    - Utiliza loop `for` do Jinja2 para iterar sobre a viriável `dados`.
    - Cada linha (`<tr>`) recebe uma **classe de cor** baseada no `STATUS_PEDIDO`.
        ```
        {% if 'RETIRAR NO CAIXA' in item.STATUS_PEDIDO %}verde{% elif ... %}
        ```
    - A propriedade `data-retirar="sim"` ou `"nao"` também é adicionada para lógica JS adicional.

---

#### 🖼️ Imagens da Campanha

```
<div class="imagens-campanha">...</div>
```

Se `imagens_campanha` estiver presente, são exibidas as imagens conforme o dicionário:

- **Chave**: nome da campanha (ex: `campanha1`, `campanha2`).
- **Valor**: lista de objetos com `path` e `filename`.

---

#### 📄 Paginação
```
<div class="pagination" id="pagination"></div>
```

Esse elemento será utilizado dinamicamente pelo JavaScript para poder fazer a paginação entre as páginas da tabela.

## painel.html

Este template é responsável por rederizar a interface de **gerenciamento de campanhas**. Ele permite que o usuário:

- Façam upload de novas imagens.
- Visualizem as campanhas existentes.
- Excluam imagens individualmente.

---

### 📂 Estrutura do Arquivo

- `<head>`
    - **CSS local**: `painel.css` é carregado do diretório `/static/css/`.
    - **Favicon**: ícone personalizado do site (`fiveicon.png`).
    - **FontAwesome**: biblioteca de ícones.

- `<body>`
    - **Navegação** (`<nav>`)

### 📄 Formulário de Upload de Imagens
```
<form action="/upload" method="post" enctype="multipart/form-data">
```

- Permite o envio de múltiplas.
- Envia via `POST` para a rota `/upload`.

### 🖼️ Secão de Imagens Salvas
```
<h2>Imagens Salvas</h2>
<div class="imagens-campanha">
```

- Verifica se existem imagens nas campanhas `campanha1` ou `campanha2`.
- Usa loops `for` para percorrer as pastas e imagens:
    - Casa imagem é renderizada com a tag `<img>`.
    - Ao lado de cada imagem, há um ícone de lixeira com atributo `data-filename` para identificar a imagem a ser deletada (manipulado por JavaScript).

# 👨‍💻 **Descrição dos scripts feitos em JavaScript**

## script.js

Este scrript alterna automaticamente entre:

- Tabelas de pedidos paginadas (com alertas sonoros caso haja itens a serem retirados no caixa).
- Imagens das campanhas.

### ⚙️ Constantes e Varíaveis Globais

| Nome da Variável               | Função                                                            |
| ------------------------------ | ----------------------------------------------------------------- |
| `rowsPerPage`                  | Quantidade de linhas mostrada por pagina                          |
| `currentPage`                  | Página atual da tabela                                            |
| `tableBody`, `tableHeader`     | Referência aos elementos `<tbody>` e `thead` da tabela            |
| `paginationDiv`                | Elemento onde os indicadores de página são renderizados           |
| `imagensCampanhaDiv`           | Container com as imagens das campanhas                            |
| `tituloPedidos`                | Título da seção de pedidos                                        |
| `allRows`                      | Array com todas as linhas da tabela                               |
| `autoSwapInterval`             | ID do `setInterval` usado para alternância                        |
| `tabelaDisplayTime`            | Tempo que cada página da tabela permanece visível                 |
| `imagemDisplayTime`            | Tempo que cada imagem permanece visível                           |
| `somEmitido`                   | Evita tocar o som de retirada múltiplas vezes                     |
| `currentImageIndex`            | Índice da imagem atualmente visível                               |
| `isShowingTable`               | Indica se a tabela está sendo exibida atualmente                  |
| `currentCampaign`              | Campanha atual (1 ou 2)                                           |
| `campaignImages`               | Lista de imagens da campanha atual                                |
| `imagesShownInCurrentCampaign` | Quantas imagens já foram exibidas nesta campanha                  |
| `isFinishingTablePages`        | Se estamos finalizando a exibição das páginas da tabela           |
| `audioNotificacao`             | Áudio que será tocado quando uma linha tiver `data-retirar="sim"` |

### 💻 Funções Auxiliares

#### `emitirSomSeHouverRetirar()`

Verifica se alguma linha tem o atributo `data-retirar="sim"` e toca um som se ainda não tiver sido emitido.

---

#### `loadTableData()`

Carrega as linhas da tabela, ordena, exibe a página atual, inicializa os indicadores e toca o som se necessário.

#### `showPage(page)`

Exibe apenas as linhas da página especificada. Oculta todas as outras.

#### `nextTablePage()`

Avança para a próxima página da tabela. Se estiver n última, volta à primeira.

#### `setupCarouselIndicators()`

Cria indicadores clicáveis para as páginas da tabela.

#### `updateActiveIndicator()`

Destaca visualmente o indicador da página atual.

#### `ordenarTabela()`

Ordena as linhas da tabela pela cor da prioridade

- Verde (1), Azul (2), Amarelo (3), Laranja (4), Vermelhor (5)

#### `hideAllElements()`, `showTableElements()`, `showImageElements(index)`

Controlam a visibilidade dos elementos da tela (tabela e imagens).

### 🔁 Função Principal: `cybleDisplay()`

Controla o que será ecibido a cada ciclo de tempo:

- Se `isShowingTable == true`: exibe a próxima página da tabela
    - Se estamos finalizando a exibição (`isFinishingTablePages`), continua mostrando até a última página e então recarrega a página com a próxima campanha.
- Se `false`: exibe a próxima imagem da campanha
    - Quando todas as imagens forem exibidas, entra no modo de finalização da tabela.

### 🎯 Função de Inicialização: `startAutoSwap()`

Reseta todos os estados e inicia o ciclo de exibição, carregando:

- Página 0 da tabela
- Imagem 0 da campanha atual
- Lista de imagens da campanha

Se nenhuma imagem da campanha atual for encontrada, tenta a outra campanha.

### 🖥️ Evento `DOMContentLoaded`

Ao carregar a página:

- Verifica no `localStorage` se havia uma campanha em andamento
- Define a campanha corrente
- Inicia o ciclo com `startAutoSwap()`

### 💻 Lógica de Campanha

- As imagens da campanha atual são exibidas uma a uma
- Após exibir todas as imagens de uma campanha:
    - A tabela volta a ser exibida até a última página.
    - Em seguida, o ciclo reinicia com a **outra campanha ( 1 ou 2)** e a página é recarregada.

### 🔁 Armazenamento Temporário (localStorage)

- `currentCampaign`: Salva qual campanha será usada após recarregar.
- É **limpo automaticamente** no primeiro carregamento após salvar, garantindo que o usuário sempre veja a campanha 1 em um reload manual.

## painel.js

Este script é responsável por adicionar um evento de clique aos ícones de lixeira (com a classe `.remover-imagem`) em no painel do sistema. Ao clicar no ícone, o script redireciona o usuário para uma URL responsável por remover o item correspondente do sistema, passando o nome do arquivo como parâmetro de consulta.

### ⚙️ Estrutura e Funcionamento

#### Evento `DOMContentLoaded`

```
document.addEventListener('DOMContentLoaded', function() { ... });
```

- Garante que o script seja executado **somente após o carregamento completo do DOM**.
- Isso evita erros ao tentar selecionar elementos que ainda não foram renderizados.

#### Seleção de elementos com `.remover-imagem`

```
const removerImagens = document.querySelectorAll('.remover-imagem');
```

- Seleciona todos os elementos que possuem a classe `remover-imagem`.
- Esses elementos são presumivelmente ícones de lixeira ou botões usados para remover imagens.

#### Adição do evento de clique

```
removerImagens.forEach(remover => {
    remover.addEventListener('click', function(event) { ... });
});
```

- Para cada botão de remoção encontrado:
    - Um **ouvinte de evento de clique** é adicionado.
    - Quando clicado, aa função de remover a imagem será executada.

#### Cancelamento do comportamento padrão

```
event.preventDefault();
```

- Impede a ação padrão do elemento, como seguir o link se for uma tag `<a>`.
- Isso permite o controle completo do que acontece após o clique.

#### Obtenção do nome do arquivo

```
const filename = this.dataset.filename;
```

- Obtém o valor do atributo `data-filename` do elemento clicado.
- Este valor é o **nome da imagem** a ser removida.

#### Redirecionamento para endpoint de remoção

```
window.location.href = `/remover_item?filename=${filename}`;
```

- Redireciona o navegador para a URL de remoção.
- O backend (em `/remover_item`) deve tratar a lógica de remoção da imagem com base no parâmetro `filename`.

#### Tratamento de erro

```
console.error('Nome do arquivo não encontrado no ícone de lixeira.');
alert('Erro: Nome do arquivo da imagem não foi encontrado.');
```

- Se `filename` estiver ausente, uma mensagem de erro será exibida no console.
- Um alerta também é mostrado ao usuário para avisá-lo do problema.

---

### 📂 Requisitos para funcionamento

- Os ícones de remoção devem conter o atributo `data-filename`, como no exemplo:

```
<i class="fa-solid fa-trash remover-imagem" data-filename="{{ imagem.filename }}"></i>
```
