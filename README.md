[README.md](https://github.com/user-attachments/files/35308295/README.md)[Uploadin# Leitor de Comprovantes

Aplicativo web (PWA) que lê fotos de notas, faturas e recibos com inteligência artificial, organiza os gastos e avisa dos próximos vencimentos de contas mensais e parceladas. Funciona no navegador do computador e do celular e pode ser instalado como app.

Os dados ficam **somente no navegador** de quem usa. Não existe servidor nem banco de dados.

## O que ele faz

- **Leitura por IA (Google Gemini):** identifica o estabelecimento, a data, cada item pago, o tipo de pagamento (compra, conta de consumo, boleto, Pix, serviço…) e a categoria de gasto.
- **Várias fotos de uma vez:** fila de até 20 imagens, com progresso, resultado de cada foto e tentativa de novo apenas das que falharam. Fotos repetidas (mesmo local, data e valor) são ignoradas.
- **Vencimentos e avisos:** a IA lê o vencimento e se a conta é mensal ou parcelada (ex.: parcela 3/10). Os lembretes aparecem em *Próximos vencimentos*, com o botão **Paguei** para avançar para o mês seguinte.
- **Resumo e gráficos:** total de gastos, divisão por categoria e por tipo de pagamento, e gastos por mês. Saques e depósitos não entram nos gastos.
- **Edição:** corrija estabelecimento, data, categoria, tipo, itens e total de qualquer comprovante.
- **OCR simples:** extrai só o texto da imagem (Tesseract), sem usar IA.
- **Exportar CSV** com a lista filtrada.
- **Backup e restauração** em arquivo `.json`, para levar os dados entre aparelhos e navegadores.
- **Instalável e offline:** abre sem internet. A leitura por IA e o OCR precisam de conexão.

## Como usar

1. Gere uma chave de API do Google Gemini em [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Abra o app, cole a chave no campo indicado e envie uma ou mais fotos (escolher, arrastar ou colar com Ctrl + V).
3. Clique em **Ler comprovante**.
4. Para receber avisos de vencimento, clique em **Ativar avisos de vencimento** e permita as notificações.

A chave fica salva só no navegador e **não entra no backup**. Em cada aparelho ou endereço novo é preciso colar a chave de novo.

## Privacidade

- Comprovantes, lembretes e a chave ficam no `localStorage` do navegador.
- As imagens são enviadas diretamente do navegador para a API do Google Gemini para serem lidas. Consulte os termos de uso do Google para saber como os dados enviados são tratados.
- Quem tiver acesso ao seu navegador consegue ver a chave salva. Use uma chave só para este app e revogue-a se desconfiar de vazamento.
- Os dados ficam presos ao endereço onde o app está hospedado. O mesmo app em outro endereço começa vazio.

## Avisos de vencimento: como funcionam e limites

A IA só lê e organiza as datas. Quem mostra o aviso é o navegador, quando o app é aberto e a cada 30 minutos com a aba ou o app aberto, no máximo uma vez por dia para cada conta (vencida ou vencendo em até 3 dias). Sem servidor, o app **não** avisa com o navegador totalmente fechado. Para isso seria preciso um servidor de notificações ou um app nativo.

No iPhone, as notificações só funcionam com o app instalado na Tela de Início (iOS 16.4 ou mais novo).

Contas vencidas há mais de 60 dias não geram lembrete.

## Estrutura do projeto

```
index.html      App completo (HTML, CSS e JavaScript em um único arquivo)
manifest.json   Nome, cores e ícones do app instalado
sw.js           Service worker: cache para uso offline e notificações
_headers        Cabeçalhos para o Netlify (evita cache antigo do sw.js)
.nojekyll       Evita o processamento Jekyll no GitHub Pages
icons/          Ícones do app (192, 512, maskable e Apple touch)
README.md       Este arquivo
```

Não há dependências nem etapa de build. As únicas bibliotecas externas são a fonte Schibsted Grotesk (Google Fonts) e o Tesseract.js, carregado da CDN apenas quando o OCR é usado.

## Rodar no computador

Abra a pasta no Visual Studio Code, instale a extensão **Live Server**, clique com o botão direito em `index.html` e escolha **Open with Live Server**. O app abre em `http://127.0.0.1:5500` com dados próprios, separados dos do app publicado.

Para trocar o modelo de IA, altere a constante `MODELO` no início do script (hoje `gemini-2.5-flash`).

## Publicar

**GitHub Pages**
1. Envie os arquivos para a raiz de um repositório público.
2. Em *Settings*, depois *Pages*, escolha *Deploy from a branch*, a branch `main` e a pasta `/ (root)`.
3. O app fica em `https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/`.

**Netlify**
1. Envie o `.zip` do projeto em *Add new site*, ou conecte o repositório do GitHub.
2. Sem comando de build e sem pasta de publicação especial: o site é estático e fica na raiz.

Todos os caminhos são relativos, então funciona tanto na raiz de um domínio quanto em um subcaminho.

## Atualizar uma versão publicada

1. Edite o código e teste localmente.
2. Aumente o número da versão em `sw.js`, na linha `const CACHE = 'comprovantes-v2'` (para `v3`, `v4`…). Sem isso, aparelhos que já abriram o app podem continuar com a versão antiga guardada.
3. Publique: *commit* e *push* no GitHub, ou envie o zip novo no Netlify.
4. Abra o app e atualize a página uma ou duas vezes para carregar a versão nova.

## Instalar como app

- **Chrome e Edge (computador e Android):** botão **Instalar app** no topo do app ou ícone de instalar na barra de endereço.
- **iPhone:** abra no Safari, toque em Compartilhar e em *Adicionar à Tela de Início*.
- **Firefox (computador):** não instala, mas o app funciona normalmente na aba.

## Problemas comuns

| Problema | O que fazer |
|---|---|
| "Chave sem permissão" ou "Requisição recusada" | Confira a chave e se a API Gemini está ativa para ela. |
| "Limite de uso atingido" | Aguarde alguns minutos. A fila para e mantém as fotos pendentes. |
| "Modelo não encontrado" | Atualize o nome do modelo em `MODELO`. |
| Os dados sumiram | O app foi aberto em outro endereço ou o navegador foi limpo. Use **Restaurar backup**. |
| Mudei o código e nada mudou | Aumente a versão em `sw.js` e recarregue a página. |
| Ícone ou botão de instalar não aparece | Confira se a pasta `icons/` está na raiz e se o endereço usa HTTPS. |

## Limitações conhecidas

- A IA pode errar valores, datas e categorias. Confira os comprovantes importantes e corrija pelo botão **Editar**.
- Fotos tortas, escuras ou com texto cortado reduzem a qualidade da leitura.
- Os dados não sincronizam sozinhos entre aparelhos. Use o backup e a restauração.
- A chave da API fica visível para quem usa o navegador onde ela foi salva.
g README.md…]()
