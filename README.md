# Classificador de Chamados — Versionamento de Dados e Modelos

Repositório da Atividade Prática 07 de Engenharia de Sistemas Inteligentes (UFC – CK0444), demonstrando o versionamento de datasets, partições e artefatos de modelo com Git e DVC.

## Pré-requisitos

- Git instalado e configurado
- Python 3.10 ou superior (o script `src/train.py` utiliza apenas a biblioteca padrão)
- DVC 3 (`dvc>=3,<4`)

Verifique as ferramentas instaladas:

```bash
git --version
python3 --version
dvc --version
```

No Windows sem o alias `python3`, utilize `python --version`.

## Configuração de Acesso ao Armazenamento DVC

Os arquivos de dados (`data/chamados.csv`, `data/split.csv`) e o artefato de modelo (`models/model.json`) são rastreados pelo DVC. O repositório Git armazena apenas os descritores `.dvc`, os manifestos, os parâmetros, o código-fonte e os relatórios.

O armazenamento remoto padrão configurado em `.dvc/config` chama-se `storage` e aponta para a pasta irmã `../dvc-storage`:

```bash
dvc remote list
```

Caso o armazenamento DVC esteja localizado em outro diretório, ajuste o caminho localmente antes de executar `dvc pull`:

```bash
dvc remote modify --local storage url /caminho/para/dvc-storage
```

## Comandos de Restauração

Para recuperar uma versão exata do dataset e do modelo (por exemplo, `modelo-r1` ou `modelo-r2`) sem precisar treinar novamente:

1. Clone o repositório ao lado da pasta `dvc-storage`:

```bash
git clone https://github.com/lucas-ferre/aula-ml.git aula-ml-restauracao
cd aula-ml-restauracao
```

2. Selecione a tag desejada (`modelo-r1` ou `modelo-r2`):

```bash
git switch --detach modelo-r1
```

3. Baixe os artefatos versionados pelo DVC:

```bash
dvc pull
```

4. Verifique a integridade dos arquivos recuperados comparando seus hashes SHA-256 com o manifesto `reports/modelo.json`:

```bash
python3 - <<'PY'
import hashlib
import json
from pathlib import Path

manifesto = json.loads(Path("reports/modelo.json").read_text(encoding="utf-8"))
for caminho, campo in [
    ("data/chamados.csv", "dataset_sha256"),
    ("data/split.csv", "split_sha256"),
    ("models/model.json", "artifact_sha256"),
]:
    obtido = hashlib.sha256(Path(caminho).read_bytes()).hexdigest()
    if obtido != manifesto[campo]:
        raise SystemExit(f"Hash divergente para {caminho}")
    print(f"OK {caminho} {obtido}")
PY
```

## Estrutura do Projeto

- `data/chamados.csv.dvc` e `data/split.csv.dvc`: descritores DVC do dataset e da partição fixa
- `data/manifesto.json`: metadados e histórico de versão do dataset
- `params.json`: parâmetros de tokenização
- `src/train.py`: script determinístico de treinamento, avaliação e geração de manifestos
- `models/model.json.dvc`: descritor DVC do artefato de modelo treinado
- `reports/metrics.json`: métricas de avaliação na partição de teste fixa
- `reports/modelo.json`: manifesto de rastreabilidade do modelo (commit, hashes, ambiente e execução)
- `reports/comparacao.md`: comparação entre `chamados-r1` e `chamados-r2`, evidência de restauração e decisão
