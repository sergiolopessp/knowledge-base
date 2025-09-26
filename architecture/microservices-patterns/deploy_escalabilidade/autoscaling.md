# Padrão Autoscaling

Estratégia que ajusta automaticamente o número de instâncias de um serviço com base na demanda, otimizando recursos e desempenho.

## Sumário

- [Padrão Autoscaling](#padrão-autoscaling)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Kubernetes](#exemplo-com-kubernetes)

## Componentes

- **Recurso Escalonável**: Serviço ou aplicação (ex.: contêineres, VMs).
- **Métricas**: Dados monitorados (ex.: CPU, memória, tráfego).
- **Orquestrador**: Gerencia escalonamento (ex.: Kubernetes HPA, AWS Auto Scaling).
- **Regras de Escalonamento**: Definidas para aumentar/reduzir instâncias.

## Benefícios

- Eficiência: Aloca recursos conforme demanda.
- Resiliência: Lida com picos de tráfego.
- Economia: Reduz custos em períodos de baixa demanda.
- Automação: Minimiza intervenção manual.

## Contras

- Complexidade: Configuração e tuning de regras.
- Latência: Atraso no escalonamento pode impactar desempenho.
- Custos iniciais: Monitoramento e orquestração requerem investimento.
- Limites: Depende da capacidade da infraestrutura.

## Quando Usar

Use em aplicações com cargas variáveis, como APIs, sites de e-commerce ou sistemas que enfrentam picos sazonais.

## Exemplo com Kubernetes

### Deployment do Serviço

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

### Configuração do Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```