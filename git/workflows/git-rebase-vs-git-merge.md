# Git Rebase vs Git Merge — Guia Simples

Este documento compara duas formas comuns de incorporar mudanças de um ramo (branch) em outro no Git: **merge** e **rebase**. Inclui definições, vantagens, desvantagens, e quando usar cada uma.

---

## 1. Definições

### Git Merge

- O que faz: combina historicos de dois branches criando um *commit de merge*.  
- História: mantém ambas as linhas de desenvolvimento visíveis (branching explícito).  
- Analogia: duas correntes de rio se juntando para formar uma maior.  

```bash
git checkout feature/login
git merge main
```

### Git Rebase

- O que faz: reaplica os commits de um branch *feature* “em cima” do branch principal (por exemplo `main`), reescrevendo o histórico.  
- História: linear, sem commit de merge explícito.  
- Analogia: imaginar que você começou seu trabalho já depois das mudanças mais recentes no `main`.  

```bash
git checkout feature/login
git rebase main
```

---

## 2. Vantagens vs Desvantagens

| Estratégia | Vantagens principais | Desvantagens / riscos |
|------------|-----------------------|-------------------------|
| **Merge** | Preserva histórico completo; fácil de ver quando branches divergiram/convergiram; seguro para usar em branches compartilhados | Histórico “mais sujo” ou “mais ramificado”; pode haver muitos commits de merge se vários branches forem integrados frequentemente |
| **Rebase** | Histórico linear; mais limpo; facilita leitura; ideal para preparar um feature branch antes de PR ou MR | Reescreve histórico — não usar em branches públicos / compartilhados; pode gerar problemas se não for bem entendido; conflitos em rebase exigem mais cuidado |

---

## 3. Quando usar cada um

| Situação | Estratégia recomendada |
|----------|-------------------------|
| Branch compartilhado que muitos desenvolvedores usam e já está publicado | **Merge** |
| Branch pessoal / feature branch antes de abrir Pull Request / Merge Request | **Rebase** |
| Quer preservar toda a “árvore” de desenvolvimento e contexto para auditoria ou histórico | Merge |
| Quer um histórico linear mais limpo para facilitar manutenção, review ou rollback | Rebase |

---

## 4. Modelo mental rápido

- **Merge** = “Adicionar um commit que diz: combinamos os trabalhos.”  
- **Rebase** = “Reaplicar meu trabalho como se eu tivesse começado a partir do estado mais recente do outro ramo.”

---

## 5. Possível resposta em entrevistas

> “Merge preserva histórico com um commit de merge, enquanto rebase reescreve commits para que pareça que meu trabalho começou a partir da ponta mais recente do branch alvo. Use merge para branches compartilhados; use rebase para limpar seu histórico local antes de integrar.”

---

## 6. Cuidados e boas práticas

- Nunca faça rebase de branch que já foi compartilhado publicamente sem coordenação.  
- Sempre resolver conflitos com atenção durante rebase.  
- Fazer backup ou usar tags antes de operações destrutivas.  
- Entender bem os fluxos de trabalho da equipe (ex: Git Flow, trunk-based, etc) antes de decidir usar rebase intensivamente.

---

## 7. Exemplos visuais

```
Histórico com Merge:

A---B---C  main
     \     
      D---E---M   feature/login

Histórico com Rebase:

A---B---C---D'---E'   feature/login
```

---

## 8. Resumo

- **Merge**: seguro para uso colaborativo, preserva contexto, visível divergência de branches.  
- **Rebase**: histórico mais limpo, linearidade, ideal para preparar seu trabalho antes de integração, mas exige disciplina.

---

## 9. Ver também / referências

- Artigo original: *Git Rebase vs Git Merge — Simple Guide* (Mohsen Fallahnejad) ([dev.to](https://dev.to/mohsenfallahnjd/git-rebase-vs-git-merge-simple-guide-4pil))  
- Documentação Atlassian sobre rebase e merge (para aprofundar)  
- Práticas de fluxo de trabalho Git na equipe  
