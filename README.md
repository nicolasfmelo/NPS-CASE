# Instituto Horizonte Digital - Plataforma de Avaliação de Atendimento com IA

Este repositório contém uma solução neutra para avaliar a qualidade de atendimento de uma IA operadora em conversas de captação/orientação de alunos. A entrega combina protótipo funcional, documentação técnica, desenhos de arquitetura, custos estimados e visão de operação.

A aplicação usa o **Instituto Horizonte Digital** como instituição fictícia e a **Clara** como consultora educacional. O objetivo é medir se o cliente avançou com clareza, baixo atrito e aderência ao que buscava.

## Visão geral

O projeto implementa um MVP com:

- chat estilo WhatsApp para texto e áudio;
- agente conversacional para atendimento educacional;
- agente avaliador NPS-like para análise estruturada de conversas;
- exportação de conversas e evidências;
- métricas operacionais, LLMOps, observabilidade e rastreabilidade;
- proposta batch para escala e redução de custo.

## Escopo do problema

A solução foi desenhada para receber conversas de atendimento em canais digitais, analisar a experiência do cliente e retornar uma avaliação estruturada por sessão. A extensão para voz entra pelo fluxo de transcrição e reaproveita a mesma régua de avaliação.

Critérios considerados:

- viabilidade econômica;
- auditabilidade e rastreabilidade;
- evolução versionada de prompts e critérios;
- apoio inicial à avaliação humana;
- cuidado com dados sensíveis;
- potencial de escala em produção.

## Metodologia de avaliação NPS-like

A unidade de análise é a sessão inteira. A avaliação principal segue a perspectiva do cliente, não a do bot.

| Métrica | Escala | Leitura prática |
| --- | --- | --- |
| Satisfação | 1 a 3 | Resultado percebido pelo cliente |
| Esforço do cliente | 1 a 5 | Quanto o cliente precisou insistir, repetir ou corrigir |
| Entendimento do objetivo | 0 a 2 | Se a IA entendeu cedo, tarde ou não entendeu |
| Resolução / avanço útil | 0 a 2 | Se houve resposta útil ou próximo passo coerente |
| Mudança comportamental | qualitativa | Evolução positiva, neutra ou negativa da conversa |

Indicador consolidado:

```text
INDICE_IA_OPERADORA = % bom - % ruim
```

## Arquitetura MVP Inline

Fluxo implementado para demonstrar o produto funcionando de ponta a ponta: front-end, API, orquestração, agentes, banco, object store e observabilidade local.

![MVP Inline Process](docs/arquitetura/mvp-inline-process.png)

Principais responsabilidades:

- **Front-end React**: chat, métricas, LLMOps e documentação interna.
- **FastAPI**: contratos HTTP, validação, orquestração e exposição de métricas.
- **Agente de atendimento**: conduz a conversa com base no perfil institucional e catálogo de cursos.
- **Agente avaliador**: gera score, justificativa, evidências e sinais operacionais.
- **Postgres/MinIO**: persistência relacional e exportação de artefatos.
- **Prometheus/Loki/Grafana**: telemetria, logs e painéis.

## Arquitetura MVP Batch

Proposta recomendada para volume, custo e governança. A análise ocorre por lotes, com armazenamento de entrada/saída, eventos, processamento assíncrono e execução batch no provider de LLM.

![MVP Batch Process](docs/arquitetura/mvp-batch-process.png)

Por que batch:

- reduz infraestrutura quente dedicada à inferência;
- melhora custo por análise em alto volume;
- facilita reprocessamento auditável;
- separa ingestão, análise e publicação de resultados;
- permite controle por versão de prompt/modelo/lote.

## Sistema de créditos / controle operacional

O desenho abaixo representa uma camada complementar para controle de consumo, rate limit, orçamento e rastreabilidade por cliente, workspace ou lote.

![Credit System](docs/arquitetura/credit-system.png)

## Decisões técnicas

- **Python 3.12 + FastAPI** no back-end por simplicidade de API, tipagem e ecossistema de IA.
- **React + TypeScript + Vite** no front-end para prototipação rápida e UX rica.
- **LangGraph** para organizar fluxos agentic quando há estado e decisões intermediárias.
- **Postgres** para dados transacionais e avaliações estruturadas.
- **MinIO/S3-compatible** para exportações e artefatos.
- **Prometheus, Loki e Grafana** para operação local observável.
- **Arquitetura MASA-inspired** no back-end para separar domain, engines, services, integrations e interfaces.

## Estrutura do repositório

```text
case/                         # materiais brutos de referência do desafio
custos.md                     # estimativa de tokens e custos de LLM
docs/arquitetura/             # diagramas de arquitetura
estimated-infra-costs.md      # estimativa de infraestrutura AWS
fluxo-docs.md                 # documentação consolidada anterior
services/back-end/            # API FastAPI e agentes
services/front-end/           # aplicação React
services/compose.yaml         # stack local completa
services/observability/       # Prometheus, Loki, Promtail e Grafana
```

## Como rodar

### Docker Compose (recomendado)

Pré-requisitos:

- Docker;
- Docker Compose Plugin.

```bash
cd services
docker compose up -d --build
```

Acessos:

| Serviço | URL |
| --- | --- |
| Front-end | http://localhost:8080 |
| Back-end / Swagger | http://localhost:8000/docs |
| Grafana | http://localhost:3000 |
| Prometheus | http://localhost:9090 |
| Loki | http://localhost:3100 |

Parar:

```bash
cd services
docker compose down
```

### Desenvolvimento local

Subir infraestrutura base:

```bash
cd services
docker compose up -d postgres minio prometheus loki promtail grafana
```

Back-end:

```bash
cd services/back-end
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/instituto_horizonte"
python run_local.py
```

Front-end:

```bash
cd services/front-end
bun install
bun run dev
```

Front-end em modo dev: http://localhost:5173

## Contratos/API principais

A documentação detalhada fica em `services/back-end/api.md` e no Swagger local. Principais grupos:

- chat e envio de mensagens;
- análise de conversa;
- exportação de sessões;
- métricas agregadas;
- registro e versionamento de prompts;
- endpoints de saúde/observabilidade.

## Observabilidade e LLMOps

O MVP inclui visão operacional para acompanhar:

- latência e status da API;
- volume de sessões e avaliações;
- distribuição de satisfação;
- esforço médio do cliente;
- tokens de prompt/completion/total;
- detecção de prompt injection;
- evolução e comparação de prompts.

## Custos

Os custos estão consolidados em dois documentos:

- [Custos de modelos/tokens](custos.md)
- [Custos estimados de infraestrutura AWS](estimated-infra-costs.md)

Resumo para **30.000 análises/mês** com infraestrutura AWS estimada em `us-east-1`:

| Modelo | LLM 30k | Total infra + LLM com margem de 30% |
| --- | ---: | ---: |
| `anthropic.claude-sonnet-4-6` | US$ 139,32 | US$ 293,81 |
| `anthropic.claude-haiku-4-5-20251001-v1:0` | US$ 46,44 | US$ 173,07 |
| `minimax.minimax-m2.5` | US$ 10,20 | US$ 125,96 |
| `amazon.nova-2-lite-v1:0` | US$ 22,92 | US$ 142,49 |

A estimativa usa batch split, 30% de margem, Fargate, ALB, S3, EventBridge, Lambda, DynamoDB, CloudFront, WAF e CloudWatch. Valores reais podem variar por região, contrato, tráfego, retenção de logs e modelo escolhido.

## Limitações

- O dataset de exemplo é pequeno e deve ser ampliado antes de calibrar pesos finais.
- A régua de avaliação precisa de validação humana inicial para reduzir viés.
- Custos são estimativas e devem ser recalculados antes de produção.
- A transcrição de áudio foi prevista no fluxo, mas a qualidade depende do provider usado.
- Regras de privacidade, retenção e anonimização devem ser endurecidas para ambiente real.

## Próximos passos

- Criar golden dataset anotado por avaliadores humanos.
- Versionar rubricas e prompts com aprovação operacional.
- Implementar pipeline batch gerenciado em cloud.
- Adicionar anonimização/redação automática de PII.
- Expandir dashboards de LLMOps com drift, custo por lote e comparação de modelos.
- Automatizar testes de regressão de qualidade do agente avaliador.

## Documentação complementar

- `services/front-end/src/components/docs/docs-page.tsx` - documentação renderizada dentro do front-end.
- `fluxo-docs.md` - fluxo de solução consolidado em formato textual.
- `services/back-end/README.md` - detalhes do back-end.
- `services/back-end/llm-contract.md` - contrato do gateway/proxy LLM.
- `services/back-end/tables.md` - modelagem de tabelas.
