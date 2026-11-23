# CI/CD Security Auditor & Software Supply Chain Hardening Hub

Guia técnico de engenharia e central corporativa de cibersegurança defensiva para auditoria e imunização de pipelines de automação (GitHub Actions, GitLab CI) e mitigação contra ataques de Software Supply Chain.

***

## Sumário Executivo

Este repositório fornece a especificação arquitetural de segurança defensiva, matriz técnica comparativa de posturas de execução e um utilitário CLI determinístico desenvolvido em Python para validação estrutural de manifestos YAML de CI/CD.

* **Organização:** TAVARDT-Portugal
* **Jurisdição:** Portugal
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

## Governança & Conformidade

Desenvolvido pela divisão de Cibersegurança da **TAVARDT Portugal**. Todos os direitos reservados.
