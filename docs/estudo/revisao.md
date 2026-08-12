# Relatório de Estudo e Revisão de Código

## 1. Ferramenta e Modelo Utilizados
- **Ferramenta:** Claude (Anthropic)
- **Modelo:** Claude 5 Sonnet
- **Skill:** "pr-code-review" -> Usada para revisar o "validador/index.html"


## 2. Pergunta Não Pensada pela Equipe
Durante o processo de execução da skill, a ferramenta solicitou o histórico do repositório Git (acesso à pasta `.git` ou comparação de diff entre branches) 
para rodar o comando `git diff origin/main...HEAD`. Não tínhamos considerado essa dependência de repositório ao enviar apenas o arquivo `validador/index.html` no chat.

## 3. Achados Relevantes da Revisão
A skill de revisão executou testes via Node.js no algoritmo de validação de CPF e na máscara de digitação, confirmando o funcionamento correto dos testes.

Como ponto de melhoria, a skill identificou um comportamento no cursor (`validador/index.html:120-129`): 
ao editar um dígito no meio do CPF, o listener de `input` reatribui o valor do campo sem preservar a posição do cursor (`selectionStart`/`selectionEnd`), fazendo o cursor saltar para o final a cada caractere digitado.

- ## 4. Saída completa do claude:
Não encontrei um repositório Git nesta pasta (nem histórico, nem origin/main) — parece ser um arquivo único enviado diretamente, sem repo. Então revisei o validador/index.html por completo, como se fosse tudo conteúdo novo em um PR.

Não há package.json no ambiente, e não encontrei tidy ou outro validador de HTML instalado. Por isso, extraí a lógica JS (máscara de input e algoritmo de validação de CPF) e rodei-a com node contra casos de teste conhecidos, para confirmar que o comportamento realmente funciona (não apenas por leitura).

**MARKDOWN CLAUDE**
## Revisão automática

### 🔴 Quebra o programa
Nenhum problema encontrado aqui. A máscara de digitação (`index.html:120-130`) e o algoritmo de validação de CPF (`index.html:158-193`) foram executados com Node.js contra casos de teste (CPFs válidos conhecidos, CPFs com dígitos repetidos, CPFs com dígito verificador incorreto e entradas de tamanho inválido) e todos os resultados bateram com o esperado.

### 🟡 Preferência / sugestão
- **`validador/index.html:120-129`** — o cursor salta para o final do campo a cada tecla digitada quando o usuário edita no meio do valor. Resolve: o listener de `input` reatribui `campoCpf.value` inteiro (linha 129) sem restaurar `selectionStart`/`selectionEnd`, então qualquer edição que não seja no final do CPF força o cursor a "pular" para o fim a cada caractere digitado. Reproduzir: digitar um CPF completo, posicionar o cursor entre o 2º e o 3º dígito e digitar um número — o cursor não permanece na posição editada.

### ✅ O que foi verificado
- Rodado: `node --check` no trecho de validação de CPF extraído (sintaxe OK).
- Rodado: script Node com 6 casos de teste no `validarCpf` (2 CPFs válidos conhecidos, 2 com dígitos repetidos, 1 com dígito verificador errado, 1 com tamanho inválido) — todos passaram.
- Rodado: script Node com 9 casos de teste na lógica de máscara de digitação (entradas parciais de 1 a 15 dígitos) — formatação e truncamento em 11 dígitos corretos em todos os casos.
- Não encontrado: `package.json` (não há `npm test`/`lint`/`build` a rodar).
- Não encontrado: validador de HTML (`tidy` ou similar) instalado no ambiente — não verificado.
- Apenas lido (não executado): marcação HTML, CSS e associação de `label`/`aria-live` no restante do arquivo.
