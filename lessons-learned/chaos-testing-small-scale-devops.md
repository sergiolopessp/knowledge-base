# Chaos Testing em Pequena Escala: A Etapa Desaparecida no DevSecOps

**Autor:** Sergio Lopes | **Data:** Outubro de 2025
**Referências:** [DEV Community - Small-Scale Chaos Testing: The Missing Step Before Production](https://dev.to/gkoos/small-scale-chaos-testing-the-missing-step-before-production-30k2)

## O Conceito de "Chaos Light"

A Engenharia de Caos (Chaos Engineering) é frequentemente associada a grandes empresas (Netflix, Amazon) que desligam servidores em produção. Para equipes menores ou microsserviços individuais, o verdadeiro valor é o **Chaos Testing em pequena escala** nos ambientes de **Dev/Staging**.

O objetivo não é quebrar a infraestrutura, mas sim **descobrir premissas falsas** e **testar resiliência** em cenários de falha simples (e muito comuns) antes que o código chegue ao usuário.

---

## Por que Fazer o "Shift-Left" do Caos

Como Engenheiro Sênior e Cloud-Native, vejo que a principal fragilidade reside nas falhas de integração e na **experiência do usuário (UX)** sob condições adversas:

1.  **Exposição de Falhas de Frontend:** Uma falha simples (como uma API que retorna um *timeout* ou um *HTTP 500*) pode causar um *crash* no frontend, um loop infinito, ou travar a aplicação, mesmo que o sistema de *backend* principal esteja se recuperando corretamente.
2.  **Validação de Padrões de Resiliência:** Você implementou o **Circuit Breaker** em Java/Go? O Chaos Testing em Dev/Staging é a maneira mais barata e rápida de provar que ele está funcionando com os *timeouts* e *fallbacks* corretos.
3.  **Ambiente Controlado:** É mais fácil e seguro introduzir latência de 10 segundos ou 10% de erros aleatórios em um *container* de teste do que orquestrar isso em pré-produção.

---

## Onde Injetar o Caos (Cloud e Container Context)

O Chaos Testing de pequena escala não requer ferramentas de infraestrutura pesada (os *Big Guns* como Chaos Monkey), mas sim alavancar o que já está na camada de desenvolvimento e CI/CD.

| Camada | Ponto de Injeção | Ferramentas/Técnicas |
| :--- | :--- | :--- |
| **Código/Backend (Java/Go)** | Comunicação entre serviços (RPC/HTTP). | **Custom Middleware/Interceptors:** Implemente uma função que injete *latency* ou *HTTP 500* aleatório em 5% das chamadas para um serviço crítico. |
| **Frontend/Serviços Mockados** | Respostas de API. | **Mock Service Worker (MSW):** Configure mocks para simular *timeouts* ou respostas de erro malformadas para testar o tratamento de erro da UI. |
| **Proxy/Rede (Docker)** | Tráfego de rede para um container. | **Toxiproxy (Local/Dev):** Use um proxy leve para simular latência, *downstream* e *upstream* throttling ou quedas de conexão para um serviço específico em seu ambiente Docker Compose. |

### Exemplo Prático com Docker

Um truque interessante é usar [**Toxiproxy**](https://github.com/Shopify/toxiproxy) em ambientes locais/CI. Ele permite simular o caos da rede de forma isolada, sem tocar na infraestrutura Cloud.

1.  **Defina o `toxiproxy` no `docker-compose.yml`.**
2.  **Direcione o Container A para o Proxy.**
3.  **Injete a Falha:** Use a API do Toxiproxy para adicionar uma toxina de latência de 500ms no tráfego do Serviço B antes de rodar os testes de integração.

**Conclusão:** O Chaos Testing em pequena escala é a melhor forma de garantir que o **código** e a **UX** sejam resilientes, movendo a qualidade para a esquerda (Shift-Left Quality) e reduzindo a probabilidade de surpresas dolorosas em produção.