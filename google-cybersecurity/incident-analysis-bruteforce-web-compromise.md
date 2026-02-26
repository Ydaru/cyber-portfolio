# Security incident Analysis – Web Application Compromise via Brute Force

## Contexto

Relatório baseado em cenário simulado de incidente envolvendo comprometimento de aplicação web após falha em política de senhas e ataque de força bruta.

## Objetivo

- Identificar o protocolo envolvido
- Analisar o vetor de ataque
- Avaliar o impacto
- Recomendar medidas de mitigação

---

## Protocolo Envolvido

O protocolo identificado no incidente foi:

**HTTP (Hypertext Transfer Protocol)**

O ataque ocorreu por meio da interface web administrativa da aplicação.

---

## Descrição do Incidente

O incidente envolveu um ex-funcionário que conseguiu acessar a página administrativa do site após explorar fragilidades na política de senhas.

Foi identificado que:

- Não havia política de senha robusta.
- O sistema permitia múltiplas tentativas de login sem bloqueio.
- Não havia mecanismo de autenticação multifator.

O invasor realizou um ataque de brute force até obter acesso à conta administrativa.

Após obter acesso, o atacante:

- Modificou o código-fonte da aplicação.
- Inseriu conteúdo malicioso.
- Fez com que o site "yummyrecipesforme.com" distribuísse malware.
- Implementou redirecionamento para outro domínio ("greatrecipesforme.com").

---

## Impacto

- Comprometimento da integridade do site
- Distribuição de malware a usuários
- Danos reputacionais
- Possível comprometimento de dados de visitantes

---

## Causa Raiz

- Política de senha inadequada
- Ausência de limitação de tentativas de login
- Falta de autenticação multifator
- Falha no processo de desligamento de funcionário (revogação de acesso)

---

## Recomendações

- Implementação de política de senhas fortes
- Bloqueio de conta após múltiplas tentativas falhas
- Implementação de autenticação multifator (MFA)
- Processo formal de revogação imediata de acessos para ex-funcionários
- Monitoramento de acessos administrativos
- Uso de WAF (Web Application Firewall)

---

## Conclusão

O incidente analisado caracteriza um comprometimento de aplicação web decorrente de falhas em controles de autenticação. A ausência de política de senha robusta e mecanismos de proteção contra brute force permitiu acesso não autorizado e modificação maliciosa da aplicação.
