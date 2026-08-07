# `.github` — configuração compartilhada

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
