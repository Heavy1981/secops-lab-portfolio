# Google SecOps — Detection Engineering Portfolio

**Autor:** Fabiano Rodrigues Santana
**Foco:** SIEM/SOAR (Google Security Operations / Chronicle) — Detection Engineering com YARA-L 2.0

## Objetivo do projeto

Este repositório documenta o desenvolvimento de regras de detecção customizadas para o Google Security Operations (SecOps/Chronicle), como parte do meu aprofundamento técnico em SIEM cloud-native, complementando minha experiência prática de ~4 anos como SOC Analyst N2 (Splunk, QRadar, EDR/XDR, DarkTrace).

As regras aqui foram desenvolvidas com base em estudo aprofundado da documentação oficial do Google SecOps, do curso "Google Security Operations - SIEM" (Google Skills) e da sintaxe YARA-L 2.0, seguindo o padrão de mapeamento MITRE ATT&CK.

> **Nota de transparência:** as regras foram escritas e validadas sintaticamente com base na documentação oficial e exemplos curados do Google. Ainda não foram executadas contra uma instância de produção do Chronicle (acesso pendente de qualificação comercial/trial). O objetivo deste repositório é demonstrar profundidade técnica em detection engineering e domínio da linguagem YARA-L — não simular um deploy real.

## Arquitetura de referência (proposta)

```
┌─────────────┐      ┌───────────┐      ┌──────────────────┐      ┌─────────────────┐
│  VPS Linux  │ ───> │ Bindplane │ ───> │  Google SecOps    │ ───> │  Detection       │
│  (auth.log, │      │  (OTel    │      │  (UDM Ingestion)  │      │  Engine (YARA-L) │
│  ufw, n8n)  │      │  Collector)│      │                   │      │                  │
└─────────────┘      └───────────┘      └──────────────────┘      └────────┬─────────┘
                                                                             │
                                                                             v
                                                                    ┌─────────────────┐
                                                                    │  SOAR Playbook   │
                                                                    │  (VirusTotal +   │
                                                                    │  auto-block)     │
                                                                    └─────────────────┘
```

Bindplane substitui o legado Chronicle Forwarder (descontinuado em janeiro/2027), usando OpenTelemetry como padrão de coleta.

## Regras de detecção

| Regra | Tipo | MITRE ATT&CK | Descrição |
|---|---|---|---|
| [`ssh_brute_force_detection.yaral`](rules/ssh_brute_force_detection.yaral) | Single-event | T1110 (Brute Force) | Detecta múltiplas falhas de autenticação SSH do mesmo IP em janela curta |
| [`successful_login_after_brute_force.yaral`](rules/successful_login_after_brute_force.yaral) | Multi-event (correlação) | T1110 | Correlaciona falhas repetidas seguidas de login bem-sucedido — indício de comprometimento |
| [`known_malicious_ip_connection.yaral`](rules/known_malicious_ip_connection.yaral) | Single-event + Reference List | T1071 (C2) | Detecta conexão originada de IP presente em lista de ameaças conhecidas |
| [`mimikatz_execution_detected.yaral`](rules/mimikatz_execution_detected.yaral) | Single-event | T1003 (Credential Access) | Detecta execução do Mimikatz via linha de comando (Sysmon Event Code 1) |

## Stack e ferramentas de referência

- **SIEM**: Google Security Operations (Chronicle) — UDM, YARA-L 2.0
- **Ingestão**: Bindplane (OpenTelemetry Collector)
- **SOAR**: Playbook conceitual replicando lógica já validada em produção no meu próprio lab n8n (webhook → VirusTotal → decisão automatizada)
- **Threat Intel**: Reference lists populadas com IOCs de investigações reais que conduzi (ex: análise de e-commerce fraudulento)

## Próximos passos

- [ ] Obter acesso a instância Chronicle (trial comercial ou parceria) para validação em ambiente live
- [ ] Rodar retrohunt e capturar evidência de detecção real
- [ ] Expandir regras para cobrir cenários de lateral movement e exfiltração

## Contato

Fabiano Rodrigues Santana
SOC Analyst N2 | Founder @ k0de
