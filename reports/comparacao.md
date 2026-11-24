# Comparação de Modelos: `chamados-r1` vs. `chamados-r2`

## Alteração Realizada

Na ramificação `correcao-rotulo`, foi corrigido o rótulo do chamado `006` (`bloqueio de acesso`) no conjunto de treino em `data/chamados.csv`, alterando sua classe de `comum` para `urgente`, em conformidade com a regra de rotulagem definida em `data/manifesto.json`.

- **Dataset `v1` (`dados-v1`) → `v2` (`dados-v2`)**: apenas a linha do ID `006` foi modificada; o manifesto foi atualizado para `dataset_id: "chamados-v2"`.
- **Partição fixa (`data/split.csv`)**: mantida inalterada entre as duas rodadas (`git diff dados-v1 dados-v2 -- data/split.csv.dvc` retornou vazio e `split_sha256` permaneceu idêntico).
- **Código e parâmetros**: `src/train.py` e `params.json` (`{"tokenizador": "primeira_palavra_minuscula"}`) permaneceram inalterados.

## Tabela de Métricas

Avaliação realizada sob a mesma partição de teste fixa (4 exemplos: 2 `comum` e 2 `urgente`).

| Versão | Tag | Dataset | Acurácia | Recall (`comum`, suporte=2) | Recall (`urgente`, suporte=2) |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `chamados-r1` | `modelo-r1` | `chamados-v1` | `0.75` (3/4) | `1.00` (2/2) | `0.50` (1/2) |
| `chamados-r2` | `modelo-r2` | `chamados-v2` | `1.00` (4/4) | `1.00` (2/2) | `1.00` (2/2) |

### Rastreabilidade dos Hashes (SHA-256)

| Artefato / Campo | `chamados-r1` (`modelo-r1`) | `chamados-r2` (`modelo-r2`) | Status |
| :--- | :--- | :--- | :--- |
| `data/split.csv` (`split_sha256`) | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` | Idêntico |
| `data/chamados.csv` (`dataset_sha256`) | `bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac` | `4ca2e29cb930a9161c2a851b3c71f5529dbb812db607ee4a5ea95d2223baa14d` | Atualizado |
| `models/model.json` (`artifact_sha256`) | `72a3a67936139a3473838a6dcfa7aae1acb4c23467be1ef1056d51d485b94840` | `7f9d94ca8dd86ccc8ad90b0839613c61a33e2168bd23bf083a4bbe1375f908ac` | Atualizado |
| Commit de entrada (`git_commit`) | `f62b4aa755b41afae2a60182884cfe675ff91ac2` | `60caa4956571e8ffb2ce3a83cfe07614e10794b9` | Atualizado |

## Evidência de Restauração em Clone Limpo (`modelo-r1`)

A versão original `modelo-r1` foi restaurada em um clone limpo (`aula-ml-restauracao`) com `git switch --detach modelo-r1` e `dvc pull` (3 arquivos recuperados sem novo treinamento). Todos os hashes conferiram com `reports/modelo.json`:

- **Commit selecionado (`git rev-parse HEAD`)**: `eb76f18543fd977d4269ee1921303026874f9886`
- `OK data/chamados.csv bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac`
- `OK data/split.csv 2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981`
- `OK models/model.json 72a3a67936139a3473838a6dcfa7aae1acb4c23467be1ef1056d51d485b94840`

## Decisão

**Decisão: ACEITAR `chamados-r2` (`modelo-r2`) para este exercício.**

- **Critério adotado**: melhorar o recall de chamados `urgente` sem reduzir o recall de chamados `comum`, utilizando a mesma partição de teste fixa.
- **Justificativa**: o modelo `chamados-r2` elevou o recall da classe `urgente` de `0.50` para `1.00` (e a acurácia global de `0.75` para `1.00`) mantendo o recall de `comum` em `1.00`, com integridade comprovada da partição de teste (`split_sha256` idêntico). Observa-se que o conjunto de teste sintético possui apenas quatro exemplos, servindo para demonstrar o efeito da correção do rótulo e o fluxo completo de rastreabilidade neste exercício.
