# `.github` — configuração compartilhada

[![CI](https://github.com/gicc-ufrn/.github/actions/workflows/ci.yml/badge.svg)](https://github.com/gicc-ufrn/.github/actions/workflows/ci.yml)

Repositório de configuração da organização. Não tem código de produto: tem o que
**todos os outros repositórios estendem**, para que a mesma regra não exista em
sete cópias que divergem.

| Caminho | O que é | Quem usa |
|---|---|---|
| `profile/README.md` | Perfil público da organização | GitHub, automático |
| `renovate.json` | Preset do Renovate | os sete repositórios, via `extends` |
| `.github/workflows/mise-ci.yml` | Workflow reutilizável | os repositórios, via `uses:` |

## Como estender

**Renovate.** No repositório, o arquivo inteiro é:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>gicc-ufrn/.github:renovate"]
}
```

**CI.** O workflow do repositório chama, em vez de copiar:

```yaml
jobs:
  ci:
    uses: gicc-ufrn/.github/.github/workflows/mise-ci.yml@main
    with:
      runner: macos-14              # opcional
      antes: rustup target add ...  # opcional
      tarefas: ci                   # tasks do mise, em ordem
```

Repositório privado pode estender preset e chamar workflow **públicos** — o
contrário não vale, e é a direção certa: o privado depende do público, nunca o
inverso.

## O contrato

O workflow reutilizável não sabe a linguagem de nada. Ele instala o `mise` e roda
as tasks que você pedir. O que `ci` significa é decisão do `mise.toml` local:

```
mise run lint      análise estática
mise run test      testes
mise run checks    integridade (fronteira arquitetural, licença, drift)
mise run ci        a soma dos três
```

No BARG, `checks` é a fronteira biblioteca × produto e a lista de licenças
permitidas. No vocabulário, é a `FF-04`. No guarda-chuva, é o backlog em dia.
Mesmo nome, significado local — que é `R66` (*"o formato é o padrão; a
implementação não"*) aplicado ao ferramental.

## Renovate: por que um PR por semana, e não cinco

O Renovate roda na infraestrutura da Mend e **não consome minutos de Actions**. O que consome
é a CI disparada por cada PR que ele abre — logo, a alavanca de custo é o **número de PRs**,
não a frequência com que ele verifica. Reduzir a frequência sem agrupar não economiza nada:
seriam os mesmos cinco PRs, todos na segunda.

Por isso o preset agrupa patch e minor num PR único, semanal, por repositório. GitHub Actions
vai a mensal — muda pouco e quase nunca quebra.

**A exceção é correção de segurança**, que fica em `at any time`. CVE conhecida não espera
segunda-feira, e é o único caso em que o custo não é o critério.

O `drumhud` tem cadência própria, mensal, por motivo mensurável: é privado (consome cota) e
roda um job em macOS, onde o minuto conta dez vezes mais. Semanal custaria ~320
min-equivalente/mês; mensal, ~80. Os repositórios públicos têm minutos ilimitados e não
precisam de nada disso.

### Duas coisas que o Renovate recusa, e que eu descobri errando

**`"//"` não é chave de comentário.** O `renovate-config-validator` a rejeita com
`Invalid configuration option: //`. A chave para documentar é **`description`** — que aceita
string ou lista de strings, e ainda por cima aparece no corpo do PR.

**O validator precisa rodar sem o nome do arquivo.** Passá-lo como argumento faz ele tratar
o conteúdo como config **global** (a do próprio bot), que tem outro schema. Sem argumento ele
encontra o `renovate.json` sozinho e valida como config de repositório.
