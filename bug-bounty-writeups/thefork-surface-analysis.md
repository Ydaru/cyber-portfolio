# Write-up – Análise de Superfície de Ataque (TheFork)

## Objetivo

Este documento reúne anotações técnicas realizadas durante estudo exploratório da aplicação web da TheFork, com foco na análise de:

* Operações GraphQL
* Controle de autorização
* Manipulação de reservas
* Sistema de reviews
* Interações entre múltiplas contas

Importante:
Trata-se de análise educacional e exploratória. Nenhuma vulnerabilidade confirmada foi identificada durante o estudo.

---

# 1. Mapeamento de Contas

Foram utilizadas três contas distintas para testes de comportamento e autorização cruzada.

## Conta 1

* UserID observado: [USER_ID_1]

## Conta 2

* UserID observado: [USER_ID_2]

## Conta 3

* Conta criada para testes adicionais (ID não identificado via interface padrão)

As contas foram utilizadas para testar possíveis cenários de:

* IDOR entre usuários
* Manipulação de reservas
* Interações indevidas entre reviews
* Testes de autorização em operações GraphQL

---

# 2. Arquitetura Identificada – GraphQL

A aplicação utiliza GraphQL como principal mecanismo de comunicação entre frontend e backend.

Foi identificada a operação:

### CreateNewCustomerAccount

Mutation responsável por criação de contas:

```graphql
mutation CreateNewCustomerAccount($customer: CreateCustomerInput!) {
  createCustomer(customer: $customer) {
    id
    __typename
  }
}
```

### Observações

* A criação de conta ocorre integralmente via mutation GraphQL.
* A requisição contém objeto `device` com fingerprint.
* Contas criadas via mutation apresentaram comportamento ligeiramente diferente na exposição de headers.
* Não foi possível escalar privilégios via manipulação da mutation.

Apesar da aparente liberdade de manipulação dos parâmetros da query, não foi possível burlar controles de autorização.

---

# 3. Análise de Reservas (Bookings)

Foram analisadas as seguintes operações:

* `UserSpaceCancelReservation`
* `EditReservation`

### Observações

* É possível interceptar e modificar parâmetros via request.
* Alterações só são aceitas quando associadas a reservas válidas.
* Reservas expiradas retornam erro adequado.
* Não foi possível modificar reserva pertencente a outro usuário.

Conclusão:
Controle de autorização vinculado à reserva e sessão parece estar funcionando corretamente.

---

# 4. Sistema de Reviews

## Operações Identificadas

### LikeReview

Mutation utilizada para curtir avaliações.

### UnlikeReview

Mutation utilizada para remover curtida.

Observação:

* As mutations podem ser replicadas manualmente via interceptação de request.
* Não foi observada validação adicional visível além da sessão autenticada.
* Não foi possível interagir com reviews de forma indevida entre contas.

---

## Fluxo de Criação de Review

Identificadas duas etapas principais via mutation `UpsertReview`.

### Mecanismos de Proteção Identificados

* Necessidade de `inviteToken` válido.
* Associação obrigatória com `reservationId`.
* Token com tempo de expiração.
* Token contém permissões específicas de rating.
* Presença de roles no payload do token.

Observação importante:

A presença do `inviteToken` impede:

* Manipulação de review de outro usuário
* Criação de review sem reserva válida
* Alteração arbitrária de avaliações

O controle de autorização aparenta estar corretamente implementado.

---

# 5. Sistema de Rating Global

Operação identificada:

### GlobalRating

Query que calcula rating agregado com base em:

* ambienceRating
* foodQualityRating
* serviceRating

Trata-se de cálculo de média/sentimento, não de submissão direta de review.

Não foi identificado vetor de exploração.

---

# 6. Pontos de Atenção Investigados

Durante a análise foram considerados os seguintes vetores:

* Manipulação de mutations GraphQL
* Tentativa de privilege escalation via CreateNewCustomerAccount
* IDOR entre reservas
* Manipulação manual de reviewUuid
* Reutilização ou modificação de inviteToken
* Testes de integridade entre múltiplas contas

Nenhum comportamento explorável foi confirmado.

---

# 7. Conclusão

Embora nenhuma vulnerabilidade tenha sido identificada, o estudo permitiu aprofundar conhecimentos em:

* Funcionamento prático de GraphQL em produção
* Estrutura de mutations e queries
* Controle de autorização baseado em token
* Validação vinculada a reservas
* Uso de JWT com escopos e permissões
* Separação lógica entre autenticação e autorização

A análise reforçou a importância de:

* Entender fluxo completo da aplicação antes de testar exploração
* Identificar onde realmente ocorre a validação (backend vs frontend)
* Não assumir vulnerabilidade apenas pela possibilidade de manipular request

Este estudo contribuiu significativamente para o desenvolvimento técnico em análise de aplicações modernas baseadas em GraphQL.

