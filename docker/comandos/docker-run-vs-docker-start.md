# Docker Run vs Docker Start

Este artigo é um resumo/commentary do post “Docker Run vs Docker Start” de Shelley Benhoff no DEV Community.  
Ele diferencia os comandos `docker run` e `docker start`, mostra casos de uso e exemplos práticos. ([dev.to](https://dev.to/sbenhoff/docker-run-vs-docker-start-3gcm))  

---

## 1. O que é `docker run`

- O comando **`docker run`** cria **e inicia** um novo container a partir de uma imagem.  
- É como uma operação “dupla”: inicialização + execução.  
- Sintaxe básica:

  ```
  docker run [OPÇÕES] IMAGEM [COMANDO] [ARGUMENTOS]
  ```

- Opções comuns:
  - `-d` — executa em modo “detached” (em background)  
  - `-it` — interação / terminal interativo  
  - `--name` — dar nome ao container  
  - `-p` — mapear porta host → container  
  - `-v` — montar volumes  

### Exemplo

```bash
docker run --name my-nginx -p 8080:80 -d nginx
```

Esse comando:
- cria um novo container com imagem `nginx`,
- atribui o nome `my-nginx`,
- mapeia porta 8080 do host para 80 do container,
- roda em background.

---

## 2. O que é `docker start`

- O comando **`docker start`** **inicia** um container existente que está parado (stopped). Ele **não cria** um novo container.  
- Sintaxe:

  ```
  docker start [OPÇÕES] CONTAINER
  ```

- Opções úteis:
  - `-i` — attach ao STDIN (modo interativo)  
  - `-a` — attach aos logs / saída do container (stdout/stderr)  

### Exemplo

```bash
docker start my-nginx
```

Se `my-nginx` foi criado antes (por `docker run`) e está parado, este comando o inicia novamente.  

---

## 3. Comparando `docker run` vs `docker start`

| Característica | `docker run` | `docker start` |
|----------------|---------------------------|--------------------------------|
| Criação de container | **Sim** — cria novo container | **Não** — usa container já existente |
| Início do container | Sim (imediatamente após criação) | Sim, mas somente se já houver container parado |
| Configurações (portas, variáveis, volumes) | Pode definir no momento da execução | Usa configurações já existentes do container |
| Velocidade de startup | Mais demorado (criação + inicialização) | Mais rápido (apenas iniciar) |
| Preservação de estado / logs | Novo estado (limpo) | Preserva dados, logs, sistema de arquivos conforme estavam |

---

## 4. Casos de uso recomendados

### Quando usar `docker run`

- Quando você ainda **não criou o container** (ex: ambiente novo)  
- Quando precisa alterar parâmetros (portas, variáveis de ambiente, volumes)  
- Quando quer um container “efêmero” / temporário  

### Quando usar `docker start`

- Quando já existe um container criado anteriormente  
- Quando deseja retomar onde parou, preservando logs / dados  
- Quando quer um startup mais rápido (pois ignora a fase de criação)  

---

## 5. Observações adicionais e dicas

- Containers parados ainda ocupam espaço (imagens, volumes) — use `docker rm` / `docker prune` quando não mais necessários.  
- Se você precisa mudar configuração (por exemplo, mapear uma nova porta ou variável), muitas vezes é mais fácil remover e recriar com `docker run`.  
- Cuidado ao usar volumes e dados para garantir que o estado persistente seja mantido conforme o desejado.  
- Usar `docker start -a` pode ser útil para debug ou inspeção de logs de containers já configurados.

---

## Referência

- “Docker Run vs Docker Start” — Shelley Benhoff, DEV Community ([dev.to](https://dev.to/sbenhoff/docker-run-vs-docker-start-3gcm))  
