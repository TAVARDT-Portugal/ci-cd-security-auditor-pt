# Auditor de Segurança CI/CD e Hardening da Cadeia de Fornecimento de Software

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-portugal.github.io/ci-cd-security-auditor-pt/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![CIS Benchmark](https://img.shields.io/badge/CIS-Benchmark-blue.svg)](https://www.cisecurity.org/)
[![OpenSSF](https://img.shields.io/badge/OpenSSF-Scorecard-green.svg)](https://openssf.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Especificação técnica de engenharia defensiva, normas de imunização de esteiras de automação e utilitário determinístico em linha de comandos para proteção da cadeia de fornecimento de software em GitHub Actions e plataformas modernas de CI/CD.

***

## Enquadramento Executivo: O Vetor de Ameaça em Ambientes de CI/CD

As esteiras de Integração e Entrega Contínuas (CI/CD) constituem a espinha dorsal do desenvolvimento de software corporativo. Pela sua posição estratégica, munidas de acesso a credenciais de publicação e ligações diretas a repositórios, as esteiras de compilação tornaram-se o alvo preferencial de agentes hostis para comprometer a cadeia de fornecimento.

Quando as equipas técnicas priorizam a celeridade nas entregas sem controlos perimetrais rigorosos, as esteiras herdam fragilidades arquiteturais de elevado impacto:
* Gatilhos de automação executam código submetido em forks externos dentro de ambientes com privilégios elevados.
* Dependências e actions de terceiros referenciam tags mutáveis passíveis de sequestro de conta.
* Tokens de execução detêm privilégios globais excessivos sobre o código e as versões publicadas.
* Os nós de execução operam sem restrições de tráfego de saída, permitindo a exfiltração de variáveis de ambiente.

Este repositório estabelece um modelo de arquitetura defensiva em profundidade. Disponibiliza configurações prontas para implementação e um analisador estático determinístico em Python (`audit_ci_cd_security.py`) para validação técnica sem telemetria externa.

***

## Superfícies Críticas de Ataque em Esteiras de Automação

### 1. O Risco Estrutural do Gatilho `pull_request_target`
Os fluxos acionados por `pull_request_target` são executados no contexto do repositório base, herdando acesso direto aos segredos corporativos:
* **Mecanismo da Vulnerabilidade:** Esteiras que utilizam este evento para descarregar o commit do pull request (`github.event.pull_request.head.sha`) injetam código não auditado diretamente no nó com privilégios.
* **Impacto Operacional:** Utilizadores externos podem introduzir alterações em scripts de compilação, executar comandos arbitrários e exfiltrar segredos de produção sem revisão da equipa de engenharia.
* **Diretriz Defensiva:** É imperativo não efetuar checkout de código não confiável em fluxos que utilizem `pull_request_target`. Isole tarefas de validação preliminar em eventos convencionais de `pull_request`.

### 2. Actions Externas sem Fixação por Hash SHA Imutável
A invocação de actions de terceiros através de tags de versão (como `@v3` ou `@main`) introduz dependências mutáveis e não verificadas:
* **Mecanismo da Vulnerabilidade:** As tags do Git são referências reconfiguráveis que podem ser manipuladas em caso de comprometimento da conta do criador da biblioteca.
* **Impacto Operacional:** Um commit malicioso introduzido numa biblioteca partilhada infeta de imediato múltiplos pipelines corporativos em simultâneo.
* **Diretriz Defensiva:** Fixe todas as dependências externas exclusivamente através do identificador SHA de 40 carateres hexadecimais, acompanhado de menção à versão auditada.

### 3. Tokens de Execução com Privilégios Globais Excessivos
As esteiras executam frequentemente com permissões amplas herdadas das configurações globais da organização:
* **Mecanismo da Vulnerabilidade:** A ausência de uma secção restritiva `permissions:` atribui ao token privilégios de escrita sobre o repositório, pacotes e versões publicadas.
* **Impacto Operacional:** Um componente comprometido durante a compilação pode reescrever ramos protegidos ou substituir artefactos de produção.
* **Diretriz Defensiva:** Defina imperativamente `permissions: contents: read` na raiz do manifesto, atribuindo permissões adicionais de escrita apenas aos passos estritamente necessários.

### 4. Ausência de Filtragem de Saída de Rede nos Executores
Os nós de execução operam habitualmente com ligações diretas e ilimitadas para o exterior:
* **Mecanismo da Vulnerabilidade:** Scripts hostis podem estabelecer sessões interativas remotas ou exportar variáveis de ambiente para servidores não identificados.
* **Diretriz Defensiva:** Implemente filtragem perimetral de saída (egress filtering), restringindo o tráfego estritamente aos repositórios oficiais de código e dependências.

***

## Modelos de Mitigação Defensiva em Produção

O repositório fornece modelos técnicos testados para aplicação direta em ambientes de produção:

### 1. Workflow Protegido para GitHub Actions (`hardened-workflow.yml`)
Assegura privilégios mínimos, fixação por commit SHA e proteção de credenciais:

```yaml
name: Enterprise Hardened Pipeline

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  audit-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Hardened Checkout
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
        with:
          persist-credentials: false

      - name: Configuracao Segura de Python
        uses: actions/setup-python@0a5c61591373683505ea898e09a3ea4f39ef2b9c # v5.0.0
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Execucao de Analise Estatica
        run: |
          python audit_ci_cd_security.py .github/workflows/
```

### 2. Script de Verificação de Fixação de Actions (`action-pinning.sh`)
Analisa os manifestos e assegura a utilização de hashes SHA criptograficamente imutáveis:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Inspeciona ficheiros de workflow e valida a utilizacao de hashes SHA de 40 carateres
find .github/workflows -type f \( -name "*.yml" -o -name "*.yaml" \) | while read -r workflow; do
  echo "[*] A auditar workflow: ${workflow}"
  grep -E "uses: [a-zA-Z0-9_\-\.\/]+@[a-f0-9]{40}" "${workflow}" || echo "[!] Actions nao fixadas identificadas"
done
```

### 3. Filtragem de Conexões de Saída do Nó de Execução (`step-security-harden.yml`)
Impede o envio de dados para servidores não autorizados a partir do nó de build:

```yaml
- name: Proteger Conexoes de Saida
  uses: step-security/harden-runner@63c24ba6bd7ba022e95695ff85de572c04a18142 # v2.7.0
  with:
    egress-policy: block
    allowed-endpoints: >
      github.com:443
      api.github.com:443
      pypi.org:443
      files.pythonhosted.org:443
```

***

## Utilitário Determinístico de Auditoria CLI

Os navegadores de internet aplicam mecanismos estritos de sandbox que impedem o acesso a ficheiros locais e a análise de diretórios de infraestrutura. Para disponibilizar avaliações técnicas rigorosas e sem recolha de telemetria externa, a TAVARDT desenvolveu uma ferramenta de análise estática em Python para execução local no terminal ou integração em pipelines de CI/CD.

### Fases da Análise Estática

O utilitário (`audit_ci_cd_security.py`) executa quatro etapas de validação:
* **Auditoria de Gatilhos:** Analisa a declaração de eventos para sinalizar o uso inseguro de `pull_request_target`.
* **Avaliação de Permissões:** Identifica diretivas `write-all` ou a inexistência de parâmetros restritivos de permissão.
* **Inspeção de Dependências:** Valida as declarações `uses:` através de expressões regulares para comprovar a presença de identificadores SHA de 40 carateres.
* **Deteção de Segredos em Texto Aberto:** Verifica comandos de terminal e variáveis de ambiente em busca de credenciais expostas.

### Modo de Utilização

Transfira e execute o utilitário diretamente sobre os manifestos do repositório:

```bash
# 1. Transferir o script oficial
curl -sO https://raw.githubusercontent.com/TAVARDT-Portugal/ci-cd-security-auditor-pt/main/audit_ci_cd_security.py

# 2. Executar analise estatica nas esteiras locais
python audit_ci_cd_security.py .github/workflows/
```

***

## Perguntas Frequentes sobre Arquitetura (FAQ Técnico)

### 1. Porque é o gatilho pull_request_target considerado uma das maiores ameaças no GitHub Actions?
O evento `pull_request_target` corre no contexto do ramo principal com privilégios de escrita e acesso a segredos. Quando combinado com o descarregamento de código submetido a partir de um fork externo, possibilita que utilizadores desconhecidos executem comandos no executor e capturem credenciais de produção.

### 2. De que forma a fixação de actions por commit SHA de 40 carateres neutraliza ataques à cadeia de fornecimento?
As tags como `@v3` ou `@main` são apontadores mutáveis no Git sujeitos a manipulação caso a conta do autor seja acedida indevidamente. O identificador SHA de 40 carateres é criptograficamente imutável, garantindo que apenas o código previamente revisto e validado é executado.

### 3. Qual o perigo associado à omissão da declaração de permissões nos manifestos de automação?
Quando um manifesto não inclui a secção `permissions:`, o token do nó de execução herda as definições padrão da organização, que incluem habitualmente permissões de escrita em múltiplos recursos. Qualquer componente adulterado pode tirar partido desta autorização para alterar versões ou contaminar o código.

### 4. Como é que a filtragem de tráfego de saída (egress filtering) impede a perda de informação sensível?
O controlo de tráfego limita as comunicações de rede do nó de execução unicamente a destinos oficiais previamente autorizados. Na eventualidade de uma dependência tentar estabelecer ligações remotas não autorizadas para descarregar segredos, a ligação é bloqueada imediatamente.

### 5. O analisador estático em Python necessita de ligação à internet ou privilégios de administração?
Não. A ferramenta funciona de forma inteiramente passiva e offline, lendo os ficheiros locais de manifesto sem transmitir quaisquer dados para o exterior e sem exigir privilégios de superutilizador no sistema.

***

## Enquadramento de Conformidade e Normas de Segurança

Arquitetura técnica alinhada com as recomendações do OWASP Top 10 CI/CD Security Risks, com os níveis de maturidade do framework SLSA e com as diretrizes do Centro Nacional de Cibersegurança (CNCS).

***

## Links Úteis

* **Portal Corporativo:** https://ag.tavardt.com/pt/
* **Suíte de Diagnósticos:** https://tools.tavardt.com/pt/
* **Contacto Técnico:** https://ag.tavardt.com/pt/contacto/

***

© 2026 TAVARDT Portugal. Todos os direitos reservados. Soberania e Defesa de Infraestrutura.
