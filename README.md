# API Containerz - Guia de Implementação

## Sumário
- [Introdução](#introdução)
- [Pré-requisitos](#pré-requisitos)
- [Fluxo de Implementação](#fluxo-de-implementação)
- [Endpoints e Exemplos](#endpoints-e-exemplos)
- [Configuração de Webhooks](#configuração-de-webhooks)
- [Tipos de Mensagens](#tipos-de-mensagens)
- [Gestão de Instâncias](#gestão-de-instâncias)
- [FAQ e Troubleshooting](#faq-e-troubleshooting)

## Introdução

A API Containerz permite a integração com o WhatsApp para envio e recebimento de mensagens, gestão de grupos e automação de comunicações. Este guia fornecerá todas as informações necessárias para implementar a API em seu sistema.

## Pré-requisitos

Para utilizar a API, você precisará dos seguintes dados:
```
URL Base: https://api.containerz.app
API Key: Sua chave de autenticação
Application UUID: Identificador da sua aplicação
Image UUID: 51c577f9-19c3-4682-8394-0d55632bec10 (padrão)
```

Todas as requisições devem incluir o header:
```
X-API-KEY: {sua_api_key}
Content-Type: application/json
```

## Fluxo de Implementação

A implementação segue um fluxo específico que deve ser seguido na ordem correta:

### 1. Criar uma Nova Instância

Primeiro, crie uma instância do WhatsApp:

```http
POST /applications/instances/create
{
    "name": "Nome da Instância",
    "application_uuid": "seu_application_uuid",
    "image_uuid": "51c577f9-19c3-4682-8394-0d55632bec10"
}
```

Guarde o `instance_uuid` retornado - ele será necessário para todas as operações subsequentes.

### 2. Configurar a Instância

Configure os parâmetros básicos de funcionamento:

```http
POST /api/instance
{
    "instance_uuid": "seu_instance_uuid",
    "model": "config",
    "method": "add",
    "payload": {
        "config": {
            "sendseen": true,
            "attachment": true,
            "listener": {
                "from_me": true
            }
        }
    }
}
```

### 3. Configurar Webhook

Configure um webhook para receber eventos:

```http
POST /api/instance
{
    "instance_uuid": "seu_instance_uuid",
    "model": "webhook",
    "method": "add",
    "payload": {
        "name": "Meu Webhook",
        "url": "https://sua-url.com/webhook",
        "events": [
            "response.status",
            "response.qrcode",
            "response.events.chat.message",
            "response.ack_message_send"
        ],
        "attachment": true,
        "ack": false
    }
}
```

## Eventos do Webhook

Os eventos disponíveis para monitoramento via webhook estão organizados nas seguintes categorias:

### 1. Eventos de Status e Dispositivo
```javascript
[
    // Status da instância
    "response.status",              // Status geral da instância
    "response.status.running",      // Instância em execução
    "response.qrcode",             // Novo QR code gerado
    "response.info_device",        // Informações do dispositivo
    "response.screen_capture",     // Captura de tela do dispositivo
]
```

### 2. Eventos de Grupos
```javascript
[
    // Listagem e Metadados
    "response.list_all_groups",    // Lista todos os grupos
    "response.list_groups",        // Lista grupos específicos
    "response.group_metadata",     // Metadados do grupo
    "request.group_metadata",      // Requisição de metadados
    "response.create_group",       // Criação de novo grupo

    // Gerenciamento
    "response.invite_group",       // Convite para grupo
    "response.leave_group",        // Saída do grupo

    // Atualizações
    "response.group_icon_updated",          // Atualização do ícone
    "response.group_description_updated",   // Atualização da descrição
    "response.group_subject_updated",       // Atualização do título
    "response.group_restrict_updated",      // Atualização de restrições
    "response.group_announcement_updated"   // Atualização de anúncios
]
```

### 3. Eventos de Mensagens
```javascript
[
    // Mensagens Individuais e Grupos
    "response.events.chat.message",     // Mensagens de chat individual
    "response.events.group.message",    // Mensagens de grupo
    
    // Mensagens Enviadas
    "response.chat_sended_message",     // Mensagem enviada em chat
    "response.group_sended_message",    // Mensagem enviada em grupo
    
    // Mensagens Recebidas
    "response.chat_received_message",   // Mensagem recebida em chat
    "response.group_received_message"   // Mensagem recebida em grupo
]
```

### 4. Eventos de Confirmação (ACK)
```javascript
[
    // ACK Geral
    "response.ack_message_send",        // Mensagem enviada
    "response.ack_message_confirm",     // Mensagem entregue
    "response.ack_message_read",        // Mensagem lida
    "response.ack_message_played",      // Mensagem de áudio reproduzida

    // ACK de Grupo
    "response.group_ack_message_send",      // Mensagem enviada no grupo
    "response.group_ack_message_confirm",   // Mensagem entregue no grupo
    "response.group_ack_message_read",      // Mensagem lida no grupo
    "response.group_ack_message_played",    // Áudio reproduzido no grupo

    // ACK de Chat Individual
    "response.chat_ack_message_send",       // Mensagem enviada no chat
    "response.chat_ack_message_confirm",    // Mensagem entregue no chat
    "response.chat_ack_message_read",       // Mensagem lida no chat
    "response.chat_ack_message_played"      // Áudio reproduzido no chat
]
```

### Exemplo de Configuração Completa de Webhook
```http
POST /api/instance
{
    "instance_uuid": "seu_instance_uuid",
    "model": "webhook",
    "method": "add",
    "payload": {
        "name": "Webhook Completo",
        "url": "https://sua-url.com/webhook",
        "events": [
            // Adicione os eventos desejados da lista acima
            "response.status",
            "response.qrcode",
            "response.events.chat.message",
            "response.group_sended_message",
            // ... outros eventos ...
        ],
        "attachment": true,
        "ack": true
    }
}
```

### Dicas para Gerenciamento de Eventos

1. **Priorização de Eventos**
   - Comece com eventos essenciais (status, mensagens, ACKs básicos)
   - Adicione eventos específicos conforme necessidade
   - Evite monitorar eventos desnecessários para sua aplicação

2. **Processamento de Eventos**
   - Implemente filas para processamento assíncrono
   - Mantenha logs de eventos recebidos
   - Implemente retry para eventos críticos

3. **Monitoramento**
   - Monitore a latência dos eventos
   - Verifique eventos perdidos
   - Mantenha métricas de sucesso/falha

### 4. Iniciar a Instância

Inicie a instância criada:

```http
POST /applications/instances/start
{
    "instance_uuid": "seu_instance_uuid"
}
```

### 5. Autenticação via QR Code

#### 5.1. Gerar Token JWT
```http
POST /api/instance/token
{
    "instance_uuid": "seu_instance_uuid",
    "expire": "48"
}
```

#### 5.2. Exibir QR Code
Utilize o token gerado para exibir o QR Code em seu sistema:

```html
<!-- Via iframe -->
<iframe 
    src="https://containerz.leadszapp.com/api/instance?model=widget&method=qrcode&instance_uuid=SEU_INSTANCE_UUID&token=SEU_TOKEN"
    width="400"
    height="400"
    frameborder="0">
</iframe>

<!-- Ou via URL direta -->
https://containerz.leadszapp.com/api/instance?model=widget&method=qrcode&instance_uuid=SEU_INSTANCE_UUID&token=SEU_TOKEN
```

## Tipos de Mensagens

### Texto Simples
```http
POST /api/instance
{
    "instance_uuid": "seu_instance_uuid",
    "model": "message",
    "method": "send",
    "payload": {
        "action": "send_message",
        "recipient": "5511999999999",
        "recipient_type": "user",
        "notify": true,
        "messages": [
            {
                "body": "Sua mensagem aqui",
                "body_type": "text"
            }
        ]
    }
}
```

### Imagem
```http
{
    // ... dados básicos ...
    "messages": [
        {
            "body": "https://url-da-imagem.com/imagem.jpg",
            "body_type": "image",
            "config": {
                "caption": "Legenda da imagem"
            }
        }
    ]
}
```

### Documento PDF
```http
{
    // ... dados básicos ...
    "messages": [
        {
            "body": "https://url-do-arquivo.com/documento.pdf",
            "body_type": "pdf",
            "config": {
                "filename": "Nome do Arquivo.pdf"
            }
        }
    ]
}
```

## Eventos do Webhook

Os principais eventos que você pode monitorar:

```javascript
[
    "response.status",           // Status da instância
    "response.qrcode",          // Novo QR code gerado
    "response.status.running",   // Instância em execução
    "response.events.chat.message",  // Mensagens recebidas
    "response.group_sended_message", // Mensagens enviadas para grupos
    "response.ack_message_send",     // Confirmação de envio
    "response.ack_message_read"      // Confirmação de leitura
]
```

## Gestão de Instâncias

### Verificar Status
```http
POST /applications/instances/list
{
    "application_uuid": "seu_application_uuid",
    "uuids": ["seu_instance_uuid"],
    "generate_token": false
}
```

### Parar Instância
```http
POST /applications/instances/stop
{
    "instance_uuid": "seu_instance_uuid"
}
```

### Remover Instância
```http
POST /applications/instances/remove
{
    "instance_uuid": "seu_instance_uuid"
}
```

## FAQ e Troubleshooting

### Códigos de Status Comuns
- `200`: Operação realizada com sucesso
- `400`: Erro nos parâmetros da requisição
- `401`: Erro de autenticação
- `404`: Recurso não encontrado
- `500`: Erro interno do servidor

### Problemas Comuns

1. **QR Code não aparece**
   - Verifique se o token JWT está válido
   - Confirme se a instância está iniciada
   - Verifique os eventos no webhook

2. **Mensagens não são enviadas**
   - Confirme o formato do número do destinatário (DDI+DDD+NÚMERO)
   - Verifique se a instância está autenticada
   - Monitore os eventos de ACK no webhook

3. **Webhook não recebe eventos**
   - Confirme se a URL do webhook está acessível
   - Verifique se os eventos desejados foram configurados
   - Teste a URL do webhook manualmente

### Boas Práticas

1. **Segurança**
   - Mantenha sua API key segura
   - Use HTTPS para seu webhook
   - Valide todos os dados recebidos no webhook

2. **Performance**
   - Monitore a fila de mensagens
   - Implemente retry em caso de falhas
   - Processe eventos do webhook de forma assíncrona

3. **Manutenção**
   - Monitore o status da instância regularmente
   - Implemente logs para debug
   - Mantenha um sistema de monitoramento dos eventos

