# Padrão Secrets Management

Estratégia para gerenciar, armazenar e acessar credenciais sensíveis (ex.: senhas, chaves API) de forma segura em sistemas.

## Sumário

- [Padrão Secrets Management](#padrão-secrets-management)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot e Vault](#exemplo-com-spring-boot-e-vault)

## Componentes

- **Segredo**: Dados sensíveis (ex.: senhas, tokens, chaves).
- **Gerenciador de Segredos**: Ferramenta para armazenamento seguro (ex.: HashiCorp Vault, AWS Secrets Manager).
- **Integração**: Mecanismo para acessar segredos (ex.: APIs, SDKs).
- **Controle de Acesso**: Define quem pode acessar os segredos.

## Benefícios

- Segurança: Armazenamento criptografado e acesso controlado.
- Rotação: Facilita atualização de credenciais.
- Auditoria: Rastreia quem acessou os segredos.
- Centralização: Gerenciamento unificado para múltiplos serviços.

## Contras

- Complexidade: Configuração e integração com ferramentas.
- Custo: Ferramentas externas podem ter custos associados.
- Dependência: Relia na disponibilidade do gerenciador.
- Curva de aprendizado: Requer conhecimento da ferramenta escolhida.

## Quando Usar

Use em aplicações que manipulam dados sensíveis, como microservices, APIs ou sistemas em nuvem (ex.: fintech, saúde).

## Exemplo com Spring Boot e Vault

###