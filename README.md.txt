# Agente de WhatsApp para orçamentos de gravação a laser

Automação em n8n que atende clientes da **LaserBA** (gravação a laser) pelo WhatsApp,
coleta os dados do pedido com um agente de IA, calcula o valor e gera o orçamento
em PDF, com **aprovação humana antes do envio ao cliente**.

> **Status:** em desenvolvimento (fase de testes). Hospedado no Render, com banco no NeonDB.

## Problema que resolve

Responder cada cliente e montar orçamentos manualmente consome tempo. Este projeto
automatiza o atendimento inicial e a geração do orçamento, mantendo o controle do
valor final nas mãos do responsável.

## Como funciona

![Fluxo no n8n](docs/fluxo.png)

```mermaid
flowchart LR
  A[Cliente no WhatsApp] --> B[Evolution API]
  B --> C[Webhook n8n]
  C --> D{Remetente é sócio?}
  D -- não --> E[Agente IA Gemini + memória]
  E --> F[Cálculo de preço - Google Sheets]
  F --> G[Cópia da planilha modelo - Drive]
  G --> H[Exporta PDF]
  H --> I[Envia ao sócio para aprovação]
  D -- sim --> J[Comando: aprovar / cancelar OS]
  J --> K[Envia PDF ao cliente]
  K --> L[Atualiza status na aba Serviços]
```

## Funcionalidades

- Atendimento por WhatsApp com agente de IA (Gemini) e **memória persistente** por telefone
- Coleta estruturada dos dados do pedido (saída em JSON validada)
- Envio de catálogo de fontes quando o cliente precisa escolher
- Dois tipos de orçamento:
  - **Gravação:** faixas "a partir de" por quantidade, com valor unitário por tipo de conteúdo (texto, arte normal, arte complexa, texto + arte)
  - **Locação de máquina:** `dias × horas × valor_hora + (km × 2 × dias × valor_km)`
- Geração do orçamento a partir de uma planilha modelo copiada no Google Drive
- Exportação do PDF somente da aba do orçamento
- **Aprovação por comando no WhatsApp** (`aprovar 123` / `cancelar 123`)
- Registro de cada serviço com status (Aguardando aprovação, Enviado, Cancelado)
- Proteção do webhook por header, deduplicação de mensagens e persistência em PostgreSQL

## Stack

| Camada | Ferramenta |
|---|---|
| Orquestração | n8n |
| Integração com WhatsApp | Evolution API v2 |
| IA | Google Gemini (Flash) |
| Dados e documentos | Google Sheets e Google Drive |
| Banco de dados | PostgreSQL (NeonDB) |
| Hospedagem (testes) | Render |

## Estrutura do repositório

```
├── workflows/    workflow exportado do n8n (sem credenciais)
├── docs/         estrutura da planilha e imagem do fluxo
├── .env.example  variáveis de ambiente necessárias
└── README.md
```

A estrutura das abas e colunas da planilha está em [`docs/planilha.md`](docs/planilha.md).

## Como rodar

1. Suba o n8n e a Evolution API (cada uma com seu banco PostgreSQL) usando as variáveis de `.env.example`
2. Importe `workflows/workflow-orcamentos.json` no n8n
3. Recrie as credenciais: Google Drive/Sheets (OAuth2), Gemini, Postgres e Header Auth do webhook
4. Substitua os placeholders do workflow (`SUA-EVOLUTION`, `SEU-N8N`, `NOME_DA_INSTANCIA`, `NUMERO_DO_SOCIO`, `ID_DA_PLANILHA`, `ID_DA_PASTA`, `SUA_APIKEY`)
5. Monte a planilha conforme `docs/planilha.md`
6. Configure o webhook da instância na Evolution apontando para o n8n, com o mesmo header de autenticação

## Aprendizados

- Hospedagem, limites de planos gratuitos e variáveis de ambiente
- Criação e conexão de banco PostgreSQL
- Consumo de APIs, webhooks e autenticação (OAuth, API keys, header de proteção)
- Agentes de IA com memória persistente e saída estruturada
- Deduplicação de mensagens e fluxo com aprovação humana
- JavaScript em nós de código e depuração de erros de requisição

## Próximos passos

- [ ] Identificar a complexidade da arte enviada pelo cliente e ajustar o valor
- [ ] Calcular a distância automaticamente na locação
- [ ] Migrar para um provedor de hospedagem mais estável
- [ ] Ampliar o banco de dados
- [ ] Tratamento de erros do agente com alerta ao responsável

## Sobre o desenvolvimento

Projeto paralelo de estudo, feito para colocar em prática o que aprendi no Santander 2026 - Automação com N8N
em um problema real da minha empresa. Foi construído com o apoio do
Claude (Anthropic), que atuou como instrutor: explicou as etapas e ajudou a depurar
erros, enquanto eu montei, testei e ajustei o fluxo.

## Autor

[Yuri Moreira Rangel](https://www.linkedin.com/in/yuri-rangel-18808541b/?isSelfProfile=true) · [GitHub](https://github.com/Yuri-m-r)