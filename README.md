# Auditor de Segurança CI/CD e Hardening da Cadeia de Suprimentos de Software

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guia técnico de engenharia e central corporativa de cibersegurança defensiva para auditoria e imunização de pipelines de automação (GitHub Actions, GitLab CI) e mitigação contra ataques de Software Supply Chain.

***

## Sumário Executivo

Este repositório fornece a especificação arquitetural de segurança defensiva, matriz técnica comparativa de posturas de execução e um utilitário CLI determinístico desenvolvido em Python para validação estrutural de manifestos YAML de CI/CD.

* **Organização:** TAVARDT-Portugal
* **Hub Oficial:** [https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/](https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/)
* **Padrões de Referência:** CIS Software Supply Chain Security Guide, OWASP Top 10 for CI/CD, SLSA Framework Level 3.

***

## Vetores Críticos Inspecionados

1. **Gatilhos Privilegiados (`pull_request_target`):** Detecção de checkout de refs não confiáveis em workflows com acesso a secrets.
2. **Permissões Globais Excessivas (`write-all`):** Imposição da política de privilégio mínimo (`permissions: contents: read`).
3. **Actions de Terceiros sem Pinning Imutável:** Identificação de tags mutáveis (`@v3`, `@main`) e conversão para commits SHA de 40 caracteres.
4. **Injeção de Expressões em Scripts Shell:** Sanitização de variáveis de contexto do GitHub em blocos `run:`.
5. **Egress Network Filtering:** Monitoramento e bloqueio de exfiltração de credenciais em runtime de compilação.

***

## Execução do Utilitário CLI

Para executar a auditoria determinística diretamente no terminal ou integrar à esteira corporativa:

```bash
# 1. Obter o script oficial
curl -sO https://raw.githubusercontent.com/TAVARDT-Portugal/ci-cd-security-auditor-pt/main/audit_ci_cd_security.py

# 2. Executar análise estática contra workflows locais
python audit_ci_cd_security.py .github/workflows/
```

***

© 2026 TAVARDT. Todos os direitos reservados.
