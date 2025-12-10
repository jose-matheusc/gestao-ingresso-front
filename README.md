# Sistema de Gestão de Compras e Ingressos

> **Observação:** este repositório é o **front-end** do sistema. O back-end (APIs/Java) está em outro repositório/projeto separado.

## Repositórios do projeto

- **Frontend (este repo):** interface web estática com HTML/CSS/JS, telas de login, cadastro, produtos, carrinho e pagamento.
- **Backend (outro repo):** expõe as APIs REST consumidas por este frontend (login, clientes, produtos, carrinho, pagamento).

> - Git: `https://github.com/jose-matheusc/gestao-compras-ingresso`

## Sumário

- [Arquitetura](#arquitetura)
- [Páginas do projeto](#páginas-do-projeto)
- [Como rodar o frontend](#como-rodar-o-frontend)
- [Integração com APIs (backend externo)](#integração-com-apis-backend-externo)
- [Fluxo de uso da aplicação](#fluxo-de-uso-da-aplicação)
- [Estrutura de diretórios](#estrutura-de-diretórios)
- [Ajustes comuns](#ajustes-comuns)

## Arquitetura

- **Frontend**: HTML + CSS + JavaScript puro
  - Não há framework (React, Angular, etc.).
  - As páginas são estáticas e se comunicam com APIs REST via `fetch`.
  - O estado de autenticação é guardado em `localStorage` com a chave `usuarioLogado`.

- **Backend / APIs**: **não fazem parte deste repositório**
  - O front foi configurado para consumir endpoints em um servidor HTTP (por padrão `http://localhost:8080/api`), mas você pode alterar isso.
  - As seções abaixo descrevem os endpoints **esperados**, apenas como contrato.

## Páginas do projeto

- `login.html`
  - Tela de login com **email + senha**.
  - Faz `POST` para o endpoint de login (por padrão `/api/clientes/login`).
  - Em caso de sucesso, o cliente retornado é salvo no `localStorage` e o usuário é redirecionado para `index.html`.

- `cadastro_cliente.html`
  - Tela de cadastro de cliente com campos: nome, email, senha, telefone, CPF.
  - Faz `POST` para o endpoint de cadastro de cliente (por padrão `/api/clientes`).
  - Em caso de sucesso, o cliente retornado é salvo no `localStorage` e o usuário é redirecionado para `index.html`.

- `index.html`
  - Lista de produtos disponíveis (ingressos).
  - Carrega produtos via `GET` em `/api/produtos`.
  - Permite adicionar produtos ao carrinho via `POST` em `/api/carrinho/adicionar`.
  - Só permite acesso se existir `usuarioLogado` no `localStorage`.

- `carrinho.html`
  - Mostra os itens do carrinho e o total.
  - Consome `GET /api/carrinho` (itens) e `GET /api/carrinho/total` (valor total).
  - Só permite acesso se existir `usuarioLogado` no `localStorage`.

- `pagamento.html`
  - Tela de pagamento com seleção de método (Cartão de Crédito, Pix, Boleto).
  - Carrega o total via `GET /api/carrinho/total`.
  - Envia `POST /api/pagamento` com os dados do pagamento.
  - Se o pagamento for bem-sucedido, faz **logout automático** (remove `usuarioLogado` e redireciona para `login.html`).

- `cadastro_produto.html`
  - Tela de cadastro de produto (nome, preço, categoria, descrição).
  - Envia `POST /api/produtos`.

## Como rodar o frontend

Este projeto é apenas HTML estático. Você pode rodar de duas formas:

### 1. Abrir os arquivos HTML diretamente

1. Vá até a pasta raiz do projeto (`ingressos/`).
2. Dê duplo clique em `login.html` para abrir no navegador.
3. Use o menu de navegação para ir para as outras páginas (`cadastro_cliente.html`, `index.html`, etc.).

> Dica: abrindo como `file:///...` as requisições `fetch` para um backend local ainda funcionarão, desde que o backend esteja acessível via HTTP (por exemplo, `http://localhost:8080`).

### 2. Usar um servidor estático (recomendado)

Se estiver usando VS Code com a extensão **Live Server**:

1. Abra a pasta `ingressos/` no VS Code.
2. Clique com o botão direito em `login.html` > **Open with Live Server**.
3. O front ficará em algo como `http://127.0.0.1:5500/login.html`.

Os scripts de frontend fazem as requisições de API sempre para a URL definida em cada arquivo (`const API_BASE = 'http://localhost:8080/api';` por padrão), então frontend e backend podem rodar em portas diferentes sem problemas.

## Integração com APIs (backend externo)

### URL base de API

Nos arquivos HTML existe uma constante:

```js
const API_BASE = 'http://localhost:8080/api';
```

Se o seu backend estiver em outro host/porta/caminho, basta alterar essa constante em **todos os arquivos HTML** para a URL desejada.

### Endpoints esperados pelo frontend

> Estes endpoints são apenas o **contrato esperado** pelo JavaScript do front. A implementação real está no seu outro repositório/projeto.

#### Clientes

- `POST ${API_BASE}/clientes`
  - Cadastro de novo cliente.
  - Corpo esperado (JSON):
    ```json
    {
      "nome": "string",
      "email": "string",
      "senha": "string",
      "telefone": "string",
      "cpf": "string"
    }
    ```
  - Retorna o cliente salvo (objeto JSON), usado para preencher `localStorage.usuarioLogado`.

- `POST ${API_BASE}/clientes/login`
  - Autenticação de cliente.
  - Corpo esperado (JSON):
    ```json
    {
      "email": "string",
      "senha": "string"
    }
    ```
  - **200 OK**: retorna o cliente autenticado (`ClienteDTO` ou similar).
  - **401 Unauthorized**: email ou senha inválidos.

#### Produtos

- `GET ${API_BASE}/produtos`
  - Retorna a lista de produtos disponíveis.

- `POST ${API_BASE}/produtos`
  - Cadastra um novo produto (usado em `cadastro_produto.html`).

#### Carrinho

- `POST ${API_BASE}/carrinho/adicionar`
  - Adiciona um item ao carrinho.

- `GET ${API_BASE}/carrinho`
  - Lista os itens do carrinho atual.

- `GET ${API_BASE}/carrinho/total`
  - Retorna o valor total do carrinho.

#### Pagamento

- `POST ${API_BASE}/pagamento`
  - Recebe os dados do pagamento e processa.
  - Corpo esperado (JSON):
    ```json
    {
      "metodo": "Cartão de Crédito | Pix | Boleto",
      "dados": "string",
      "valorTotal": 123.45
    }
    ```
  - **200 OK**: pagamento aprovado (texto simples na resposta).

## Fluxo de uso da aplicação

### 1. Cadastro de cliente

1. Abra `cadastro_cliente.html`.
2. Preencha **nome**, **email**, **senha**, telefone, CPF.
3. Ao clicar em **Cadastrar Cliente**:
   - O frontend envia um `POST ${API_BASE}/clientes` com os dados do cliente.
   - Em caso de sucesso, o cliente retornado é salvo no `localStorage` como `usuarioLogado`.
   - O usuário é redirecionado para `index.html` e já está logado.

### 2. Login

1. Abra `login.html`.
2. Informe **email** e **senha** cadastrados.
3. Ao clicar em **Entrar**:
   - O frontend envia um `POST ${API_BASE}/clientes/login` com `{ email, senha }`.
   - Se as credenciais estiverem corretas (`200 OK`), o cliente retornado é salvo em `localStorage`.
   - O usuário é redirecionado para `index.html`.

### 3. Proteção de telas

- `index.html`, `carrinho.html` e `pagamento.html` verificam se existe `usuarioLogado` em `localStorage`.
- Caso não exista:
  - É exibido um alerta informando que é necessário fazer login.
  - O usuário é redirecionado para `login.html`.

### 4. Produtos e carrinho

- Na `index.html`:
  - Logo no carregamento, é chamada a API `GET ${API_BASE}/produtos`.
  - Os produtos são exibidos em cards com botão **Adicionar ao Carrinho**.
  - Ao adicionar, é feito um `POST ${API_BASE}/carrinho/adicionar` com os dados do item.

- Em `carrinho.html`:
  - São chamados `GET ${API_BASE}/carrinho` (para listar itens) e `GET ${API_BASE}/carrinho/total` (para o valor total).

### 5. Pagamento e logout automático

- Em `pagamento.html`:
  - Na abertura da página, o total do carrinho é carregado via `GET ${API_BASE}/carrinho/total`.
  - Ao enviar o formulário de pagamento:
    - É feito `POST ${API_BASE}/pagamento` com `{ metodo, dados, valorTotal }`.
    - Se o pagamento der certo (`200 OK`):
      - É mostrado um alerta de sucesso.
      - O `usuarioLogado` é removido do `localStorage` (logout automático).
      - O usuário é redirecionado para `login.html`.

### 6. Logout manual

- Em `login.html` existe um botão **Sair** no menu.
- Ao clicar em **Sair**:
  - O `usuarioLogado` é removido do `localStorage`.
  - O usuário permanece/volta para `login.html`.

## Estrutura de diretórios

```text
ingressos/
├─ index.html
├─ login.html
├─ cadastro_cliente.html
├─ cadastro_produto.html
├─ carrinho.html
├─ pagamento.html
├─ assets/
│  └─ style.css
└─ README.md
```

## Ajustes comuns

### 1. Trocar a URL base das APIs

Se o backend não estiver em `http://localhost:8080/api`, edite nos arquivos HTML os trechos:

```js
const API_BASE = 'http://localhost:8080/api';
```

Colocando a URL correta do seu backend (por exemplo, `https://meu-servidor.com/api`).

### 2. Erros de CORS (quando usar backend externo)

Se o navegador bloquear requisições por CORS (principalmente ao usar `127.0.0.1:5500` no front e outro host/porta no backend), será necessário **habilitar CORS no backend**.

Como este repositório não contém o backend, a configuração deve ser feita **no projeto de backend** (por exemplo, no seu projeto Spring Boot ou Node.js).

### 3. Campos de senha

Certifique-se de que o backend que você está usando:

- Possui o campo `senha` na entidade/DTO de cliente.
- No cadastro (`POST /clientes`), salva a senha corretamente.
- No login (`POST /clientes/login`), valida **email e senha**.

Se você trocar o formato desses campos no backend, ajuste também os `fetch` no frontend para enviar/ler o JSON no formato correto.
