# 🚖 Corrida+ (Gestão Financeira para Motoristas TVDE)

O **Corrida+** é um aplicativo web focado na gestão financeira de motoristas de aplicativo (Uber, Bolt e corridas particulares). Projetado com foco na experiência mobile, ele funciona como uma SPA (*Single Page Application*) com visual nativo de aplicativo (estilo iOS).

O grande diferencial do projeto é sua arquitetura híbrida: ele roda de forma 100% instantânea salvando os dados no dispositivo através do `localStorage` e utiliza uma planilha do **Google Sheets** como banco de dados em nuvem gratuito e sem servidor (Serverless), permitindo sincronizar os seus dados entre vários aparelhos (ex: celular e tablet) em tempo real.

---

## ✨ Funcionalidades Implementadas

### 🚗 Ganhos por Plataforma
* **Registro Multi-Plataforma:** Lance os ganhos separadamente por Uber, Bolt, corridas Particulares e Outros (entregas, bicos extras, etc).
* **Visão Diária, Semanal e Mensal:** Acompanhe o faturamento bruto em qualquer período, com navegação rápida entre datas.
* **Total Bruto Consolidado:** Veja de imediato o somatório de tudo o que entrou (TVDE + Pessoal) num único card em destaque.

### 💸 Despesas Fixas e Variáveis
* **Despesas Fixas Recorrentes:** Cadastre contas mensais (aluguel, financiamento, internet, etc.) que se repetem automaticamente mês a mês.
* **Despesas Variáveis:** Lance combustível, manutenção, alimentação e outras despesas do dia a dia, por categoria.
* **Dados Ocultos e Limpeza:** No Modo Dev, é possível visualizar e excluir despesas fixas ocultadas ou registros antigos sem perder o histórico dos demais meses.

### 🧮 Cálculo Automático de IVA e Comissão
* **IVA Configurável:** Defina a sua taxa de IVA (%) nas configurações — o app desconta automaticamente do faturamento bruto das apps (Uber/Bolt).
* **Comissão da Plataforma:** Defina também a taxa de comissão cobrada pela própria Uber/Bolt, descontada logo após o IVA.
* **Detalhamento Completo:** A aba Lucro mostra o passo a passo exato: Bruto → IVA → Comissão → Líquido TVDE → + Particular/Outros → − Combustível → − Despesas fixas → − Outras variáveis → **Lucro líquido final**.

### 📊 Estatísticas e Comparativos
* **Gráficos Ganhos vs. Despesas:** Visualize por dia, semana ou mês, com barras lado a lado para comparar entrada e saída de dinheiro.
* **Separação TVDE vs. Pessoal:** Estatísticas mostram claramente o que veio de Uber/Bolt (sujeito a IVA e comissão) e o que veio de corridas particulares (sem desconto).
* **Evolução dos Últimos Meses:** Acompanhe a tendência de lucro ao longo do tempo.

### 🛠️ Modo Dev (Avançado)
* **Acesso Oculto:** Toque 5 vezes no número da versão, nas configurações, para ativar o Modo Dev.
* **Dados Ocultos e Limpeza:** Visualize e exclua registros (ganhos, despesas, contas fixas ocultadas) de qualquer mês.
* **Importar Dados:** Cole um JSON de backup para restaurar seus dados manualmente, caso a sincronização automática falhe.

---

## 🚀 Como Configurar o Banco de Dados (Google Sheets)

Para sincronizar os seus dados entre aparelhos, precisamos criar a planilha e **gerar o link de sincronização**. Siga o passo a passo com atenção:

### Passo 1: Criar a Planilha e Inserir o Código
1. Acesse o [Google Sheets](https://sheets.google.com) e crie uma planilha em branco (ex: `BD_CorridaPlus`).
2. No menu superior da planilha, clique em **Extensões** > **Apps Script**.
3. Apague todo o código que estiver na tela e cole este bloco abaixo:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = e.postData.contents; 
    
    sheet.getRange('A1').setValue(data);
    sheet.getRange('B1').setValue("Última sincronização: " + new Date().toLocaleString("pt-BR"));
    
    return ContentService.createTextOutput(JSON.stringify({"status": "sucesso"})).setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({"erro": error.toString()})).setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = sheet.getRange('A1').getValue(); 
  if (!data) { data = "{}"; }
  return ContentService.createTextOutput(data).setMimeType(ContentService.MimeType.JSON);
}
```

4. Clique no ícone de **Salvar** (disquete) no menu superior.

### Passo 2: Como Gerar e Pegar o Link (Atenção Aqui!)
Este é o momento de criar a URL que o seu aplicativo vai usar.
1. No canto superior direito do Apps Script, clique no botão azul **Implantar** e escolha **Nova implantação**.
2. Clique no ícone de engrenagem (`⚙️`) ao lado de "Selecione o tipo" e escolha **App da Web**.
3. Preencha os campos **EXATAMENTE** desta forma:
   * **Descrição:** Pode escrever "Versão 1".
   * **Executar como:** *Você (seu-email@gmail.com)*
   * **Quem tem acesso:** Selecione **Qualquer pessoa** *(Se não marcar "Qualquer pessoa", o celular não vai conseguir salvar)*.
4. Clique no botão azul **Implantar**.
5. Vai aparecer um aviso de "Autorização necessária". Clique em **Autorizar acessos** e escolha o seu e-mail.
6. O Google mostrará uma tela dizendo "O Google não verificou este app". Clique na palavra **Avançado** (lá embaixo) e depois clique em **Acessar projeto sem título (não seguro)**. Clique em **Permitir**.
7. Na última tela que aparecer, você verá escrito "URL do app da Web" e um link gigante embaixo (terminando com `/exec`). **Copie este link gigante.**

### Passo 3: Colar o Link no Aplicativo
1. Abra o site do seu aplicativo (o link do GitHub Pages) no seu celular.
2. Navegue até a aba inferior direita chamada **Você** (Configurações).
3. Procure o campo **"URL de Sincronização (Google Sheets)"** e cole o link gigante lá dentro.
4. Defina a sua taxa de **IVA** e de **Comissão** nas configurações.
5. Pronto! Faça o mesmo em outro aparelho com o mesmo link para manter os dados sincronizados.

---

## 🛠️ Performance e Sincronização Manual
Para não estourar o limite de acessos diários do Google:
* O app salva localmente e só envia dados para a nuvem alguns segundos após você terminar de fazer uma alteração.
* Ele puxa dados novos automaticamente apenas quando o aplicativo é aberto.
* **Quer puxar atualizações manualmente?** Basta tocar na pílula escrita "sincronizado" ou "offline" no topo do app para forçar a atualização da tela naquele momento.

---

## 📱 Dica: Instale no Celular como um App Real
Abra o site no navegador do celular (Safari ou Chrome), clique no botão de **Compartilhar** do navegador e escolha **"Adicionar à Tela de Início"**. O navegador ocultará as barras de endereço e o ícone ficará junto com seus outros aplicativos.
