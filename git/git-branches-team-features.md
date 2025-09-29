# Git Branches: como equipes desenvolvem features sem quebrar código alheio

## Visão geral

Trabalhar em equipe num repositório compartilhado exige disciplina para manter a estabilidade da **branch principal** (geralmente chamada `main`, `master` ou `production`).  
O uso estratégico de **branches dedicadas** permite que cada desenvolvedor ou grupo trabalhe isoladamente, reduzindo conflitos e regressões.

Esse artigo adapta conceitos do post *“Git Branches: How Teams Build Features Without Breaking Each Other’s Code”* da Swathi Macha. ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))

---

## Conceitos-chave

- **Branch**: linha paralela de desenvolvimento — você pode criar uma branch nova para experimentar ou desenvolver uma feature sem alterar a branch “estável”. ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))  
- **HEAD**: aponta para o commit atual no qual você está operando; ao trocar de branch, o HEAD muda. ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))  
- **Merge**: operação de incorporar mudanças de uma branch em outra  
  - *Fast-forward merge*: quando não há commits divergentes na branch de destino, o ponteiro simplesmente é movido para frente. ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))  
  - *Merge com commit explícito (no‑fast-forward)*: quando há divergência, é criado um novo commit que une os históricos. ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))  

---

## Estratégias típicas de branches em times

Aqui estão padrões comuns de ramificação (branching) usados por equipes maduras:

| Nome da branch | Propósito | Convenções comuns |
|----------------|-----------|--------------------|
| `main` / `master` | Código estável, pronto para produção | Só recebe merges via pull request / merge review |
| `feature/<nome>` | Desenvolvimento de nova funcionalidade | Curta duração, deve ser integrada cedo |
| `bugfix/<nome>` | Correção de defeitos urgentes | Pode ser branch de `main` ou de release próxima |
| `release/<versão>` | Preparação da próxima entrega | Permite ajustes finais, testes e validação |
| `hotfix/<nome>` | Correção crítica em produção | Criada a partir de `main`, mesclada de volta para `main` e `develop` (ou branches de release) |

---

## Fluxo de trabalho recomendado

1. **Criar a branch** a partir da base estável (ex: `main`):
   ```bash
   git checkout -b feature/minha-nova-funcionalidade
   ```
2. Desenvolver em isolamento, com commits frequentes.
3. **Rebase ou merge de sincronização** (opcional): incorporar mudanças recentes da base estável para evitar divergências:
   ```bash
   git fetch origin
   git rebase origin/main
   ```
   ou
   ```bash
   git merge origin/main
   ```
4. **Abrir Pull Request / Merge Request** para mesclar a branch de feature na branch de destino.
5. Executar revisões de código, testes automatizados, integração contínua (CI).
6. **Mesclar (merge)** com política acordada (fast-forward ou não) e deletar a branch de feature local/remota após o merge.

---

## Benefícios desse approach

- Permite paralelismo: vários devs trabalhando ao mesmo tempo em diferentes áreas.
- Isola riscos: mudanças em desenvolvimento não afetam imediatamente a branch principal.
- Históricos mais claros: merges explícitos ajudam a rastrear quando e por quem algo foi integrado.
- Facilita revisões de código e testes em branches isoladas antes da integração final.

---

## Pontos de atenção / desafios

- Branches longas demais tendem a gerar muitos conflitos de merge.
- Rebase de branches compartilhadas exige cuidado para evitar reescrita de histórico público.
- Política de mesclagem (fast-forward ou não) deve ser bem definida no time.
- Automatização de testes, validações e CI/CT é crucial para garantir qualidade antes do merge.

---

## Referências & leitura complementar

- *Git Branches: How Teams Build Features Without Breaking Each Other’s Code* — artigo-base ([dev.to](https://dev.to/swathi_macha/git-branches-how-teams-build-features-without-breaking-each-others-code-3364))  
- Documentação oficial do Git sobre branching e merging  
- Estratégias de branching como Git Flow, Trunk Based Development, GitHub Flow  
