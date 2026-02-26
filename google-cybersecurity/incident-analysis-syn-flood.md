# Incident Analysis – Suspected SYN Flood Attack

## Contexto

Este relatório é baseado em um cenário simulado de interrupção de serviço em um servidor web, onde usuários relataram erro de timeout ao tentar acessar o site.

## Objetivo

Identificar:
- O possível tipo de ataque
- Como o ataque impactou o servidor
- Evidências nos logs
- Recomendações iniciais de mitigação

---

## Identificação do Incidente

Os logs indicaram um volume anormalmente alto de requisições TCP SYN provenientes de um endereço IP desconhecido.

Com base nesses indícios, o evento foi classificado como um possível:

**SYN Flood (ataque de negação de serviço – DoS).**

---

## Análise Técnica

### Funcionamento normal do TCP (Three-Way Handshake)

1. O cliente envia um pacote SYN ao servidor.
2. O servidor responde com SYN-ACK.
3. O cliente retorna com ACK, estabelecendo a conexão.

### Comportamento observado

O servidor recebeu grande quantidade de pacotes SYN, mas as conexões não foram completadas (ausência de ACK final).

Isso resulta em:
- Conexões semiabertas
- Consumo excessivo da tabela de conexões
- Sobrecarga de recursos
- Timeout para usuários legítimos

---

## Impacto

- Indisponibilidade do site
- Possível exaustão de recursos do servidor
- Interrupção do serviço para usuários legítimos

---

## Recomendações

- Implementação de SYN cookies
- Configuração de rate limiting
- Uso de firewall com proteção contra DoS
- Monitoramento contínuo de tráfego de rede
- Análise mais aprofundada do IP de origem

---

## Conclusão

O incidente analisado apresenta características compatíveis com um ataque SYN Flood. A evidência principal é o alto volume de pacotes SYN não finalizados, resultando em esgotamento de recursos do servidor e indisponibilidade do serviço.
