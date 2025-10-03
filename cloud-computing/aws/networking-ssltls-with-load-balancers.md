# AWS Networking: SSL/TLS com Load Balancers

## TL;DR

Este documento detalha o uso e a configuração de SSL/TLS em Load Balancers na AWS, com foco no processo de *TLS offloading* (também conhecido como *SSL termination*). A estratégia consiste em centralizar o trabalho computacionalmente caro de criptografia/descriptografia no Load Balancer, liberando os recursos das instâncias de backend para focarem exclusivamente na lógica da aplicação.

---

## Conceitos Fundamentais

### 1. O que é SSL/TLS?

TLS (Transport Layer Security) é o sucessor do SSL (Secure Sockets Layer). É um protocolo criptográfico que garante a segurança e a integridade dos dados em trânsito entre um cliente (ex: browser) e um servidor. Ele opera através de um processo conhecido como *handshake*.

### 2. O Processo de Handshake SSL/TLS

O *handshake* é uma sequência de mensagens que estabelece uma conexão segura. De forma simplificada:

1.  **Client**: O cliente envia uma mensagem inicial, informando as versões do TLS e os algoritmos de criptografia que suporta.
2.  **Server**: O servidor responde escolhendo as versões e os algoritmos e envia seu certificado digital.
3.  **Autenticação**: O cliente valida o certificado do servidor junto a uma Autoridade Certificadora (CA).
4.  **Troca de Chave**: O cliente gera uma chave de sessão (pre-master secret), a criptografa com a chave pública do servidor (presente no certificado) e a envia.
5.  **Criação das Chaves de Sessão**: Tanto o cliente quanto o servidor usam a chave de sessão para gerar chaves simétricas idênticas.
6.  **Comunicação Criptografada**: A partir deste ponto, toda a comunicação entre as partes é criptografada com as chaves de sessão.

### 3. SSL/TLS Offloading com Load Balancers

*TLS offloading* é a prática de mover o processo de handshake e a descriptografia do tráfego HTTPS do servidor de aplicação para o Load Balancer.

-   **Cliente -> Load Balancer**: A comunicação ocorre via HTTPS, totalmente criptografada. O Load Balancer gerencia o certificado e realiza o handshake.
-   **Load Balancer -> Instância de Backend**: Após descriptografar a requisição, o Load Balancer a encaminha para a instância de backend em HTTP (tráfego não criptografado) dentro da VPC.

Isso cria uma arquitetura onde a complexidade dos certificados e o custo de CPU da criptografia são centralizados, simplificando a gestão e otimizando a performance dos servidores.

---

## Tipos de Load Balancer na AWS e o Manuseio de TLS

A escolha do Load Balancer impacta como o TLS é gerenciado.

### Application Load Balancer (ALB)

-   **Camada 7 (Aplicação)**: Opera no nível de requisições HTTP/HTTPS.
-   **Funcionalidade Principal**: É ideal para *TLS offloading*. Ele termina a conexão HTTPS, inspeciona o conteúdo da requisição (headers, paths) e a roteia para os *target groups* apropriados.
-   **Header `X-Forwarded-For`**: Automaticamente injeta este header com o IP original do cliente, permitindo que a aplicação de backend tenha visibilidade de quem iniciou a requisição.
-   **Gerenciamento de Certificados**: Integra-se nativamente com o AWS Certificate Manager (ACM) para provisionamento e renovação automática de certificados.

### Network Load Balancer (NLB)

-   **Camada 4 (Transporte)**: Opera no nível de conexões TCP/UDP. É projetado para altíssima performance e baixa latência.
-   **Modos de Operação para TLS**:
    1.  **TLS Termination**: Similar ao ALB, o NLB pode terminar a conexão TLS e enviar o tráfego descriptografado para os alvos.
    2.  **TLS Passthrough**: O NLB pode simplesmente encaminhar o tráfego TCP criptografado diretamente para as instâncias de backend (na porta 443, por exemplo). Nesse cenário, a própria instância é responsável por descriptografar a requisição e gerenciar o certificado. Isso é útil para casos de uso que exigem criptografia de ponta a ponta (end-to-end encryption) ou requisitos de conformidade específicos.
-   **Mutual TLS (mTLS)**: O NLB suporta mTLS, onde tanto o cliente quanto o servidor se autenticam mutuamente usando certificados.

## Benefícios do TLS Offloading

1.  **Otimização de Performance**: Libera ciclos de CPU e memória dos servidores de backend, que não precisam mais gastar recursos com criptografia.
2.  **Gerenciamento Centralizado de Certificados**: Simplifica a instalação, renovação e monitoramento dos certificados em um único ponto (o Load Balancer), em vez de em dezenas ou centenas de servidores.
3.  **Arquitetura Simplificada**: As aplicações de backend podem ser configuradas para servir apenas HTTP, simplificando seu setup e configuração.
4.  **Flexibilidade e Segurança**: Permite inspecionar o tráfego na Camada 7 (com ALBs) para tomar decisões de roteamento avançadas, ao mesmo tempo que mantém a segurança na borda da sua rede.

### Dica

Ao configurar serviços em contêineres (ECS, Kubernetes) por trás de um ALB, lembre-se de que a comunicação entre o ALB e o contêiner dentro da VPC é segura e, na maioria dos casos, pode ser feita via HTTP. Isso simplifica drasticamente a imagem Docker e o entrypoint do seu serviço, pois você não precisa incluir um servidor web complexo como NGINX ou Apache apenas para lidar com TLS dentro do contêiner. Deixe o Load Balancer fazer o trabalho pesado. Foco na aplicação.