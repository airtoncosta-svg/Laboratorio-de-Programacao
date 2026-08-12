---
name: pr-code-review
description: Revisa as mudanças de um branch de código (JS/CSS/HTML, incluindo repositórios HTML estáticos sem build tooling) comparando com a main, e gera um comentário pronto para colar em um Pull Request. Use esta skill sempre que o usuário disser que vai abrir um PR, pedir para revisar seu próprio branch antes de abrir o PR, pedir para revisar o PR de um colega, mencionar "code review", "revisão de código", "revisar esse branch/PR" ou pedir feedback sobre um diff. Dispare mesmo que o usuário não peça explicitamente um "review formal" — qualquer pedido para olhar/checar/avaliar mudanças de código antes de subir ou mesclar justifica usar esta skill.
---

# PR Code Review

Revisa apenas o que mudou em um branch (comparado com `main`), roda as
verificações automáticas disponíveis no repositório — sejam elas baseadas em
npm ou, na ausência de qualquer manifest de build, validadores de HTML
disponíveis no ambiente — e produz um comentário pronto para colar em um
Pull Request no GitHub/GitLab.

## Quando usar

- Antes do usuário abrir um PR (revisão do próprio branch).
- Quando o usuário for revisar o PR de um colega.

## Passo 1 — Descobrir o que mudou

Rode, na raiz do repositório:

```bash
git fetch origin main
git diff origin/main...HEAD --stat
git diff origin/main...HEAD
```

Se o repositório não tiver remoto configurado ou a branch principal tiver
outro nome (`master`, `trunk`), detecte com `git branch -r` ou pergunte ao
usuário. Revise **somente** as linhas alteradas/adicionadas no diff — não
comente sobre código pré-existente que o diff não tocou, mesmo que pareça
problemático.

## Passo 2 — Rodar as verificações automáticas

O repositório é JS/CSS/HTML — mas nem todo repositório HTML tem tooling de
build (alguns são HTML estático, sem `package.json`). Não assuma comandos
fixos — detecte automaticamente o que existe:

1. Leia o `package.json` na raiz (e em subpastas, se for um monorepo).
2. **Se houver `package.json`**, rode os scripts relevantes que existirem em
   `"scripts"`, tipicamente:
   - `test` (ex: `npm test` / `npm run test`)
   - `lint` (ex: `npm run lint` — pode ser ESLint, Stylelint, etc.)
   - `build` (ex: `npm run build`)
   - `typecheck` / `tsc` se existir
3. **Se NÃO houver `package.json`** (HTML puro/estático, sem build
   tooling), procure validadores de HTML disponíveis no ambiente, por
   exemplo `tidy -q -e <arquivo>` (HTML Tidy), e rode-os apenas nos
   arquivos `.html` alterados no diff. Se nenhum validador estiver
   disponível, não instale nada por conta própria — apenas registre que
   nenhuma verificação automática estava disponível no repo/ambiente.
4. Em qualquer um dos dois casos, se algum comando/script não existir ou
   não estiver disponível, não invente o comando — apenas registre que
   aquela verificação não estava disponível.
5. Rode os testes/validações existentes normalmente. **Não é necessário**
   escrever testes novos para a revisão — a menos que o usuário peça.

Guarde a saída de cada comando (passou/falhou, e o trecho relevante do erro)
para usar no Passo 4.

## Passo 3 — Analisar o diff

Para cada problema encontrado no código alterado, é obrigatório informar:

- **Arquivo e linha** (ex: `src/utils/parse.js:42`).
- **Como reproduzir**: um input, comando, ou passo concreto que expõe o
  problema (ex: "chamar `parseDate('')` retorna `NaN` sem tratamento" ou
  "rodar `npm test` falha em `parse.test.js:18`").

Classifique cada achado em uma de duas categorias, **sem misturar as duas na
mesma seção**:

- **Quebra o programa**: erro de build, teste que falha, erro de lint que é
  regra de erro (não de estilo), exceção não tratada, bug de lógica
  comprovável com um exemplo concreto.
- **Preferência / sugestão**: qualquer coisa que não quebra nada, mas que
  vale considerar (nomenclatura, abordagem alternativa, etc.) — só inclua se
  vier acompanhada do problema real que resolve (ver regras abaixo).

### O que NÃO fazer

- Não reclamar de formatação (espaçamento, indentação, ponto e vírgula,
  aspas) — isso é trabalho de formatter/linter automático, não de revisão
  humana.
- Não apontar um problema sem dizer onde ele está (arquivo + linha).
- Não sugerir refatoração sem explicar qual problema concreto ela resolve.
  "Isso poderia ser mais limpo" não é válido; "isso duplica a lógica de
  `validateUser` em `auth.js:10`, o que já causou o bug X" é válido.

## Passo 4 — Montar o comentário do PR

Gere o resultado em Markdown, pronto para colar em um PR. Estrutura:

```markdown
## Revisão automática

### 🔴 Quebra o programa
- **`arquivo:linha`** — descrição do problema. Como reproduzir: ...
(ou "Nenhum problema encontrado aqui.")

### 🟡 Preferência / sugestão
- **`arquivo:linha`** — sugestão. Resolve: <problema concreto>.
(ou "Nenhuma sugestão.")

### ✅ O que foi verificado
- Rodado: `npm test` (passou / falhou em X)
- Rodado: `npm run lint` (passou / N avisos)
- Rodado: `npm run build` (não existe no repo — não verificado)
- Apenas lido (não executado): <partes do código analisadas só por leitura,
  ex: lógica condicional em `checkout.js`>
```

A seção "O que foi verificado" é obrigatória e deve deixar claro,
separadamente, o que foi confirmado **rodando comando** e o que foi apenas
**lido**. Nunca apresente uma leitura de código como se fosse uma verificação
executada.

Entregue o resultado direto no chat, em um bloco de código Markdown, para o
usuário copiar e colar no PR.
