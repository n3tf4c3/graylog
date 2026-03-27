# Graylog Stack com Docker Compose

Ambiente local com Graylog Open, Graylog Data Node e MongoDB.

## Objetivo

Este projeto sobe um stack completo do Graylog para uso local, com:

- imagens oficiais
- persistencia em volumes nomeados
- validacao de variaveis obrigatorias na inicializacao
- portas bindadas em `127.0.0.1` para reduzir exposicao externa

## Stack atual

- `graylog/graylog:7.0.5-1`
- `graylog/graylog-datanode:7.0.5-1`
- `mongo:7.0.31`

## Arquitetura

- `mongodb`: metadados e configuracoes do Graylog
- `datanode`: armazenamento e indexacao (OpenSearch embarcado)
- `graylog`: API, UI e pipeline de ingestao/consulta

## Estrutura do projeto

```text
.
|-- .env.example
|-- .gitignore
|-- docker-compose.yml
|-- .env
`-- README.md
```

## Pre-requisitos

- Docker Engine 24+ (ou Docker Desktop recente)
- Docker Compose v2
- Recursos recomendados:
  - 4 vCPU
  - 8 GB RAM
  - 20 GB de disco
- Em host Linux/WSL2, configure `vm.max_map_count=262144` para o Data Node

## Variaveis de ambiente (.env)

Obrigatorias:

- `GRAYLOG_PASSWORD_SECRET`
- `GRAYLOG_ROOT_PASSWORD_SHA2`

Opcional:

- `GRAYLOG_HTTP_EXTERNAL_URI` (default: `http://localhost:9000/`)
- `GRAYLOG_ROOT_PASSWORD` apenas como lembrete local, nao e lida pelo compose

Exemplo:

```env
GRAYLOG_PASSWORD_SECRET=troque-por-um-segredo-forte-com-64-ou-mais-caracteres
GRAYLOG_ROOT_PASSWORD_SHA2=hash-sha256-da-senha-do-admin
GRAYLOG_HTTP_EXTERNAL_URI=http://localhost:9000/
```

### Gerar hash SHA256 da senha admin (PowerShell)

```powershell
$pwd = "SuaSenhaForteAqui"
$sha = [System.Security.Cryptography.SHA256]::Create()
try {
  $hashBytes = $sha.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($pwd))
} finally {
  $sha.Dispose()
}
-join ($hashBytes | ForEach-Object { $_.ToString("x2") })
```

Use o resultado em `GRAYLOG_ROOT_PASSWORD_SHA2`.

## Subida do ambiente

1. Validar configuracao:

```bash
docker compose config
```

2. Baixar imagens:

```bash
docker compose pull
```

3. Subir stack:

```bash
docker compose up -d
```

4. Acompanhar logs:

```bash
docker compose logs -f
```

## Primeiro acesso

1. Acesse `http://localhost:9000`.
2. Se aparecer a tela de preflight (primeiro bootstrap com Data Node), pegue a senha inicial em:

```bash
docker compose logs graylog
```

3. Depois da configuracao inicial, faca login com usuario `admin` e a senha em texto claro usada para gerar `GRAYLOG_ROOT_PASSWORD_SHA2`.

## Portas publicadas no host

Todas bindadas em `127.0.0.1`.

- `9000/tcp`: Graylog UI/API
- `5044/tcp`: Beats
- `1514/tcp` e `1514/udp`: Syslog
- `5555/tcp` e `5555/udp`: RAW
- `12201/tcp` e `12201/udp`: GELF
- `13301/tcp` e `13302/tcp`: Forwarder
- `8999/tcp`: Data Node API
- `9200/tcp` e `9300/tcp`: OpenSearch API/cluster

## Persistencia

Volumes nomeados usados pelo compose:

- `mongodb_data`
- `mongodb_config`
- `graylog-datanode`
- `graylog_data`
- `graylog_journal`

## Operacao

Status:

```bash
docker compose ps
```

Reiniciar:

```bash
docker compose restart
```

Parar e remover containers/rede:

```bash
docker compose down
```

## Atualizacao de versoes

1. Atualize as tags no `docker-compose.yml`.
2. Rode:

```bash
docker compose pull
docker compose up -d
```

3. Valide com `docker compose ps` e `docker compose logs -f`.

## Troubleshooting

- Erro de variavel obrigatoria:
  - confira se `.env` esta no mesmo diretorio do `docker-compose.yml`
  - confira se `GRAYLOG_PASSWORD_SECRET` e `GRAYLOG_ROOT_PASSWORD_SHA2` estao preenchidas
- Falha no login `admin`:
  - confirme que a senha em texto claro corresponde ao hash em `GRAYLOG_ROOT_PASSWORD_SHA2`
- Containers nao sobem:
  - rode `docker compose logs -f graylog datanode mongodb`
- Performance ruim:
  - aumente CPU/RAM do host e revise limites de SO/WSL2

## Seguranca para Git

- nao publique `.env` com segredos reais
- use `.env.example` para versionar somente placeholders
- para producao, use secret manager e TLS/HTTPS

## Referencias oficiais

- Graylog docs: https://go2docs.graylog.org/current/home.htm
- Run Graylog in Docker: https://go2docs.graylog.org/current/downloading_and_installing_graylog/docker_installation.htm
- Graylog and Docker Compose: https://go2docs.graylog.org/current/downloading_and_installing_graylog/graylog_and_docker.htm
- Compatibility Matrix: https://go2docs.graylog.org/current/downloading_and_installing_graylog/compatibility_matrix.htm
