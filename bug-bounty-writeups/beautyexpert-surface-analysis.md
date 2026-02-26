# Write-up – Análise de Superfície de Ataque (Beauty Expert)

## Objetivo

Este documento reúne anotações técnicas realizadas durante estudo exploratório da aplicação web Beauty Expert (grupo THG), com foco na análise de mecanismos de autenticação, gestão de sessão, funcionamento de cookies e identificação de possíveis vetores de vulnerabilidade.

Importante: Trata-se de análise educacional e exploratória. Nenhuma vulnerabilidade confirmada foi identificada durante o estudo.

---

# 1. Mapeamento de Contas

Foram utilizadas duas contas distintas para análise de comportamento entre sessões.

## Conta 1
- Identificador interno observado: [UUID_1]
- Identificador global THG observado: [THG_ID_1]

## Conta 2
- Identificador interno observado: [UUID_2]
- Identificador global THG observado: [THG_ID_2]

Observação:
O identificador global (thgUserId) aparenta ser utilizado de forma padronizada entre diferentes sites do grupo THG. Hipótese futura: verificar se há segregação adequada entre domínios.

---

# 2. Análise de Autenticação

## Cookies de Sessão

Durante a autenticação foram identificados cookies relevantes para manutenção de sessão:

- Cookie principal de validação de conta (exemplo mascarado):
  `Opaque_beint_en=[TOKEN_SESSAO]`

Observação:
A conta autenticada parecia depender majoritariamente desse cookie específico para validação de sessão.

Testes realizados:
- Reutilização manual do cookie
- Testes cross-account
- Uso do cookie após logout

Resultado:
Não foi identificado bypass de autenticação.

---

# 3. Análise de Autorização

## Basket (Carrinho)

Foi identificado um cookie específico responsável por manter o estado da cesta:

`ElysiumBasketbeint_V6=[BASE64_TOKEN]`

Ao decodificar o valor Base64, foi observado o seguinte padrão:

Testes realizados:

- Reutilização do cookie entre contas diferentes
- Uso do cookie deslogado
- Tentativa de manipulação manual do valor

Observações:

- A cesta era restaurada ao reutilizar o cookie correspondente.
- O valor decodificado sugere identificador único associado à conta.
- Não foi possível manipular o UUID para acessar cesta de outro usuário.

Hipótese descartada:
Manipulação direta do cookie para IDOR em basket.

---

# 4. Análise de Proteções – WAF

Durante testes de injeção (XSS/SQLi) via formulário de contato:

Endpoint analisado:
`/customerQuery.account`

Foi identificado:

- Retorno HTTP 406 para payloads suspeitos.
- Indício de proteção ativa contra requisições maliciosas.

### Investigação do WAF

Foi identificado nos headers de resposta:
Via: 1.1 varnish


Indício inicial de uso de Varnish (reverse proxy/cache).

Posteriormente, foi realizado fingerprint com ferramenta específica:
wafw00f https://[DOMINIO]


Resultado indicou possível uso de:

**NetScaler AppFirewall (Citrix Systems)**

Evidência adicional encontrada em header:

set-cookie: NSC_[...]


Observação:
O prefixo "NSC" é característico de implementações NetScaler.

Ferramenta adicional utilizada:
- identYwaf.py (sem confirmação conclusiva)

Conclusão:
Há forte evidência de proteção por Web Application Firewall (NetScaler AppFirewall).

---

# 5. Testes de Injeção

Foram realizados testes básicos de XSS com:

- Payloads padrão
- Técnicas de evasão (quebra de palavra-chave)
- Obfuscação com caracteres especiais
- Variação de maiúsculas/minúsculas

Resultado:

- Status HTTP 200 em algumas variações
- Erro de renderização da página
- Nenhuma execução confirmada

Nenhuma vulnerabilidade explorável foi identificada.

---

# 6. Possíveis Vetores Investigados

- IDOR entre contas consumidor
- Manipulação de cookie da basket
- Reutilização de sessão
- Bypass de WAF
- Enumeração de identificadores globais (THG ID)
- Upload de arquivos via formulário

Nenhuma vulnerabilidade confirmada.

---

# 7. Conclusão

A análise permitiu compreender:

- Estrutura de autenticação baseada em cookies persistentes
- Funcionamento interno do mecanismo de basket
- Processo de decodificação e análise de tokens Base64
- Identificação e fingerprint de WAF
- Técnicas básicas de evasão e validação de proteção

Embora nenhuma vulnerabilidade tenha sido identificada, o estudo contribuiu significativamente para o entendimento prático de:

- Gestão de sessão
- Controle de autorização
- Proteção de aplicações via WAF
- Investigação estruturada em programas de Bug Bounty
