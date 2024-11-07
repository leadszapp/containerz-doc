# Guia de Implementação da API Containerz

## Índice
1. [Configuração Inicial](#configuração-inicial)
2. [Autenticação](#autenticação)
3. [Fluxo de Implementação](#fluxo-de-implementação)
4. [Endpoints Principais](#endpoints-principais)
5. [Webhooks](#webhooks)
6. [Envio de Mensagens](#envio-de-mensagens)
7. [Configurações Avançadas](#configurações-avançadas)
8. [Gerenciamento de Fila](#gerenciamento-de-fila)

## Configuração Inicial

### Base URL
```
https://api.containerz.app
```

### Variáveis Necessárias
- `api_key`: Chave de API para autenticação
- `application_uuid`: UUID da aplicação
- `instance_uuid`: UUID da instância
- `image_uuid`: UUID da imagem (padrão: "51c577f9-19c3-4682-8394-0d55632bec10")
- `token`: Token de autenticação para widgets

## Autenticação

Todas as requisições devem incluir o header:
```
X-API-KEY: {sua_api_key}
```

## Fluxo de Implementação

### 1. Criação da Instância
1. Faça uma requisição POST para `/applications/instances/create`
2. Forneça:
   - Nome da instância
   - UUID da aplicação
   - UUID da imagem

### 2. Gerenciamento da Instância
Após criar a instância, você pode:
- Iniciar: POST `/applications/instances/start`
- Parar: POST `/applications/instances/stop`
- Remover: POST `/applications/instances/remove`

### 3. Configuração de Webhook
Configure webhooks para receber eventos da instância:
1. Adicione webhook: POST `/api/instance` (model: webhook, method: add)
2. Configure eventos desejados
3. Gerencie webhooks existentes com os métodos get e delete

## Endpoints Principais

### Gerenciamento de Instância
1. **Criar Instância**
   ```http
   POST /applications/instances/create
   ```

2. **Iniciar Instância**
   ```http
   POST /applications/instances/start
   ```

3. **Parar Instância**
   ```http
   POST /applications/instances/stop
   ```

4. **Remover Instância**
   ```http
   POST /applications/instances/remove
   ```

### Informações da Instância
```http
POST /applications/instances/list
```

## Envio de Mensagens

### Tipos de Mensagens Suportados

1. **Texto**
```json
{
    "body": "Sua mensagem",
    "body_type": "text"
}
```

2. **Imagem**
```json
{
    "body": "URL_DA_IMAGEM",
    "body_type": "image",
    "config": {
        "caption": "Legenda opcional"
    }
}
```

3. **PDF**
```json
{
    "body": "URL_DO_PDF",
    "body_type": "pdf",
    "config": {
        "filename": "nome_arquivo.pdf"
    }
}
```

4. **Áudio**
```json
{
    "body": "URL_DO_AUDIO",
    "body_type": "audio"
}
```

5. **Vídeo**
```json
{
    "body": "URL_DO_VIDEO",
    "body_type": "video",
    "config": {
        "caption": "Legenda opcional"
    }
}
```

6. **Link Preview**
```json
{
    "body": "URL_COM_PREVIEW",
    "body_type": "linkpreview"
}
```

### Estrutura Básica de Envio
```json
{
    "instance_uuid": "UUID_DA_INSTANCIA",
    "model": "message",
    "method": "send",
    "payload": {
        "action": "send_message",
        "recipient": "NUMERO_TELEFONE",
        "recipient_type": "user",
        "notify": true,
        "messages": [
            {
                "body": "CONTEUDO",
                "body_type": "TIPO",
                "config": {
                    "caption": "LEGENDA",
                    "callback": {
                        "id_interno": "ID"
                    }
                }
            }
        ]
    }
}
```

## Webhooks

### Eventos Disponíveis
- Status e QR Code
  - `response.status`
  - `response.status.running`
  - `response.qrcode`
  - `response.info_device`
  - `response.screen_capture`

- Mensagens
  - `response.events.chat.message`
  - `response.events.group.message`
  - `response.group_sended_message`
  - `response.chat_sended_message`

- Confirmações (ACK)
  - `response.ack_message_send`
  - `response.ack_message_confirm`
  - `response.ack_message_read`
  - `response.ack_message_played`

- Grupos
  - `response.group_metadata`
  - `response.create_group`
  - `response.invite_group`
  - `response.leave_group`
  - `response.group_icon_updated`
  - `response.group_description_updated`
  - `response.group_subject_updated`
  - `response.group_restrict_updated`
  - `response.group_announcement_updated`

### Configuração de Webhook
```json
{
    "instance_uuid": "UUID_DA_INSTANCIA",
    "model": "webhook",
    "method": "add",
    "payload": {
        "name": "Nome do Webhook",
        "url": "URL_DO_WEBHOOK",
        "events": ["LISTA_DE_EVENTOS"],
        "attachment": true,
        "ack": false
    }
}
```

## Configurações Avançadas

### Configuração da Instância
```json
{
    "instance_uuid": "UUID_DA_INSTANCIA",
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

## Gerenciamento de Fila

### Contagem de Mensagens na Fila
```http
POST /api/instance
{
    "instance_uuid": "UUID_DA_INSTANCIA",
    "model": "queue",
    "method": "count"
}
```

### Limpar Fila
```http
POST /api/instance
{
    "instance_uuid": "UUID_DA_INSTANCIA",
    "model": "queue",
    "method": "delete_all"
}
```

## Notas Importantes

1. Mantenha sua `api_key` segura e não a compartilhe
2. Configure corretamente os webhooks para receber atualizações em tempo real
3. Utilize o endpoint de fila para gerenciar o fluxo de mensagens
4. Monitore os eventos de ACK para confirmar a entrega das mensagens
5. Configure adequadamente as propriedades de grupos ao gerenciá-los
