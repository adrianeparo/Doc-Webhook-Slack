# Documentação Técnica: Criar Webhook do Slack

**Nome:** Adriane Paro  
**Data:** 30/10/2025  
**Objetivo:** Explicar o processo de criação e configuração de um Webhook do Slack para enviar notificações automáticas sobre pull requests no projeto.

---

## 1. Introdução

Um **Webhook** é uma URL que permite que aplicativos enviem notificações automáticas para outros serviços.  
No **Slack**, os Webhooks possibilitam o envio de mensagens diretas para canais específicos, automatizando a comunicação entre ferramentas e a equipe.

Esta documentação aborda:

- Como criar um Webhook no Slack.  
- Estrutura do payload em JSON para envio de mensagens.  
- Integração do Webhook com GitHub Actions.  
- Boas práticas de segurança.  
- Sugestão de prova de conceito.

---

## 2. Criando um Webhook do Slack

### 2.1 Acesse a página de criação de aplicativos do Slack

Acesse:  
👉 [https://api.slack.com/apps](https://api.slack.com/apps)

Clique em **“Create New App”** (Criar novo aplicativo).  

![Create new app](../Webhook-Guide/img/Create_new_app.png)

### 2.2 Escolha o tipo de app

Na janela que aparecer:

1. Selecione **“From scratch”** (Do zero).  
2. Dê um nome para o app (por exemplo, `Webhook PR Notifier`).  
3. Escolha o **workspace** onde o app será criado.  
4. Clique em **Create App**.  
5. Após a criação, você verá a página principal do app, onde estarão disponíveis as **credenciais e opções de configuração**.

### 2.3 Ative os Incoming Webhooks

1. No menu lateral esquerdo, clique em **Incoming Webhooks**.  
2. Ative a opção **“Activate Incoming Webhooks”**.  
3. Em seguida, clique em **“Add New Webhook to Workspace”**.

### 2.4 Escolha o canal de destino

1. O Slack abrirá uma janela pedindo para você escolher o **canal** que receberá as mensagens.  
2. Escolha o canal desejado (ex: `#notificacoes`, `#geral`, `#devs`).  
3. Clique em **Install App**.  

![Install app in Slack](../Webhook-Guide/img/Install-app-in-slack.png)

### 2.5 Copie a URL do Webhook

Após a instalação, será exibida uma **URL de Webhook** semelhante a esta:

![Webhook URL](../Webhook-Guide/img/webhook-url.png)

Essa é a URL que você usará para enviar mensagens ao Slack — manualmente, via script, ou integrando com **GitHub Actions**.

⚠️ **Importante:**  
Essa URL é sensível — qualquer pessoa com acesso a ela pode enviar mensagens para o canal.  

💡 **Dica:** Armazene-a em uma variável de ambiente, por exemplo:

```bash
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/XXX/YYY/ZZZ"
```

---

## 3. Estrutura do Payload

O **payload** é o corpo da mensagem enviada pelo Webhook do Slack.  
Ele deve ser formatado em **JSON**, seguindo a estrutura que o Slack reconhece para exibir mensagens no canal.

### 3.1 Estrutura Básica

O formato mais simples contém apenas o campo `text`, que representa a mensagem principal:

```json
{
  "text": "Olá, este é um teste de mensagem enviada pelo Webhook do Slack! 🚀"
}
```

Essa estrutura envia uma mensagem de texto simples para o canal configurado.

**Resultado no Slack:**
![Payload básico](../Webhook-Guide/img/Payload%20básico.png)

### 3.2 Estrutura com Attachments

Para mensagens mais ricas e personalizadas, é possível usar attachments — seções adicionais que permitem incluir títulos, links, cores e descrições.

**Exemplo:**

```json
{
  "text": "Novo Pull Request aberto!",
  "attachments": [
    {
      "title": "Pull Request #123",
      "title_link": "https://github.com/seu-repo/pull/123",
      "text": "O PR #123 foi aberto por @usuario.",
      "color": "#36a64f"
    }
  ]
}
```

**Resultado no Slack:**
![Estrutura com Attachments](../Webhook-Guide/img/Attachments.png)

#### Explicação dos Campos

| **Campo**         | **Descrição**                                                        |
|-------------------|----------------------------------------------------------------------|
| `text`            | Mensagem principal exibida no canal.                                 |
| `attachments`     | Bloco adicional para informações complementares.                     |
| `title`           | Título exibido em negrito dentro do *attachment*.                    |
| `title_link`      | URL clicável vinculada ao título.                                    |
| `color`           | Cor da barra lateral da mensagem (em formato hexadecimal).           |
| `text` *(interno)*| Texto descritivo adicional, exibido abaixo do título.                |

### 3.3 Payload com Blocos (Blocks)

O Slack também permite mensagens no formato Block Kit, que oferece layouts mais flexíveis e interativos.

**Exemplo de payload com blocks:**

```json
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Novo Pull Request aberto!*"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Autor:*\n@usuario"
        },
        {
          "type": "mrkdwn",
          "text": "*Link:*\n<https://github.com/seu-repo/pull/123|Ver PR>"
        }
      ]
    }
  ]
}

```

**Resultado no Slack:**
![Block Kit](../Webhook-Guide/img/Blocks.png)

#### Vantagens dos Blocks

- Permitem formatação Markdown (`mrkdwn`).
- Suportam listas, negrito, links e até botões.
- interativos.
- Proporcionam mensagens mais legíveis e visuais.

### 3.4 Testando seu Payload

Você pode testar seu payload usando o comando `cURL` no seu terminal:

```bash
curl -X POST -H 'Content-type: application/json' \
--data '{"text":"Mensagem de teste via payload JSON"}' \
https://hooks.slack.com/services/sua_URL_do_Webhook
```

Se o formato estiver correto, a mensagem aparecerá instantaneamente no canal do Slack.

### 3.5 Boas Práticas

- Valide seu JSON antes de enviar (use sites como <https://jsonlint.com>).
- Mantenha o payload conciso e direto.
- Use cores e emojis com moderação para não gerar ruído visual.
- Utilize blocks para mensagens informativas e attachments para alertas simples.

Você pode criar e testar Block Kits com visualização imediata usando o [Block Kit Builder](https://app.slack.com/block-kit-builder/T9W1EEPK5#%7B%22blocks%22:%5B%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22Hello,%20Assistant%20to%20the%20Regional%20Manager%20Dwight!%20*Michael%20Scott*%20wants%20to%20know%20where%20you'd%20like%20to%20take%20the%20Paper%20Company%20investors%20to%20dinner%20tonight.%5Cn%5Cn%20*Please%20select%20a%20restaurant:*%22%7D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Farmhouse%20Thai%20Cuisine*%5Cn:star::star::star::star:%201528%20reviews%5Cn%20They%20do%20have%20some%20vegan%20options,%20like%20the%20roti%20and%20curry,%20plus%20they%20have%20a%20ton%20of%20salad%20stuff%20and%20noodles%20can%20be%20ordered%20without%20meat!!%20They%20have%20something%20for%20everyone%20here%22%7D,%22accessory%22:%7B%22type%22:%22image%22,%22image_url%22:%22https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg%22,%22alt_text%22:%22alt%20text%20for%20image%22%7D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Kin%20Khao*%5Cn:star::star::star::star:%201638%20reviews%5Cn%20The%20sticky%20rice%20also%20goes%20wonderfully%20with%20the%20caramelized%20pork%20belly,%20which%20is%20absolutely%20melt-in-your-mouth%20and%20so%20soft.%22%7D,%22accessory%22:%7B%22type%22:%22image%22,%22image_url%22:%22https://s3-media2.fl.yelpcdn.com/bphoto/korel-1YjNtFtJlMTaC26A/o.jpg%22,%22alt_text%22:%22alt%20text%20for%20image%22%7D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Ler%20Ros*%5Cn:star::star::star::star:%202082%20reviews%5Cn%20I%20would%20really%20recommend%20the%20%20Yum%20Koh%20Moo%20Yang%20-%20Spicy%20lime%20dressing%20and%20roasted%20quick%20marinated%20pork%20shoulder,%20basil%20leaves,%20chili%20&%20rice%20powder.%22%7D,%22accessory%22:%7B%22type%22:%22image%22,%22image_url%22:%22https://s3-media2.fl.yelpcdn.com/bphoto/DawwNigKJ2ckPeDeDM7jAg/o.jpg%22,%22alt_text%22:%22alt%20text%20for%20image%22%7D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22actions%22,%22elements%22:%5B%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Farmhouse%22,%22emoji%22:true%7D,%22value%22:%22click_me_123%22%7D,%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Kin%20Khao%22,%22emoji%22:true%7D,%22value%22:%22click_me_123%22,%22url%22:%22https://google.com%22%7D,%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Ler%20Ros%22,%22emoji%22:true%7D,%22value%22:%22click_me_123%22,%22url%22:%22https://google.com%22%7D%5D%7D%5D%7D)

O **Block Kit Builder** é a ferramenta oficial do Slack para **construir, testar e visualizar layouts de mensagens interativas** em tempo real.

Com ele, você pode montar blocos como seções, botões, imagens e divisores, além de visualizar imediatamente como a mensagem será exibida no canal antes de enviar o payload final.

---

## 4. Integração com GitHub

A integração entre **GitHub** e **Slack** permite automatizar notificações sobre eventos importantes, como abertura, fechamento ou atualização de *pull requests*.  

Com essa integração, sempre que uma ação ocorrer no repositório, o Slack receberá automaticamente uma mensagem informando o status da atividade — melhorando a comunicação entre os desenvolvedores.

### 4.1 Configuração no GitHub

Para enviar mensagens ao Slack sempre que um Pull Request for criado, fechado ou reaberto, podemos usar o **GitHub Actions**.

#### Passo a passo:

1. No repositório do GitHub, crie (ou edite) a pasta `.github/workflows/`.  
2. Dentro dela, crie o arquivo **`pr-notifier.yml`**.  
3. Adicione o conteúdo abaixo:

```yaml
name: PR Notifier

on:
  pull_request:
    types: [opened, closed, reopened]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Enviar notificação para o Slack
        uses: slackapi/slack-github-action@v1.23.0
        with:
          payload: |
            {
              "text": "Novo Pull Request aberto!",
              "attachments": [
                {
                  "title": "PR #${{ github.event.pull_request.number }} - ${{ github.event.pull_request.title }}",
                  "title_link": "${{ github.event.pull_request.html_url }}",
                  "text": "Aberto por *${{ github.actor }}*",
                  "color": "#36a64f"
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### Explicação do Workflow

| Seção | Função |
|:------|:--------|
| `on.pull_request` | Define quais eventos vão acionar o fluxo (nesse caso: abertura, fechamento e reabertura de PRs). |
| `jobs.notify` | Cria o job que enviará a notificação. |
| `uses: slackapi/slack-github-action` | Usa a Action oficial do Slack para enviar mensagens. |
| `payload` | Contém o corpo da mensagem em formato JSON (como no tópico anterior). |
| `SLACK_WEBHOOK_URL` | É a variável de ambiente que armazena o Webhook. Deve ser configurada como *secret* no repositório. |

#### Adicionando o Webhook como Secret no GitHub

1. No repositório, vá em **Settings → Secrets → Actions**.
2. Clique em **New repository secret**.
3. Nomeie o secret como `SLACK_WEBHOOK_URL`.
4. Cole a URL do seu Webhook do Slack.
5. Clique em **Add secret**.

Agora o seu Webhook está protegido e pode ser usado com segurança dentro do workflow.

#### Teste da Integração

1. Crie um novo Pull Request no repositório.
2. Aguarde a execução do workflow.
3. Verifique o canal do Slack configurado — a mensagem deve aparecer automaticamente.

Se tudo estiver correto, você verá algo como:
![Novo Pull request](../Webhook-Guide/img/novo-pull-request.png)

Você pode adaptar esse mesmo fluxo para outros eventos do GitHub, como:

- `push` **→ para notificar sobre commits.**
- `issues` **→ para alertar sobre novas issues abertas.**
- `deployment` **→ para informar sobre implantações em produção.**

---

## 5. Considerações de Segurança

Proteger a URL do Webhook é **essencial** para evitar que terceiros enviem mensagens não autorizadas para o seu canal do Slack. A URL do Webhook funciona como uma **chave de acesso** — qualquer pessoa que a possua poderá publicar mensagens no canal vinculado.

### 5.1 Boas práticas de segurança

#### 1. Nunca exponha a URL do Webhook no código-fonte

- Evite colocar a URL diretamente em arquivos `.js`, `.py`, `.yml` ou qualquer outro arquivo do repositório.
- Mesmo em repositórios privados, é recomendável ocultar essas informações sensíveis.

#### 2. Use variáveis de ambiente

- Armazene a URL em uma variável, por exemplo:

```bash
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
```

- No código, acesse a variável assim (exemplo em JavaScript):

```javascript
const webhookUrl = process.env.SLACK_WEBHOOK_URL;
```

#### 3. Configure secrets no GitHub Actions

- Vá até **Settings > Secrets and variables > Actions.**
- Clique em **New repository secret.**
- Dê um nome (por exemplo, `SLACK_WEBHOOK_URL`) e cole a URL.
- No workflow do GitHub Actions, utilize:

```yaml
- name: Enviar notificação ao Slack
  run: curl -X POST -H 'Content-type: application/json' \
    --data '{"text":"Deploy realizado com sucesso!"}' \
    ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### 4. Revogue Webhooks não utilizados

- Acesse o **Slack API Dashboard → selecione seu app → Incoming Webhooks.**
- Desative ou exclua os Webhooks que não são mais necessários.

Essas práticas ajudam a manter a integração segura e confiável, evitando o uso indevido do seu Webhook e protegendo as informações do seu workspace.

---

## 6. Prova de Conceito (PoC)

A melhor forma de entender como os **Webhooks do Slack** funcionam é realizando uma **Prova de Conceito (PoC)**.  

Essa etapa serve para testar se o envio de mensagens automatizadas para um canal do Slack está configurado corretamente antes da integração com o GitHub.

### Objetivo da PoC

Verificar se o Webhook é capaz de **receber mensagens** enviadas por uma aplicação simples ou comando manual.  
Isso garante que:

- A URL do Webhook está ativa e válida;  
- O canal escolhido no Slack está recebendo as notificações corretamente;  
- A estrutura do payload JSON está bem formatada.

#### Passo a passo

1. **Crie um canal de teste no Slack**  
   - Vá até o Slack e clique em **“Add channel” → “Create a new channel”**.  
   - Dê um nome, como `#teste-webhook`.  
   - Defina se será público ou privado e clique em **Create**.

2. **Crie um Webhook para esse canal**  
   - Acesse o [Slack API Dashboard](https://api.slack.com/apps).  
   - Escolha seu app e vá até **Incoming Webhooks**.  
   - Clique em **“Add New Webhook to Workspace”** e selecione o canal de teste criado.  
   - Copie a **URL do Webhook** gerada.

3. **Envie uma mensagem manualmente (via terminal)**  
   No seu terminal, use o comando abaixo substituindo a URL pela do seu Webhook:

   ```bash
   curl -X POST -H 'Content-type: application/json' \
   --data '{"text":"🚀 Teste de Webhook funcionando com sucesso!"}' \
   https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
   ```

---

## 7. Conclusão

O **Webhook do Slack** automatiza o envio de notificações sobre pull requests e outras atividades do GitHub diretamente para o canal da equipe.  

Isso garante que todos fiquem atualizados em tempo real, melhora a comunicação e agiliza a colaboração entre desenvolvedores.  

Seguindo as boas práticas de segurança, como usar **variáveis de ambiente**, o Webhook permanece seguro e confiável, tornando o fluxo de trabalho mais eficiente e transparente.

---

## 8. Referências

- [Slack API – Incoming Webhooks](https://api.slack.com/messaging/webhooks)  
- [Slack Message Formatting](https://api.slack.com/messaging/composing/layouts)  
- [Slack GitHub Action](https://github.com/slackapi/slack-github-action)  
- [GitHub Actions – Workflow Syntax](https://docs.github.com/pt/actions/using-workflows/workflow-syntax-for-github-actions)  
- [JSONLint – Validador de JSON](https://jsonlint.com/)
