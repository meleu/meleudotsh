# Use o Bash como um programador


- motivação: escreva código para humanos entenderem

- motivos para não usar bash
- sempre use aspas
- sempre use `set -Eeo pipefail`
- sempre use shellcheck
    - leia o wiki quando se deparar com um warning
- use shfmt para padronização de coding-style
    - leia o google shell style guide
- todo código sempre tem que estar dentro de funções (mesmo se for apenas uma única função: `main`).
    - scripts executáveis devem chamar a função principal com `main "$@"`
    - se o script possui funções reutilizáveis, chame com `[[ "$0" == "$BASH_SOURCE" ]] && main "$@"`
- evite variáveis globais. Quando usar, declare com `readonly`.