# Docker Volumes

> **Resumo:** Guia rápido sobre persistência de dados em contêineres utilizando Docker Volumes. Essencial para manter dados seguros independentemente do ciclo de vida do contêiner.

---

## Conceito
Contêineres são efêmeros por design. Se um contêiner morre, o que está no seu *writable layer* morre com ele. **Volumes** são a solução nativa do Docker para persistir dados, compartilhar informações entre contêineres e facilitar backups, desacoplando o armazenamento do ciclo de vida da aplicação.

### Benefícios
1.  **Persistência:** Os dados sobrevivem à remoção do contêiner.
2.  **Compartilhamento:** Múltiplos contêineres podem montar e acessar o mesmo volume.
3.  **Gerenciamento:** Mais fácil de fazer backup ou migrar do que *bind mounts* dependentes do host.

---

## 🛠 Comandos Essenciais (CRUD)

### 1. Criar um Volume
O Docker gerencia onde e como os dados são armazenados (geralmente em `/var/lib/docker/volumes/` no Linux).

```bash
docker volume create myvolume
```

Dica de Captain: Sempre use nomes descritivos para seus volumes. Em produção, isso salva vidas na hora de depurar.

### 2. Usar/Montar um Volume
Ao rodar um contêiner, usamos a flag -v (ou --mount) para mapear o volume para um diretório dentro do contêiner.

Exemplo Prático (MySQL):

```Bash

docker run -d \
  -v myvolume:/var/lib/mysql \
  -p 3306:3306 \
  --name mycontainer \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  mysql
```

Entendendo as flags:

-d: Executa em background (detached).

-v myvolume:/var/lib/mysql: O Pulo do Gato. Mapeia o volume myvolume (host) para /var/lib/mysql (contêiner). Tudo gravado ali será persistido.

-p 3306:3306: Expõe a porta do banco.

--name: Nomeia o contêiner para fácil referência.

### 3. Inspecionar um Volume
Útil para descobrir o Mount Point físico no host e metadados.

```Bash

docker volume inspect myvolume
```

### 4. Remover um Volume
Para limpar a casa. Cuidado: isso apaga os dados permanentemente.

```Bash

docker volume rm myvolume
```

Atenção: O Docker não remove volumes automaticamente quando você remove um contêiner. Você deve garantir que nenhum contêiner (mesmo parado) esteja usando o volume antes de tentar removê-lo.

## Notas Adicionais
Volumes vs Bind Mounts: Prefira Volumes (docker volume create) para bancos de dados e persistência de aplicação. Use Bind Mounts (-v /path/host:/path/container) apenas quando precisar editar código fonte do host em tempo real (ambiente de desenvolvimento).

Drivers: Por padrão, usa-se o driver local, mas em ambientes de cloud ou cluster (como Swarm/K8s), você pode usar drivers para armazenar volumes em storages de rede (NFS, S3, EBS), garantindo que os dados sigam o contêiner entre nós diferentes.