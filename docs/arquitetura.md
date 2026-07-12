# Arquitetura de Ingestão — Google SecOps

## Visão geral

Este documento detalha a arquitetura proposta para ingestão de logs de uma VPS Linux (Ubuntu 24.04) no Google Security Operations, usando Bindplane como agente de coleta em substituição ao Chronicle Forwarder legado (descontinuado em janeiro/2027).

## Fontes de log

| Fonte | Caminho | Tipo de evento |
|---|---|---|
| SSH Auth | `/var/log/auth.log` | Autenticação (sucesso/falha) |
| Firewall | `/var/log/ufw.log` | Bloqueios de rede |
| n8n / Docker | container logs | Automação/aplicação |

## Pipeline de coleta (Bindplane)

O Bindplane roda como um coletor OpenTelemetry na própria VPS, com receivers dedicados por fonte de log, processadores de batch, e exportador nativo para o Chronicle via credenciais de service account.

Estrutura conceitual do `config.yaml`:

```yaml
receivers:
  filelog/auth:
    include:
      - /var/log/auth.log
    start_at: end

processors:
  batch:

exporters:
  chronicle/chronicle_w_labels:
    compression: gzip
    creds_file_path: "/opt/observiq-otel-collector/creds.json"
    log_type: "LINUX_AUTH"
    customer_id: "<CUSTOMER_ID>"
    region: "us"

service:
  pipelines:
    logs/auth:
      receivers: [filelog/auth]
      processors: [batch]
      exporters: [chronicle/chronicle_w_labels]
```

## Normalização (UDM)

Após ingestão, os eventos passam pelo parser padrão do Google para o tipo `LINUX_AUTH`, mapeando para os campos UDM:

- `principal.user.userid` — usuário envolvido na tentativa
- `principal.ip` — IP de origem
- `security_result.action` — ALLOW / BLOCK
- `metadata.event_type` — USER_LOGIN

Quando o parser padrão não cobre um formato de log customizado (como logs de aplicação do n8n), a estratégia é escrever um parser customizado usando a linguagem de parsing do Google SecOps (baseada em Grok/regex + transformação de campos).

## Camada de resposta (SOAR)

A lógica de resposta segue o mesmo padrão já validado no meu playbook pessoal em n8n:

1. Webhook recebe alerta da regra `ssh_brute_force_detection`
2. Consulta o IP na API da VirusTotal
3. Se reputação maliciosa confirmada, adiciona o IP à reference list `known_malicious_ips`
4. Notifica o analista via canal configurado

## Limitações conhecidas / débito técnico

- Acesso a uma instância Chronicle real ainda depende de qualificação comercial ou trial de parceiro — não há mais provisionamento self-service gratuito na plataforma (validado em julho/2026)
- As regras deste repositório não foram testadas contra uma instância live; a validação de sintaxe seguiu a documentação oficial e exemplos curados do curso Google Skills
