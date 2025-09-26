# Padrão Canary Release

Estratégia de implantação que libera uma nova versão para um subconjunto de usuários, permitindo testes graduais antes do rollout completo.

## Sumário

- [Padrão Canary Release](#padrão-canary-release)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Kubernetes](#exemplo-com-kubernetes)

## Componentes

- **Versão Estável**: Aplicação atual em produção.
- **Versão Canary**: Nova versão implantada para um grupo pequeno de usuários.
- **Roteador/Balanceador**: Direciona tráfego para versões (ex.: Istio, Nginx, Kubernetes Ingress).
- **Monitoramento**: Coleta métricas para avaliar a versão Canary (ex.: Prometheus).

## Benefícios

- Baixo risco: Testa mudanças em pequena escala.
- Feedback rápido: Identifica problemas antes do rollout total.
- Rollback fácil: Remove Canary se houver falhas.
- Flexibilidade: Ajusta tráfego gradualmente.

## Contras

- Complexidade: Configuração de roteamento e monitoramento.
- Custos: Recursos adicionais para ambiente Canary.
- Consistência: Possíveis discrepâncias entre versões.
- Monitoramento intensivo: Requer observação contínua.

## Quando Usar

Use para aplicações críticas onde mudanças precisam ser validadas com usuários reais sem impactar todos (ex.: APIs, e-commerce).

## Exemplo com Kubernetes

### Definição do Serviço (Versão Estável e Canary)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: app
        image: myapp:2.0
        ports:
        - containerPort: 8080
```

### Serviço Kubernetes

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### Configuração do Istio (Roteamento Canary)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp-service
  http:
  - route:
    - destination:
        host: myapp-service
        subset: stable
      weight: 90
    - destination:
        host: myapp-service
        subset: canary
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp-dr
spec:
  host: myapp-service
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```