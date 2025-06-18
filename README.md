# 📄 Documentação: Painel de pedidos da loja 1031

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

## 💻 **Como executar a API e Acessando o Site**

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

Esta aplicação Flask serve como um **painel abragente** para monitorar o status de pedidos, gerenciar campanhas promocionais (incluindo upload e romeção de imagens e vídeos), e atuar como um ponto de integração para dados externos. Ela recebe informações de um sistema COBOL e as persiste em um banco de dados MySQL.

---

### 🗄️ Estrutura de Arquivos

A estrutura de arquivos e diretórios da aplicação é orfanizada da seguinte forma:

- `routes.py`: O **arquivo principal** de aplicação Flask, contendo a lógica de roteamento e as funções das views.
- `grava_banco.py`: Um módulo auxiliar responsável por **persistir dados** recebidos do sistema COBOL no banco de dados.
- `concectar_mysql.py`: Um módulo dedicado ao **gerenciamento da conexão** com o banco de dados MySQL.
- `templates;`: Diretório que armazena os templates HTML da aplicação.
    - `templates/index.html`: A **página inicial** que exibe o acompanhamento dos pedidos.
    - `templates/painel.html`: A interface de **administração** para gerenciar companhas e vídeos.
- `static`: Diretório para arquivos estáticos como imagens, vídeos e scripts JavaScript.
    - `static/img/campanha1`: Pasta para imagens da Campanha 1.
    - `static/img/campanha2`: Pasta para imagens da Campanha 2.
    - `static/video`: Pasta para vídeos sobre a empresa.
    - `static/js/script.js`: Script JavaScript que gerencia a exibição e interatividade da tabela de pedidos, campanhas e vídeos na página inicial.
    - `static/js/painel.js`: Script JavaScript responsável pela funcionalidade de remoção de imagens e vídeos no painel de administração.

### 📂 Estrutura de Diretórios Criada Automaticamente

Para garantir a funcionalidade de upload e armazenamento, os seguintes diretórios são **criados automaticamente** na inicialização da aplicação, caso não existam:

- `static/img/campanha1`
- `statis/img/campanha2`
- `static/video`

### ✅ Tipos de Arquivos Permitidos para Upload

A aplicação aceita o upload dos seguintes tipos de arquivos:

- Imagens: `png`, `jpg`, `jpeg`, `gif`
- Vídeos: `mp4`

### 💻 Endpoints da Aplicação

1. `/` (GET)

**Descrição**:

Este endpoint carrega a **página principal** da aplicação, exibindo a lista de pedidos, imagens de campanhas promocionais e um vídeo institucional da empresa.

**Funcionalidade Detalhada**:

- `Consulta de Pedidos`: Realiza uma consulta ao banco de dados (`statu_venda`) para buscar pedidos recentes com base na data e hora.

- `Filtragem de Pedidos`:
    - Exibe apenas os pedidos com `DATA` igual à data atual.
    - Filtra pedidos onde `FATURADO` é igual a `X` (indicando que o pedido ainda não foi faturando).
    - A hora do pedido deve estar dentro desses padrões:
        - Para pedidos com `STATUS_PEDIDO = 'RETIRAR'`: `HORA_PEDIDO` deve estar no prazo de 30 minutos antes da hora atual.
        - Para outros `STATUS_PEDIDO` : `HORA_PEDIDO` deve ser 1 hora antes da hora atual e 1 hora depois da hora atual.

- `Priozação e Ordenação`:
    - Os pedidos são ordenados por prioridade baseados no `STATUS_PEDIDO`:
        - `RETIRAR` (maior prioridade - vizualiação verde)
        - `CONFERIDO` (visualização azul)
        - `SEPARADO` (visualização amarela)
        - `SEPARACAO`: (visualização laranja)
        - Outros status (menor prioridade - visualização vermelha)
    - Dentro do mesmo nível de prioridade, os pedidos são ordenados por `NUM_PEDIDO` em **ordem decrescente**.

- `Exibição de Mídia`:
    - Apresenta um **vídeo** sobre a empresa.
    - Exibe **imagens das campanhas** promocionais, organizadas por suas respectivas pastas (`campanha1`, `campanha2`).

- Reset de Campo `RETIRAR`:
    - Se a requisição for originada do **endereço IP 192.9.200.231**, o campo `RETIRAR` de todos os pedidos que estão com `'sim'` é automaticamente **resetado para** `'nao'` no banco de dados.

---

2. `/cobol` (POST)

**Descrição**:

Este endpoint é dedicado à **recepção de dados JSON** enviados via requisição POST de um sistema COBOL. Os dados recebidos são processados e gravados no banco de dados utilizando o módulo `grava_banco.py`

**Entrada (Exemplo JSON):**

```
{
    'COD-VEND': '',
    'PEDIDO-SEPARADO': '',
    'NUM-PEDIDO': '',
    'RETIRA': '', 
    'NOME-CLIENTE': '', 
    'COD-CLIENTE': '', 
    'BOLETO': '', 
    'DESCRICAO': '', 
    'SEPARADOR': '', 
    'HORA-NOTA': '', 
    'HORA-INI-CONFERENCIA': '', 
    'HORA-INI-SEPARACAO': '', 
    'NOTA-FATURADA': '', 
    'DATA-PEDIDO': ''
}
```

**Saída**:

- **Sucesso**: Retorna a string `"ok"` com um `status HTTP 200`.
- **Falha**: Retorna um **JSON de erro** (`{"error": "Erro ao processar dados"}`) com um **status HTTP 500**. Erro são logados no console.

---

3. `/painel` (GET)

**Descrição**:

Carrega a **interface de administração** da aplicação. Nesta página, os usuários podem adicionar novas campanhas, carregar vídeos e visualizar todas as imagens e vídeos já enviados, organizados por campanha.

---

4. `/upload` (POST)

**Descrição**:

Permite o **upload de múltiplos arquivos** (imagens ou vídeos) através de um formulário. O sistema alterna automaticamente entre as pastas `campanha1`, `campanha2` para imagens e salva vídeos em `static/video`.

**Validações e Comportamento**:

- **Verificação de Anexo**: Verifica se um arquivo foi enviado no formulário (`arquivo` no `request.files`).
- **Seleção de Arquivos**: Garante que pelo menos um arquivo foi selecionado no input do formulário.
- **Extensão Permitida**: Confere se a extensão de cada arquivo é uma das permitidas
- **Nome seguro**: Utiliza a função `secure_filename()` do Werkzeug para sanitizar os nomes dos arquivos, prevenindo vulnerabilidades de seguraça.
- **Determinação da Pasta de Destino**:
    - Se o arquivo for um `.mp4`, ele é salvo em `static/video`.
    - Para imagens (`.png`, `.jpg`, `.jpeg`, `.gif`), a pasta de destino alterna entre `static/img/campanha1` e `static/img/campanha2`.
- **Tratamento de Erros de Gravação**: Se ocorrer um erro durente a gravação de um arquivo, o erro é registrado e o upload do arquivo é contabilizado como falha.

**Mensagens Flask**:

- São exibidas mensagens de **sucesso ou erro** (usando `flash`) para o usuário, indicando o resultado de cada upload.

- Um **feedback agregado** é fornecido ao final do processo, informando o total de arquivos enviados com sucesso e o número de erros, se houver.

---

5. `/remover_item` (GET)

**Descrição**:

Permite a **remoção de um arquivo** (imagem ou vídeo) do servidor com base no nome do arquivo, tipo e nome da pasta fornecedos via query string.

**Parâmetros**:

- `filename`: O nome completo do arquivo a ser removido (ex: `minha_imagem.jpg`).
- `file_type`: O tipo de arquivo, podendo ser `imagem` ou `video`.
- `folder`: O nome da subpasta onde o arquivo está localizado (ex: `campanha1` ou `video`)

**Comportamento**:

- Valida a presença dos parâmetros necessários.
- Constrói o **caminho completo** do arquivo com base no `file_type` e `folder`.
    - Para `file_type='image'`, busca em `static/img/<folder_name>`.
    - Para `file_type='video'`, busca em `static/video`.
- Verifica se o arquivo existe no caminho especificado.
- Se o arquivo for encontrado, ele é **removido** do sistema de arquivos.
- Mensagens de sucesso ou erro (incluido "arquivo não encontrado" ou erros de permissão) são impressas no console do servidor.
- Após a operação, o usuário é **redirecionado de volta para o painel** de administração.

---

### 🔁 Funções Auxiliares

#### `allowed_file(filename)`
**Descrição:**

Esta função auxiliar verifica se a extensão do arquivo fornecido (`filename`) está na lista de <ins>tipos de arquivos permitidos</ins>.

**Retorno**:

- `True` se a extensão do arquivo for permitida.
- `False` caso contrário.

---

#### `get_campaing_imagens()`

**Descrição**:

Esta função escaneia os diretórios de campanhas e vídeos e retorna um dicionário estruturado contendo todos os arquivos de mídia encontrados. Ela distingue entre imagens de campanha (organizadas por pasta) e vídeos.

**Exemplo de retorno**:

```
{
    "images": {
        "campanha1": [
            {"filename": "img1.png", "path": "/static/img/campanha1/img1.png", "type": "image"},
            {"filename": "banner.jpeg", "path": "/static/img/campanha1/banner.jpeg", "type": "image"}
        ],
        "campanha2": [
            {"filename": "promo.jpg", "path": "/static/img/campanha2/promo.jpg", "type": "image"}
        ]
    },
    "videos": [
        {"filename": "empresa.mp4", "path": "/static/video/empresa.mp4", "type": "video"}
    ]
}
```

---

### 🖥️ Lógica de Consulta de Pedidos (`/` endpoint)

A consulta de pedidos é realizada na tabela `status_venda` do banco de dados, seguindo critérios específicos para garantir a exibição dos pedidos mais relevantes e urgentes:

- **Filtro por Data e Hora**:
    - A consulta considera a **data atual** (`DATA = %s`)
    - Para pedidos com `STATUS_PEDIDO = 'RETIRAR'`, a `HORA_PEDIDO` deve estar entre 30 minutos antes da hora atual e 30 minutos depois da hora atual.
    - Para os demais status de pedidos, a `HORA_PEDIDO` deve estar entre 1 hora a menos da hora atual e 1 hora depois da hora atual.
- **Pedidos Não Faturados**: Apenas pedidos com `FATURADO = 'X'` (indicando não faturado) são incluídos.
- **Prioridade dde Status**: Os pedidos são ordenados por prioridade de `STATUS_PEDIDO` usando uma função `CASE`:
    - `RETIRAR` (prioridade 1)
    - `CONFERINDO` (prioridade 2)
    - `SEPARADO` (prioridade 3)
    - `SEPARACAO` (prioridade 4)
    - Qualquer outro status (prioridade 5)
- **Ordenação Secundária**: Dentro de cada grupo de prioridade, os pedidos são ordenados por `NUM_PEDIDO` em **ordem decrescente**.

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

Verifica se a tabela `status_venda` existe no banco de dados. Caso não exista, ela é **criada automaticamente** com a seguinte estrutura:

- `DATA`: Data do pedido (`DATE`)
- `CODIGO_VENDEDOR`: Identificador do vendedor (`INT`)
- `STATUS_PEDIDO`: Status atual do pedido (`VARCHAR(50)`)
- `NUM_PEDIDO`: Número do pedido (Chave Primária `INT PRIMARY KEY`)
- `HORA_NOTA`: Hora em que o pedido foi emitido (`VARCHAR(5)`)
- `RETIRA`: Indica se o pedido é retirado na loja (`VARCHAR(2)`)
- `HORA_PEDIDO`: Hora do registro no banco (`TIME`)
- `NOME_CLIENTE`: Nome do cliente (`VARCHAR(255)`)
- `CODIGO_CLIENTE`: Código do cliente (`INT`)
- `TIPO_PAGAMENTO`: Tipo de pagamento (ex:`pix` ou `boleto`. - `VARCHAR(6)`)
- `NOME_SEPARADOR`: Quem está separando o pedido ou quem está conferindo, junto com a hora de que começou a separação ou conferencia (`VARCHAR(30)`)
- `RETIRAR`: Se o pedido está pronto para retirada (`VARCHAR(3)`)
- `FATURADO`: Se o pedido já foi conferido pelo cliente e teve a nota faturada (`VARCHAR(45)`)
- `TEMPO`: Mostra por quanto tempo um pedido esta sendo separado (`VARCHAR(4)`)

---

`deletar_dados()`

Remove registro da tabela (`status_venda`) que foram **criados no dia atual**, têm mais de **30 minutos** (em relação à `HORA_PEDIDO` do registro) e **não** possuem o `STATUS_PEDIDO` como "`RETIRAR NO CAIXA`". Esta função ajuda na limpeza de dados antigos e irrelavantes.

---

`formatar_hora_banco(hora_valor)`

Função auxiliar que converte valores da hora vindo do banco de dados (que podem ser `timedelta`, `time` ou `str` em diferentes formatos) para o tipo `timedelta`. Isso padroniza o formato e facilita comparações de tempo. Retorna `None` se a conversão falhar.

---

`gravar_banco(...)`

Insere um novo registro na tabela `status_venda` como os dados processados do sistema COBOL.

A lógica de inserção varia ligeiramente com base no `STATUS_PEDIDO` do pedido:
    - **'SEPARAR'**: Insere o registro e preenche o campo `TEMPO` com a `hora_separacao`.
    - **'SEPARACAO'**: Calcula o `tempo` decorrido desde o início da separação (`hora_separacao`) e o anexa ao `NOME_SEPARADOR`.
    - **Outros status**: Insere o registro com os campos fornecidos.

---

`verificar_dados(...)`

Verifica a existência de um registro no banco de dados com base em uma combinação de `CODIGO_VENDEDOR`, `STATUS_PEDIDO`, `NUM_PEDIDO`, `CODIGO_CLIENTE` e `TEMPO`.
- se **existe** o `mudou` recebe `'nao'`;
- caso **não exista** o `mudou` recebe `'sim'`.

---

`atualizar_dados(...)`

Atualiza o registro existente na tabela `status_venda` se houver uma alteração no `STATUS_PEDIDO` ou no status `FATURADO`.
- Compara o `STATUS_PEDIDO` e `FATURADO` atual com os gravador no banco de dados.
- Se o `STATUS_PEDIDO` mudou:
    - Para `RETIRAR`: Atualiza o status, anexa a `hora_conferecia` ao `NOME_SEPARADOR` e define `RETIRAR` como `sim`.
    - Para `SEPARACAO`: Calcula o tempo decorrido da separação (`TEMPO` do banco ou `hora_separacao` se o `TEMPO` estiver vazio) e o anexa ao `NOME_SEPARADOR`. Define `RETIRAR` como `'nao'`.
    - Para `CONFERINDO` (ou outros status): Anexa a `hora_conferencia` ao `NOME_SEPARADOR` se `hora_conferencia` não estiver vazia, e define `RETIRAR` como `'nao'`.
- Se o `STATUS_PEDIDO` for **'SEPARAR'** e o `TEMPO` **precisar seu atualizado**: Apenas o campo `TEMPO` é atualizado.
- Se apenas o `FATURADO` **mudou** ( e o `STATUS_PEDIDO` for **'RETIRAR'**): Atualiza apenas o status `FATURADO`.
- Retorna `'sim'` se a atualização ocorrer, e `'nao'` caso contrário.

---

`deletar_info(...)`

Remove um registro específico da tabela `status_venda` com base no `CODIGO_VENDEDOR`, `NUM_PEDIDO` e `CODIGO_CLIENTE`. Esta função é acionada quando dado COBOL contém uma `DESCRICAO`, o que geralmente indica um cancelamento ou devolução ao estoque do pedido.

---

`filtrar_dados(dados_cobol)`

Recebe um dicionário com os dados brutos vindos do sistema COBOL e executa a lógica principal de processamento:

1. **Filtra dados irrelevantes**: Ignora pedidos onde `COD-VEND` é `99999` ou `NUM-PEDIDO` é `000000`.
2. Verifica `DESCRICAO`:
    - Se `DESCRICAO` **não estiver vazia**: Chama `deletar_info()` para remover o pedido.
    - Se `DESCRICAO` **estiver vazia**: Procede com o processamento do pedido.
3. **Mapeamento de Status**: Converte status de pedidos antigos (`ASEPARAR`, `SEPARADOS`, `CONFERENCIA`) para seus equivalentes padronizados (`SEPARAR`, `SEPARADO`, `CONFERENCIO`). `SEPARACAO` e `RETIRAR` são mantidos.
4. **Definição de** `RETIRAR`: Baseia o valor inicial de `RETIRAR` (`'sim'` ou `'nao'`) no `STATUS_PEDIDO`.
5. **Conversão de** `TIPO_PAGAMENTO`: Converte códigos de pagamento (`'x'` para `'pix'`, `'b'` para `'boleto'`).
6. **Extração de Horas**: Obtém `HORA-INI-CONFERENCIA`, `HORA-INI-SEPARACAO` (se aplicável), `HORA-NOTA`, `NOME-CLIENTE`, `SEPARADOR`, `NOTA-FATURADA` e `DATA-PEDIDO` dos dados COBOL.
7. **Verificação e Ação no Banco**:
    - Chama `verifica_dados()` para checar a existência e o estado do pedido.
    - Se o pedido **não existe** ou se o `STATUS_PEDIDO` ou `FATURADO` **tiverem mudado**, tenta `atualizar_dado()`.
    - Se `atualizar_dados()` não efeticar a mudança (ou seja, o registro não existia ou não foi atualizado), então `gravador_banco()` é chamado para inserir o novo registro.

---

`main(dados_cobol)`

É o ponto de entrada principal do script:

1. **Log de Entrada**: Registra os dados COBOL recebidos para fins de depuração e monitoramento.
2. **Criação da Tabela**: Chama `criar_tabela()` para garantir que a tabela `status_venda` exista.
3. **Processamento de Dados**: Invoca `filtrar_dados()` para remover registros desatualizados.

### 🖥️ Fluxo Geral
```
Dados COBOL 
      ⬇
main(dados_cobol)
      ├── Logging (registra dados recebidos)
      ├── criar_tabela()          (garante que a tabela existe)
      ├── filtrar_dados()         (lógica de processamento principal)
      │      ├── Condicional: DESCRICAO vazia?
      │      │    ├── SIM (processa pedido)
      │      │    │    ├── verifica_dados()        (checa se o registro e status existem)
      │      │    │    └── Condicional: Registro mudou ou não existe?
      │      │    │          ├── SIM: atualizar_dados()  (tenta atualizar o registro existente)
      │      │    │          └── SE atualizar_dados RETORNOU 'nao': gravar_banco() (insere novo registro)
      │      │    └── NAO (ignora e deleta)
      │      │          └── deletar_info()          (remove o pedido)
      └── deletar_dados()         (limpa registros antigos do dia)

```

# 👨‍💻 **Descrição dos templates em HTML**

## index.html

Este template HTML é utilizado para exibir o **acompanhamento dos pedidos**, apresentando visualmente o status de cada pedido, juntamente com campanhas promocionais de imagens e vídeos, com destaque visual para facilitar a identificação por cores.

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
<img src="{{ url_for('static', filename='img/logo.png') }}" alt="Logo PPL" class="logo">
```

Exibe o logotipo da PPL, centralizado na parte superiro da página.

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

Este elemento é uma div que contém spans vazios, estilizados via CSS para funcionar como uma **legenda visual** para os diferentes status de pedidos na tabela. As cores e seus significados são:

- `Verde`: Pedido pronto para retirada.
- `Azul`: Pedido em Conferência.
- `Amarelo`: Pedido Separado.
- `Laranja`: Pedido em Separação.
- `Vermelho`: Pedido a Separar.

--- 

#### Áudio de Notificação

```
<audio id="notificacao" src="{{ url_for('static', filename='effect_som/notificacao.mp3') }}" preload="auto"></audio>
```

Um elemento de áudio que carrega o arquivo `notificacao.mp3`. Ele é configurado para `preload="auto"` para estar pronto para reprodução. A reprodução deste áudio é **ativada via JavaScript** para notificar o usuário quando um pedido está pronto para retirar no caixa.

#### 📂 Tabela de Pedidos

```
<table class="tabela-pedidos" id="tabelaPedidos">...</table>
```

A tabela principal que exibe os detalhes dos pedidos:

- Cabeçalho (`thead` com `id="cabecalhoTabela"`):
    - Define as colunas da tabela: **PEDIDO, PEDIDO EMITIDO, NOME CLIENTE, SEPARADOR/CONFERENTE** e **STATUS.**
- Corpo(`tbody` com `id="corpoTabela"`):
    - Utiliza um **loop** `{% for item in dados %}` **do Jinja2** para iterar sobre a viriável `dados` (que contém os registro de pedidos do banco de dados).
    - **Cada linha** (`<tr>`) representa um pedido e recebe uma **classe de cor** (verde, azul, amarelo, laranja e vermelho) baseada no `item.STATUS_PEDIDO`, refletindo a legenda.
        ```
        <tr class="{% if 'RETIRAR' in item.STATUS_PEDIDO %}verde{% elif 'CONFERINDO' in item.STATUS_PEDIDO %}azul{% elif 'SEPARADO' in item.STATUS_PEDIDO %}amarelo{% elif 'SEPARACAO' in item.STATUS_PEDIDO %}laranja{% else %}vermelho{% endif %}" ...>
        ```
    - Uma propriedade `data-retirar` é adicionada a cada linha, com valor `"sim"` se `item.RETIRAR == 'sim'` e `'nao'` caso contrário, para ser utilizada pela lógica JavaScript.
    - As células (`<td>`) exibem os dados correspondentes: `NUM_PEDIDO`, `HORA_NOTA`, `NOME_CLIENTE`, `NOME_SEPARADOR` e `STATUS_PEDIDO`. A coluna `NOME_CLIENTE` possui a classe `cliente`.
    - Caso não haja nenhum pedido na variável `dados`, uma linha com a mensagem "Nenhum pedido encontrado." é exibida (`{% else %}` do Jinja2).

---

#### 🖼️ Imagens da Campanha

```
<div class="imagens-campanha">...</div>
```

Um contêiner (`div`) com o `id="media-slides-container"` e a classe `hidden` (indicando que sua visibilidade é controlada por JavaScript). Esta contêiner funciona como um **carrossel de mídia**, exibindo vídeos e imagens de campanhas.

- Vídeos:
    - Itera sobre `all_media.videos`. Cada vídeo é encapsulado em uma `div` com `class="carousel-slide-item"` e `data-type="video"`.
    - Utiliza a tag `<video>` com atributos `controls`, `muted`, `playsinline` e `preload="auto"` para exibir o vídeo. A `scr` é definida dinamixamente pelo Jinja2.
- Imagens de campanha 1:
    - Itera sobre `all_media.images.campanha1`. Cada imagem está em uma `div` com `class="carousel-slide-item"`, `data-type='image'` e `data-compaign-floder="campanha1"`.
    - Exibe a imagem usando a tag `<img>` com `src` e `alt` definidor dinamicamente.
- Imagens da campanha2:
    - Itera sobre `all_media.imagens.campanha2`. Estrutura similar às imagens da campanha 1, mas com `data-campaign-floder="campanha2"`.

---

#### 📄 Paginação
```
<div class="pagination" id="pagination"></div>
```

Um elemento `div` com a classe `pagination` e `id=pagination`. Este elemento será utilizado **dinamicamente pelo JavaScript** (no `script.js`) para criar e gerenciar os controler de paginação da tabela de pedidos, permitindo a navegação entre múltiplas páginas de resultados.

#### 💻 Script JavaScript

```
<script src="{{ url_for('static', filename='js/script.js') }}"></script>
```

Inclui o arquivo JavaScript `script.js`, que é responsável pela lógica de interatividade da página, incluindo:
    - Gerenciamento da tabela de pedidos (paginação, filtros).
    - Controle do áudio de notificação.
    - Controle do carrossel de mídia (exibição e transição de vídeos e imagens).

## painel.html

Este template é responsável por rederizar a interface de **gerenciamento de campanhas e vídeos**. Ele permite que o usuário faça o upload de novos arquivos de mídia, visualize as mídias existentes (imagens e vídeos) e exclua itens individualmente.

---

### 📂 Estrutura do Arquivo

- `<head>`:
    - **CSS local**: `painel.css` é carregado do diretório `/static/css/`.
    - **Favicon**: ícone personalizado do site (`fiveicon.png`).
    - **Font Awesome**: Linka a biblioteca de ícones Font Awesome para utilizar ícones visuais na interface.

- `<body>`:
    - Contém todo o conteúdo visível da aplicação, aninhado dentro da tag `<main>`.

### 📄 Formulário de Upload de Imagens
```
<form action="/upload" method="post" enctype="multipart/form-data">
```

Este formulário permite o envio de arquivos de mídia para o servidor:
- **Action**: Envia os dados via método `POST` para a rota `/upload`.
- `enctype="multipart/form-data"`: necessário para upload de arquivos.
- **Campo de Entrada de Arquivos**:
    ```
        <input type="file" id="arquivo" name="arquivo" multiple accept="image/*,video/*">
    ```
Permite selecionar **múltiplos arquivos (`multiple`)** e aceita tando **imagens** (`image/*`) quanto **vídeos** (`video/*`).


### 🖼️ Secão de Mídia Salvas
```
<h2>Imagens Salvas</h2>
<div class="all-media-display">
    {# ... conteúdo dinâmico ... #}
</div>
```
Esta seção exibe todas as imagens e vídeos que já foram enviados e estão armazenados no servidor.

- **Contêiner** `all-media-display`: Uma nova classe para agrupar visualmente todos os itens de mídia.
- **Verificação de Mídia**: Um bloco Jinja2 `{% if all_media %}` verifica se há alguma mídia para exibir. Se não houver, uma mensagem "Nenhuma mídia (imagens ou vídeos) encontrada." é mostrada.
- **Exibição de Imagens por Campanha**:
    ```
    {% for folder_name, images_list in all_media.images.items() %}
        {% if images_list %}
            {% for imagem in images_list %}
                <div class="media-item">
                    <img class="campanhas" src="{{ imagem.path }}" alt="{{ imagem.filename }}">
                    <i class="fa-solid fa-trash remover-media" data-filename="{{ imagem.filename }}" data-type="image" data-folder="{{ folder_name }}"></i>
                </div>
            {% endfor %}
        {% endif %}
    {% endfor %}
    ```
- Itera sobre as subpastas de imagens (ex: `campanha1`, `campanha2`) e exibe cada imagem.
- Cada imagem é renderizada com a tag `<img>` e envolta em uma `div` com a classe `media-item`.
- Um **ícone de lixeira** (`fa-solid fa-tranh`) é adicionado ao lado de cada imagem, com atributos `data-filename`, `data-type="image"` e `data-folder` para identificar a imagem a ser deletada. Estes atributos são usados pelo JavaScript.

- **Exibição de Vídeos**:

```
{% if all_media.videos %}
    {% for video in all_media.videos %}
        <div class="media-item">
            <video controls preload="auto" style="width: 100%; height: 100%;">
                <source src="{{ video.path }}" type="video/mp4">
                Seu navegador não suporta a tag de vídeo.
            </video>
            <i class="fa-solid fa-trash remover-media" data-filename="{{ video.filename }}" data-type="video" data-folder="video"></i>
        </div>
    {% endfor %}
{% endif %}
```

- Itera sobre a lista de vídeos e exibe cada um.
- Cada vídeo é incorporado usando a tag `<video>` com controles e `preload="auto"`, também envolto em um `div.media-item`.
- Assim como nas imagens, um **ícone de lixeira** é fornecido para remoção, com `data-type="video"` e `data-folder="video"`.

### 💻 Script JavaScript
```
<script src="{{ url_for('static', filename='js/painel.js') }}"></script>
```

Inclui o arquivo JavaScript `painel.js`, que é responsável por:
- Manipular a lógica de exclusão de arquivos de mídia (imagens e vídeos) através dos atributos `data-*` nos ícones de lixeira.
- Interagir com a rota `/remover_item` no backend para efetivar a remoção.

# 👨‍💻 **Descrição dos scripts feitos em JavaScript**

## script.js

Este script JavaScript é o motor da interface de acompanhamento de pedidos, orquestrando a exibição dinâmica de conteúdo. Ele gerencia a alternância programada entre a **tabela de pedidos paginada** (com alertas sonoros para pedidos prontos para retirada) e a **exibição de mídias de campanhas** (imagens e vídeos), seguindo um roteiro pré-definido.

### ⚙️ Constantes e Varíaveis Globais

| Nome da Variável / Constante | Função  
|----------------------------- | ------- 
| `rowsPerPage`                | `const`: Define o número de linhas de pedidos a serem exeibidas por páginas na tabela.
| `tabelaDisplayTime`          | `const`: Duração, em milissegundos, que cada página da tabela permanece visível.
| `imagemDisplayTime`          | `const`: Duração, em milissegundos, que cada imagem de campanha permanece visível.
| `videoDisplayTime`           | `const`: Duração máxima, em milissegundos, que cada vídeo permanecerá visivel antes de avançar para o próximo estágio do roteiro (mesmo que o vídeo não tenha termiado).
| `tableBody`                  | `const`: Referência ao elemento `<tbody>` da tabela de pedidos.
| `tableHearder`               | `const`: Referência ao elmento `<thead>` da tabela de pedidos.
| `paginationDiv`              | `const`: Elemento `div` onde os indicadores de página da tabela são renderizados.
| `mediaSlidesContainer`       | `const`: Contêiner que engloba todos os slides de mídia (imgens e vídeos).
| `tituloPedidos`              | `const`: Referência ao elemento `<h1>` que exibe o título da seção de pedidos.
| `audioNotificacao`           | `const`: Referência ao elemento `<audio>` para reprodução do som de notificação.
| `allRows`                    | `let`: Array que armazena todas as linhas (`<tr>`) da tabela de pedidos para manipulação de paginação e ordenação.
| `allVideoMedia`              | `let`: Array de elementos DOM de todos os vídeos disponíveis no carrossel de mídia.
| `allImageMedia_Campaign1`    | `let`: Array de elementos DOM de todas as imagens da `campanha1`.
| `allImageMedia_Campaign2`    | `let`: Array de elementos DOM de todas as imagens da `campanha2`.
| `currentPage`                | `let`: Acompanha a página atual que está sendo exibida na tabela.
| `currentVideoIndex`          | `let`: Armazena o índice do vídeo a se exibido no array `allVideoMedia`. Este índece é **persistido no** `localStorage` para continuar a sequência após um recarregamento da página.
| `currentImageIndex`          | `let`: Acompanha o índece da imagem atual sendo exibida dentro de uma campanha específica.
| `sequenceIndex`              | `let`: Acompanha a posição atual no `displaySequence` (roteiro de exibição).
| `autoSwapInterval`           | `let`: Armazena o ID do timer (`setTimeout` ou `setInterval`) para controlar a alternância entre os conteúdos.
| `somEmitido`                 | `let`: Flag booleana para controlar se o som de notificação já foi emitido, evitando reprodução duplicadas.
| `displaySequence`            | `const`: Um array que define a **ordem exata** dos estágios de exibição. Seus valores são identificadores para os tipos de conteúdo a serem mostrados.
| `totalTablePages()`          | `const`: Uma função auxiliar que calcula o número total de páginas da tabela com base em `allRows.length` e `rowsPerPage`. Retorna no mínimo 1.

### 💻 Funções Auxiliares

#### `displaySequence`

Define um "roteiro" sequencial para o conteúdo. Cada item no array representa um estágio de exibição:

- `'TABLE_PAGE'`: Exibe a página atual da tabela de pedidos. O sistema avança para a próxima página da tabela a cada vez que este estágio é alcançado.
- `'VIDEO'`: Exibe um único vídeo do array `allVideoMedia`. A sequência avança quando o vídeo termina ou após `videoDisplayTime`. O índice do próximo vídeo a ser exibido é salvo no `localStorage`.
- `'CAMPAIGN_1'`: Inicia a exibição sequencial de **todas as imagens** da `campanha1`.
- `'CAMPAIGN_2'`: Inicia a exibição sequencial de **todas as imagens** da `campanha2`.
- `'TABLE_REMAINDER'`: Exibe as páginas restantes da tabela que ainda não foram mostradas no ciclo atual, até que todas as páginas tenham sido percorridas.
- `'RELOAD'`: Salva o estado atual (próximo vídeo) e recarrega a página. Isso reinicia o roteiro do início, mas com a continuidade de onde o último vídeo parou.

#### `emitirSomSeHouverRetirar()`

Percorre todas as linhas da tabela. Se encontrar uma linha com `data-retirar="sim"` e o som ainda não foi emitido (`somEmitido` for `false`), ele reproduz o áudio de notificação. A flag `somEmitido` é então definida como `true` para evitar repetições. Se nenhuma linha de retirada for encontrada, `somEmitido` é resetada para `false`.

---

#### `loadTableData()`

Inicializa a gestão dos dados da tabela:

1. Popula o array `allRows` com todas as linhas do `tableBody`.
2. Chama `ordenarTabela()` para aplicar a ordem de prioridade.
3. Chama `setupCarouselIndicators()` para configurar os indicadores de paginação.

#### `showPage(page)`

Responsável por exibir uma página específica da tabela:

1. Calcula os índices de início e fim das linhas com base na `page` e `rowsPerPage`.
2. Oculta todas as linhas da tabela.
3. Remove a classe `hidden` das linhas que pertencem à página atual, tornando-as visíveis.
4. Chama `updateActiveIndicator()` para destacar o indicador de paginação correspondente.
5. Chama `emitirSomSeHouverRetirar()` para verificar a necessidade de reproduzir o áudio.

#### `setupCarouselIndicators()`

Cria e renderiza dinamicamente os indicadores de paginação na `paginationDiv`. Cada indicador é um `<span>` que visualmente representa uma página da tabela.

#### `updateActiveIndicator()`

Atualiza o estado visual dos indicadores de paginação, adicionando ou removendo a classe `active` para destacar o indicador da `currentPage` atual.

#### `ordenarTabela()`

Ordena as linhas da tabela (`allRows`) com base na prioridade definida pela cor (status do pedido), colocando os pedidos "prontos para retirada" (verdes) no topo, seguidos por "em conferência" (azul), "separados" (amarelo), "em separação" (laranja) e "a separar" (vermelho) por último. Após a ordenação, as linhas são removidas e reinseridas no `tableBody` na nova ordem.

#### `hideAllElements()`

Uma função utilitária que oculta todos os elementos principais da interface (cabeçalho e corpo da tabela, paginação, título de pedidos e o contêiner de slides de mídia). Também pausa e reinicia o tempo de todos os vídeos dentro do contêiner de mídia, garantindo que eles comecem do zero na próxima vez que forem exibidos.

#### `showTableElements()`

Exibe os elementos relacionados à tabela de pedidos (cabeçalho, corpo, paginação e título), garantindo que os elementos de mídia estejam ocultos.

#### `showImageElements(index, mediaArray)`

Exibe um item de mídia específico (imagem ou vídeo) do `mediaArray` no `index` fornecido:

1. Oculta todos os outros elementos da interface.
2. Torna o `mediaSlidesContainer` visível e ativa o `mediaItem` correspondente ao `index`.
3. Lógica de Vídeo e Tela Cheia:
    - Se o item for um vídeo, ele tenta reproduzi-lo.
    - Além disso, tenta colocar o `mediaSlidesContainer` em modo de tela cheia usando a API Fullscreen do navegador, proporcionando uma experiência de visualização imersiva para vídeos.

### 🔁 Função de Lógica de Exibição

#### `advanceSequence()`

Esta função é a chave para o avanço no roteiro de exibição. Ela incrementa o `sequenceIndex` para o próximo estágio no `displaySequence` e chama `cycleDisplay()` para executar o novo estágio.

#### `playCampaign(campaignMediaArray)`

Gerencia a exibição sequencial de imagens de uma campanha:

1. Verifica se a `campaignMediaArray` está vazia. Se estiver, pula para o próximo estágio do roteiro.
2. Reseta `currentImageIndex` para 0.
3. Define uma função interna `showNextImage()` que:
    - Exibe a imagem atual usando `showMediaElement()`.
    - Incrementa `currentImageIndex`.
    - Agenda a próxima imagem via `setTimeout` com `imagemDisplayTime`.
    - Quando todas as imagens da campanha forem exibidas, chama `advanceSequence()` para ir para o próximo estágio do roteiro.
4. Inicia a exibição chamando `showNextImage()`.

#### `playTableRemainder()`

Responsável por exibir as páginas restantes da tabela até que todas as páginas tenham sido vistas, ou até que um ciclo completo seja concluído:

1. Armazena a `currentPage` inicial.
2. Define uma função interna `showNextPage()` que:
    - Exibe os elementos da tabela e a `currentPage`.
    - Avança `currentPage` para a próxima página (`% totalTablePages()`).
    - A condição de parada ocorre quando todas as páginas foram exibidas (retornando à `startPage` ou se houver apenas uma página).
    - Agenda a próxima página via `setTimeout` com `tabelaDisplayTime`.
3. Inicia a exibição chamando `showNextPage()`.

### 🔁 Função Principal de Ciclo (`cycleDisplay()`)

Esta é a função central que orquestra todo o processo de exibição, baseando-se no `displaySequence`:

1. Limpa qualquer timer (`setTimeout` ou `setInterval`) anterior para evitar execuções sobrepostas.
2. Verifica se o `sequenceIndex` excedeu o tamanho do roteiro e, se sim, tenta forçar um recarregamento.
3. Obtém o `currentStage` do `displaySequence`.
4. Utiliza uma estrutura `switch` para executar a lógica correspondente a cada tipo de estágio:
    - `'TABLE_PAGE'`: Exibe a página atual da tabela, avança a `currentPage` e agenda a `advanceSequence` após `tabelaDisplayTime`.
    - `'VIDEO'`: Exibe o vídeo em `currentVideoIndex`. Salva o `currentVideoIndex` no `localStorage` para a próxima vez. A sequência avança quando o vídeo termina (`onended`) ou após `videoDisplayTime`, o que ocorrer primeiro.
    - `'CAMPAIGN_1'`: Chama `playCampaign()` para iniciar a exibição da `campanha1`.
    - `'CAMPAIGN_2'`: Chama `playCampaign()` para iniciar a exibição da `campanha2`.
    - `'TABLE_REMAINDER'`: Chama `playTableRemainder()` para exibir o restante das páginas da tabela.
    - `'RELOAD'`: Recarrega a página. Isso permite que o roteiro seja reiniciado do começo, enquanto a persistência do `currentVideoIndex` via `localStorage` mantém a continuidade dos vídeos.

### 🎯 Função de Inicialização: `startAutoSwap()`

Esta função é o ponto de partida do script, sendo chamada assim que o DOM está pronto:

1. Carrega o estado do vídeo: Recupera o `currentVideoIndex` do `localStorage` (ou define como 0 se não encontrado).
2. Popula arrays de mídia: Preenche `allVideoMedia`, `allImageMedia_Campaign1` e `allImageMedia_Campaign2` com os elementos de mídia correspondentes.
3. Chama `loadTableData()` para preparar a tabela.
4. Define `sequenceIndex` como 0 (início do roteiro).
5. Inicia o ciclo de exibição chamando `cycleDisplay()`.

### 🖥️ Evento `DOMContentLoaded`

Este é o ponto de entrada principal do script, garantindo que o código seja executado somente após o carregamento completo do DOM:

1. **Verificações Essenciais**: Realiza uma verificação crucial para garantir que todos os elementos HTML necessários estejam presentes na página. Se algum elemento faltar, um erro é registrado no console e o script é interrompido para evitar falhas.
2. Chama a função `start()` para iniciar todo o processo de inicialização e o ciclo de exibição.

## painel.js

Este script JavaScript é responsável por habilitar a funcionalidade de **exclusão de itens de mídia** (tanto imagens quanto vídeos) diretamente do painel de gerenciamento. Ele faz isso ao adicionar ouvintes de evento de clique aos ícones de lixeira visíveis na interface. Quando um desses ícones é clicado, o script coleta as informações necessárias sobre o item de mídia e redireciona o navegador para uma URL específica no backend, que é responsável por realizar a remoção.

### ⚙️ Estrutura e Funcionamento Detalhado
O script opera dentro do evento `DOMContentLoaded`, garantindo que todo o HTML da página esteja completamente carregado e acessível antes que qualquer manipulação de elementos ocorra.

1. **Seleção de Elementos de Remocação**
```
const removerMediaButtons = document.querySelectorAll('.remover-media');
```

Ao carregar a página, o script seleciona todos os elementos que possuem a classe CSS `.remover-media`. No contexto do painel, esses são os ícones de lixeira (`<i>`) associados a cada imagem e vídeo que o usuário pode gerenciar. A variável `removerMediaButtons` armazena uma `NodeList` (semelhante a um array) desses elementos.

2. **Adição de Evento de Clique**
```
removerMediaButtons.forEach(button => {
    button.addEventListener('click', function(event) {
        // ... lógica de remoção ...
    });
});
```

O script então percorre cada botão de remoção encontrado na `removerMediaButtons` e adiciona um **ouvinte de evento de clique** a cada um. Isso significa que, sempre que um usuário clica em um ícone de lixeira, a função definida dentro do `addEventListener` será executada.

3. **Prevenção do Comportamento Padrão**
```
event.preventDefault(); // Evita qualquer comportamento padrão do elemento
```

Dentro da função de clique, `event.preventDefault()` é chamado. Esta é uma etapa crucial que **interrompe qualquer ação** padrão que o navegador executaria para o elemento clicado (por exemplo, se fosse um link `<a>`, ele impediria a navegação para o `href`). Isso dá ao script controle total sobre a ação de remoção.

4. **Obtenção de Atributos de dados**(`data-*`)
```
const filename = this.dataset.filename;
const fileType = this.dataset.type;   // 'image' ou 'video'
const folderName = this.dataset.folder; // 'campanha1', 'campanha2' ou 'video'
```

Ao invés de depender apenas do nome do arquivo, o script agora extrai múltiplos atributos `data-*` do elemento que foi clicado (`this` refere-se ao próprio botão/ícone clicado). Esses atributos são:

- `filename`: O nome completo do arquivo a ser removido (ex: `minha_foto.jpg`, `meu_filme.mp4`).
- `fileType`: Indica o tipo de mídia (`'image'` para imagens ou `'video'` para vídeos).
- `folderName`: Especifica a pasta onde o arquivo está localizado. Para imagens, será o nome da campanha (ex: `'campanha1'`, `'campanha2'`). Para vídeos, o valor é `'video'`.
- Esses dados são **essenciais** para que o backend possa localizar e remover o arquivo correto de forma segura e precisa.

5. **Redirecionamento para o Endpoint de Remoção**
```
if (filename && fileType && folderName) {
    window.location.href = `/remover_item?filename=${filename}&file_type=${fileType}&folder=${folderName}`;
} else {
    // ... tratamento de erro ...
}
```

Após coletar as informações necessárias, o script verifica se todos os dados (`filename`, `fileType`, `folderName`) estão presentes. Se estiverem, ele redireciona o navegador para a URL `/remover_item`. Os dados coletados são passados como parâmetros de consulta na URL.

É o **backend da aplicação** que deve ter uma rota configurada para `/remover_item`. Esta rota receberá os parâmetros e será responsável por:
    - Validar a requisição.
    - Localizar o arquivo correspondente no servidor, utilizando o nome do arquivo, seu tipo e a pasta.
    - Remover fisicamente o arquivo do sistema de arquivos do servidor.
    - Gerenciar a resposta ao usuário (por exemplo, redirecionar de volta para o painel ou exibir uma mensagem de sucesso/erro).

6. **Tratamento de Erro (Local no Frontend)**
```
else {
    console.error('Dados incompletos para remover o item:', { filename, fileType, folderName });
    alert('Erro: Não foi possível obter todas as informações necessárias para remover o arquivo.');
}
```

Caso um ou mais dos atributos `data-*` (filename, fileType, folderName) estejam ausentes no elemento clicado, o script impede o redirecionamento. Em vez disso, ele registra uma mensagem de erro detalhada no console do navegador e exibe um alerta amigável ao usuário, indicando que a remoção não pôde ser processada devido à falta de informações.

### 📂 Requisitos para Funcionamento Adequado
Para que o script `painel.js` funcione corretamente, é fundamental que os elementos de ícone de lixeira no seu `painel.html` sigam esta estrutura, incluindo os atributos de dados (`data-*`) necessários:

```
<i class="fa-solid fa-trash remover-media"
   data-filename="minha_foto_legal.jpg"
   data-type="image"
   data-folder="campanha1">
</i>

<i class="fa-solid fa-trash remover-media"
   data-filename="video_institucional.mp4"
   data-type="video"
   data-folder="video">
</i>
```

Isso garante que o JavaScript possa identificar corretamente qual arquivo remover e de qual local
