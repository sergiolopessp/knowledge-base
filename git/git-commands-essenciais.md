# Git: Comandos Essenciais e Dicas de Produtividade

**Autor:** Sergio Lopes | **Data:** Outubro de 2025
**Referência:** [DEV Community - Top 7 Git Commands Every Developer Should Know](https://dev.to/george_hany_899df0785b4fe/top-7-git-commands-every-developer-should-know-46b3)

## Os 7 Fundamentos Diários

Estes comandos são o ciclo de vida básico de qualquer mudança de código em um repositório.

| Comando | Função | Exemplo |
| :--- | :--- | :--- |
| `git init` | Inicializa um novo repositório Git local. | `git init` |
| `git clone` | Cria uma cópia local completa de um repositório remoto. | `git clone <URL_REPO>` |
| `git status` | Exibe o estado da árvore de trabalho e do *staging area*. **Use sempre!** | `git status` |
| `git add .` | Adiciona **todos** os arquivos modificados e não rastreados ao *staging area*. | `git add .` |
| `git commit -m` | Salva as mudanças que estão no *staging area* no histórico local. | `git commit -m "feat: Adiciona logs de monitoramento da JVM"` |
| `git push` | Envia os commits locais para o branch remoto. | `git push origin develop` |
| `git pull` | Busca e *merges* (ou *rebases*) as alterações do repositório remoto. | `git pull` |

---

## Comandos para Colaboração e Refatoração

Para ter produtividade real em um time de Engenharia de Software e manter um histórico limpo, é crucial dominar a manipulação de branches e a reescrita de histórico.

### 1. Manipulação de Branches (`branch` e `switch`)

O trabalho em equipes Cloud-Native exige isolamento de features. `git branch` e `git switch` são vitais.

| Comando | Função | Exemplo |
| :--- | :--- | :--- |
| `git branch <nome>` | Cria um novo branch local. | `git branch feature/gcs-tuning` |
| `git switch <branch>` | Alterna o branch de trabalho. Mais seguro que o `checkout` antigo. | `git switch feature/gcs-tuning` |
| `git switch -c <nome>` | Cria e alterna para o novo branch em um passo. | `git switch -c fix/data-bug` |

### 2. Higiene do Histórico (`rebase`)

Um histórico limpo e linear é obrigatório para *code review* eficiente e *deploy* contínuo.

```bash
# Sincroniza o branch local com a 'main' antes de enviar o PR
# Isso move seus commits *depois* dos commits mais recentes da main.
git rebase main

```
Use git rebase -i HEAD~N para editar, juntar (squash), ou reordenar os últimos N commits antes de enviar seu Pull Request (PR). Isso transforma uma série de commits "WIP", "Fix typo", "Forgot one file" em um único commit coeso.

 ### 3. Desfazer e Limpar (reset e clean)
Em Cloud, testes de ambiente e ajustes de última hora são comuns. Você precisa saber como desfazer o que fez de forma segura.

| Comando | Função | Exemplo |
| :--- | :--- | :--- |
| git reset --soft |	Desfaz o último commit, mas mantém as mudanças no staging area.|	git reset --soft HEAD~1 |
|git reset --hard |	CUIDADO! Descarta o último commit e todas as mudanças locais na sua cópia. Use apenas se tiver certeza. |	git reset --hard HEAD~1|
|git stash |	Salva temporariamente suas alterações não comitadas para trocar de branch rapidamente.|	git stash push -m "work-in-progress"|
| git clean -fd	| Remove arquivos não rastreados e diretórios vazios. Essencial após testar dependências temporárias. |	git clean -fd|

