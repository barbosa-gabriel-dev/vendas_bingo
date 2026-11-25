# 🎱 Sistema de Geração de Cartelas e Gestão de Vendas de Bingo (PWA)

Sistema completo para geração de cartelas de bingo (padrão **75 bolas com centro livre**), diagrama de impressão paginado em folha A4 (**4, 5 ou 6 cartelas iguais por folha**, referentes aos prêmios) e aplicativo web progressivo (**PWA**) para controle de vendas em campo, baixa de pagamentos, atribuição de faixas a vendedores e conferência de autenticidade.

---

## 🏗️ Arquitetura do Projeto

O projeto utiliza uma arquitetura *serverless* de baixo custo e fácil manutenção:

* **Banco de Dados & Gerador de Impressão:** Google Sheets (`Config`, `Cartelas_Base`, `Vendas`, `Vendedores` e `Impressao`).
* **Backend / API REST:** Google Apps Script (`Codigo.gs`) implantado como Web App.
* **Frontend Mobile (PWA):** HTML5, CSS3 e JavaScript puro (`index.html`, `manifest.json`, `sw.js`) hospedado no **GitHub Pages**, instalável na tela inicial do celular.

---

## 📁 Estrutura de Arquivos

```text
├── Codigo.gs        # Script do Google Apps Script (gerador da planilha, impressão A4 e API)
├── index.html       # Interface do PWA (Login, Vendas, Lista, Conferência e Painel Admin)
├── manifest.json    # Manifesto PWA para instalação no Android/iOS
├── sw.js            # Service Worker para cache offline de assets estáticos
└── README.md        # Documentação de implantação e operação
```

---

## 🚀 Passo a Passo de Implantação

### 1. Configurar a Planilha e o Backend (`Codigo.gs`)

1. Crie uma planilha em branco no **Google Sheets**.
2. Acesse **Extensões > Apps Script**, apague o conteúdo padrão e cole o código completo do arquivo **`Codigo.gs`**.
3. Salve o projeto (`Ctrl + S`).
4. Na barra superior do editor, selecione a função **`setupPlanilhaBingo`** e clique em **Executar** (autorize as permissões solicitadas na primeira execução).
   * Todas as abas (`Config`, `Cartelas_Base`, `Vendas`, `Vendedores` e `Impressao`) serão criadas e formatadas automaticamente.
5. Para publicar a API do PWA:
   * Clique em **Implantar > Nova implantação**.
   * Selecione o tipo **App da Web**.
   * Configure **Executar como:** `Eu` e **Quem pode acessar:** `Qualquer pessoa`.
   * Clique em **Implantar** e copie a **URL do App da Web** (terminada em `/exec`).

> **Nota sobre atualizações:** Sempre que alterar o `Codigo.gs`, vá em **Implantar > Gerenciar implantações > Editar (ícone de lápis) > Versão: Nova versão > Implantar** para atualizar a URL `/exec` ativa.

---

### 2. Configurar e Publicar o PWA (`GitHub Pages`)

1. Abra o arquivo **`index.html`** e localize a constante `URL_PADRAO_API` no início da tag `<script>`:
   ```javascript
   const URL_PADRAO_API = "https://script.google.com/macros/s/SEU_ID_AQUI/exec";
   ```
2. Cole a URL `/exec` copiada no passo anterior.
3. Suba os arquivos `index.html`, `manifest.json`, `sw.js` e `README.md` para o seu repositório no GitHub.
4. No repositório, acesse **Settings > Pages**, selecione a branch principal (`main` / `root`) e salve.
5. Abra o link do GitHub Pages no navegador do celular e toque em **Adicionar à tela inicial** para instalar o app.

---

## 🖨️ Geração e Impressão das Cartelas

1. Na aba **`Config`** da planilha, ajuste:
   * **Nome do Bingo:** Título exibido no topo de cada folha (caso não utilize imagem).
   * **URL da Imagem do Cabeçalho (Opcional):** Link direto (`.png`/`.jpg`) ou link compartilhado do Google Drive ("Qualquer pessoa com o link") para inserir um banner/logo no cabeçalho.
   * **Cartelas por Página:** Selecione `4`, `5` ou `6` cartelas idênticas por página (`1º PRÊMIO`, `2º PRÊMIO`, `3º PRÊMIO`...).
   * **Quantidade de Páginas:** Quantidade inicial de compradores/blocos a gerar.
   * **Prefixo do Código:** Prefixo utilizado no código único de validação no rodapé de cada folha (ex.: `GBN-0001-756D51`).
2. Execute **🎱 Sistema Bingo > 2. Gerar Cartelas e Montar Impressão** (ou gere lotes incrementais diretamente pelo painel **Admin** no PWA).
3. Para imprimir a aba **`Impressao`**:
   * **Tamanho do papel:** A4 (`21 cm x 29,7 cm`)
   * **Orientação:** Retrato
   * **Escala:** `Ajustar à largura`
   * Cada comprador/bloco ocupa exatamente **1 página A4** (47 linhas calibradas) com o número sequencial da cartela e o código único no rodapé.

---

## 📱 Funcionalidades por Perfil de Acesso

### 👤 Perfil Vendedor
* **Login Persistente:** Autentica uma única vez no aparelho usando `Login` e `PIN` cadastrados na aba `Vendedores`.
* **Registrar Venda (`➕ Vender`):** Registra o número da cartela, nome do comprador e status inicial (`Pago` ou `Pendente`). Permite vender apenas cartelas atribuídas ao próprio vendedor.
* **Minhas Vendas (`📋 Vendidos`):** Lista apenas as vendas do vendedor logado, com filtro por status (`Todos`, `Pendentes`, `Pagos`), busca por número/comprador, **Edição**, **Exclusão** e **Baixa de Pagamento** (com modal de confirmação: *"Confirma o pagamento de [nome_comprador]?"*).
* **Conferência Rápida (`🔎 Conferir`):** Valida o código único de marcação ou número da cartela contra a base oficial, exibindo comprador, status de pagamento e a grade $5 \times 5$ sorteada.

### 🛡️ Perfil Administrador (`Admin`)
* Todas as funcionalidades do vendedor (sem restrição de faixa na venda).
* **Visão Global de Vendas:** Filtra vendas de todos os vendedores ou de um vendedor específico.
* **Aba Gestão (`🎟️ Cartelas`):**
  * **Lotes Incrementais:** Gera novos lotes de cartelas (ex.: `+100`, `+50`) dando sequência automática ao último número da base sem repetir matrizes.
  * **Atribuir e Retirar Faixas:** Atribui intervalos de cartelas (ex.: `0001` a `0050`) a um vendedor ou retira cartelas não vendidas de volta para `Disponível`, sincronizando a coluna `Faixas_Atribuidas` na aba `Vendedores`.
  * **Gestão de Equipe (Modal):** Cadastra novos vendedores/administradores, edita nome, PIN, perfil e ativa/inativa acessos.

---

## 🔄 Iniciar um Novo Evento de Bingo

Para zerar as vendas e cartelas ao encerrar um evento (mantendo suas configurações e o cadastro de vendedores intactos):

1. Abra a planilha no computador.
2. No menu superior, acesse **🎱 Sistema Bingo > 3. Zerar Vendas e Cartelas (Novo Evento)** e confirme o alerta.
   *(Por segurança contra acidentes irreversíveis, essa função fica disponível exclusivamente na planilha e não aparece no aplicativo mobile.)*
