# Auditor de Segurança CI/CD & Detetor de Fuga de Secrets | TAVARDT Portugal

[![Licence](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![CI/CD Security](https://img.shields.io/badge/CI%2FCD-Security%20Hardened-00d26a.svg)](README.md)
[![Language](https://img.shields.io/badge/Language-Português%20(PT)-green.svg)](README.md)
[![Engineering](https://img.shields.io/badge/TAVARDT-Cybersecurity-5b3cf5.svg)](https://ag.tavardt.com/pt/ciberseguranca/)

Auditor estático defensivo e linter de segurança para esteiras de integração e entrega contínua (GitHub Actions). Identifica configurações de alto risco de supply chain, gatilhos vulneráveis a pull requests externos, permissões excessivas no GITHUB_TOKEN e fuga de secrets corporativos.

## Aplicação Web Interativa

Aceda à interface visual para auditar os seus workflows YAML em tempo real diretamente no navegador:

- [https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/](https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/)

---

## 1. Visão Geral do Auditor de Segurança CI/CD

Esteiras de automação e integração contínua tornaram-se o alvo prioritário de campanhas avançadas de supply chain. Atacantes exploram contas comprometidas de programadores e gatilhos mal configurados em repositórios para executar código arbitrário nos runners e exfiltrar variáveis de ambiente protegidas, tokens de nuvem (AWS, GCP, Azure) e credenciais de deploy.

O Auditor de Segurança CI/CD da TAVARDT analisa manifestos YAML de forma passiva e determinística, confrontando cada instrução contra o catálogo de riscos do OWASP Top 10 para CI/CD Security e o framework SLSA.

---

## 2. Vulnerabilidades e Vetores Auditados

| Vetor de Risco | Cenário no Pipeline CI/CD | Impacto Técnico |
| :--- | :--- | :--- |
| **pull_request_target com Checkout Inseguro** | O workflow executa no contexto do branch base e faz checkout do código submetido no PR do atacante (`ref: ${{ github.event.pull_request.head.sha }}`). | Execução remota de código (RCE) imediata no runner com acesso irrestrito a todos os secrets de produção do repositório. |
| **Permissões Excessivas write-all** | O pipeline utiliza `permissions: write-all` ou omite o bloco restritivo de permissões no nível raiz. | O token padrão herda autorização para alterar tags, sobrescrever releases e publicar artefactos adulterados no ecossistema. |
| **Injeção de Expressões Shell no Bloco run:** | Variáveis de contexto geradas por terceiros (ex: `${{ github.event.issue.title }}`) são concatenadas diretamente em linhas de script shell. | Permite quebra de aspas e execução de comandos maliciosos diretos no sistema operativo do runner. |
| **Ações Externas Fixadas por Tag Mutável** | Instruções `uses:` apontam para referências mutáveis como `@v3` ou `@main` em vez de commit SHA imutável. | Expõe o pipeline a supply chain poisoning caso a conta de um mantenedor da ação seja comprometida. |
| **Exfiltração e Credenciais Estáticas** | Ocorrência de chaves de API privadas codificadas diretamente ou comandos `curl` despachando secrets para servidores externos. | Comprometimento instantâneo de infraestruturas conectadas e violação das políticas de governança e conformidade. |

---

## 3. Guia de Instalação e Execução Local (CLI)

O scanner pode ser executado em ambiente de terminal sem dependências externas, utilizando exclusivamente os módulos nativos do Python.

### Clonar o Repositório

```bash
git clone https://github.com/TAVARDT-Portugal/ci-cd-security-auditor-pt.git
cd ci-cd-security-auditor-pt
```

### Auditar um Workflow YAML Específico

```bash
python scanner.py .github/workflows/deploy.yml
```

### Auditar Todo o Diretório de Workflows com Saída JSON

```bash
python scanner.py .github/workflows/ --json
```

---

## 4. Blindagem Profissional e Suporte Técnico

Caso a auditoria tenha identificado fragilidades nas suas esteiras, credenciais expostas ou necessidade de governança Zero Trust com fornecedores de nuvem:

- **Portal Institucional:** [TAVARDT Portugal](https://ag.tavardt.com/pt/ciberseguranca/)
- **Central de Ferramentas:** [Ferramentas Defensivas TAVARDT](https://tools.tavardt.com/pt/)
- **Atendimento com Engenharia:** [Suporte Técnico e Resposta a Incidentes](https://ag.tavardt.com/pt/contato/)

---

## Licença

Distribuído sob a licença de código aberto MIT. Consulte o ficheiro [`LICENSE`](LICENSE) para mais detalhes.  
Copyright (c) 2026 TAVARDT Portugal.
