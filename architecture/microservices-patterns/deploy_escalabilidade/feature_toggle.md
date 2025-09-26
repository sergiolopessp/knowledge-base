# Padrão Feature Toggle

Estratégia que habilita ou desabilita funcionalidades em tempo de execução, permitindo controle sobre a liberação de recursos sem nova implantação.

## Sumário

- [Padrão Feature Toggle](#padrão-feature-toggle)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Spring Boot](#exemplo-com-spring-boot)

## Componentes

- **Feature Flag**: Chave que ativa/desativa uma funcionalidade.
- **Gerenciador de Toggles**: Sistema para configurar flags (ex.: banco de dados, arquivo, serviço externo).
- **Código Condicional**: Lógica que verifica o estado da flag.
- **Monitoramento**: Rastreia uso e impacto das flags.

## Benefícios

- Liberação gradual: Testa funcionalidades com subgrupos de usuários.
- Rollback rápido: Desativa recursos problemáticos sem reimplantação.
- Testes A/B: Compara comportamentos com diferentes flags.
- Flexibilidade: Permite experimentação sem risco.

## Contras

- Complexidade: Aumenta condicionais no código.
- Dívida técnica: Flags antigas precisam ser removidas.
- Gerenciamento: Requer ferramentas para controlar flags.
- Risco de erros: Configurações incorretas podem expor funcionalidades.

## Quando Usar

Use para lançamentos graduais, testes A/B, ou para desativar funcionalidades instáveis em aplicações como APIs, e-commerce ou SaaS.

## Exemplo com Spring Boot

### Configuração do Feature Toggle

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeatureToggleConfig {
    @Value("${feature.new-endpoint.enabled:false}")
    private boolean newEndpointEnabled;

    public boolean isNewEndpointEnabled() {
        return newEndpointEnabled;
    }
}
```

### Serviço com Feature Toggle

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class FeatureService {
    private final FeatureToggleConfig toggleConfig;

    @Autowired
    public FeatureService(FeatureToggleConfig toggleConfig) {
        this.toggleConfig = toggleConfig;
    }

    public String getFeatureResponse() {
        if (toggleConfig.isNewEndpointEnabled()) {
            return "Nova funcionalidade ativa!";
        }
        return "Funcionalidade antiga.";
    }
}
```

### Configuração do Application Properties

```properties
# Desativado por padrão
feature.new-endpoint.enabled=false
```