# Write-up – Análise de Superfície de Ataque (TomTom)

## Objetivo

Este documento reúne anotações técnicas realizadas durante estudo exploratório
da aplicação web da TomTom (ambiente consumidor e desenvolvedor),
com foco na compreensão dos mecanismos de autenticação, autorização,
gestão de sessão e possíveis vetores de vulnerabilidade.

Importante: Trata-se de análise educacional e exploratória. Nenhuma vulnerabilidade confirmada foi identificada durante o estudo.

---

# 1. Mapeamento de Contas

## Contas Consumidor

Foram utilizadas duas contas distintas para testes de comportamento:

- Conta 1 – ID observado: [ID_1]  
- Conta 2 – ID observado: [ID_2]  

A diferença de IDs foi utilizada para testar possíveis cenários de IDOR.

## Conta Desenvolvedor

- ID observado: [ID_3]  

Conta utilizada para testar funcionalidades relacionadas a API e SDK.

---

# 2. Análise de Autenticação

## Protocolo

A aplicação utiliza:

- SAML
- SSO (Single Sign-On)
- RelayState
- OAuth2 Tokens

### Observações

- O fluxo de login redireciona para endpoint contendo múltiplos parâmetros.
- O parâmetro `sessionDataKey` é gerado no carregamento da página de login.
- O `sessionDataKey` não é regenerado ao recarregar a página.
- Possível cenário teórico de sequestro de sessão caso o link seja interceptado antes da expiração.

---

## Tokens Identificados

Na área autenticada, foram identificados os seguintes tokens:

- web_token_s
- web_token_b
- web_token_w (em alguns casos)
- oauth2_token
- web_logged_in

Observação:
- web_token_s e web_token_b frequentemente apresentam o mesmo valor.
- Tokens são regenerados a cada nova sessão.

Testes realizados:
- Manipulação manual de tokens
- Tentativas de reutilização entre contas
- Testes básicos para IDOR

Resultado:
Não foi possível explorar falha de autorização.

---

# 3. Análise de Autorização

Apesar da identificação dos IDs das contas,
não foi possível realizar tomada de conta via modificação direta de parâmetros.

Observação relevante:
Mesmo após logout, o sistema ainda identificava ID associado via RelayState em determinados fluxos.

Não foi confirmado impacto explorável.

---

# 4. Área Desenvolvedor

## API & SDK Keys

Foi possível criar chaves customizadas.

Cada chave possui:
- Um valor secreto
- Um ID fixo (UUID)

Observação importante:
Ao realizar rotação de chave, apenas o valor secreto é alterado.
O ID permanece o mesmo.

Potencial análise futura:
- Verificar se ID isolado pode ser utilizado indevidamente
- Testar exposição indevida de chaves via frontend

---

# 5. Análise de Funcionalidades

## Basket

- Remoção de produto via requisição PUT.
- Não é possível adicionar múltiplas assinaturas do mesmo produto.

Nenhum comportamento inesperado foi identificado.

## Orders

Pedidos possuem ID sequencial (exemplo: 841307).
Possível vetor teórico para teste de IDOR em pedidos, porém não foi identificado acesso indevido.

---

# 6. Possíveis Vetores Investigados

- IDOR entre contas consumidor
- Reutilização de tokens
- Manipulação de OAuth2 token
- Sequestro de sessionDataKey
- Enumeração de OrderID
- Acesso a endpoint administrativo (/admin)

Nenhuma vulnerabilidade explorável foi confirmada.

---

# 7. Conclusão

O estudo permitiu compreender:

- Arquitetura de autenticação baseada em SAML/SSO
- Gestão de sessão e tokens
- Separação entre ambientes consumidor e desenvolvedor
- Estrutura de rotação de chaves de API
- Controle de autorização aparentemente robusto

Embora nenhuma vulnerabilidade tenha sido identificada, a análise contribuiu para aprimorar a compreensão prática de mecanismos de autenticação e controle de acesso em aplicações web reais.
