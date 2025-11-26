# Worktrees para Aplicações com Serviços

> **Nota:** Esta é uma versão traduzida para o Português Brasileiro do documento original `apps.md`.

Este guia fornece passos de configuração adicionais ao trabalhar com worktrees para projetos que executam serviços (apps web com servidores HTTP, serviços docker-compose, etc.).

## Quando Usar Este Guia

Use estas instruções após seguir a configuração principal de worktree em [SKILL-PTBR.md](SKILL-PTBR.md) (ou [SKILL.md](SKILL.md)) **apenas se**:
- O projeto possui arquivos `.env*` no diretório principal
- O projeto executa serviços como servidores web, bancos de dados ou docker-compose

Se o projeto for uma biblioteca ou ferramenta CLI sem um arquivo `.env`, pule estas instruções completamente.

## Copiar Arquivos de Ambiente

Antes de configurar portas e iniciar serviços, copie quaisquer arquivos `.env*` do diretório de trabalho principal:

```bash
cp ../../.env* .
```

Isso garante que a worktree tenha a configuração de ambiente necessária para executar a aplicação. Arquivos de ambiente são tipicamente ignorados pelo git (gitignored) e não estariam disponíveis na nova worktree de outra forma.

## Alocação de Portas

Para evitar conflitos ao executar múltiplas instâncias da aplicação em worktrees paralelas, cada worktree precisa do seu próprio conjunto de portas.

### Portas Necessárias

Aloque 4 portas disponíveis para os seguintes serviços:
1. `SERVER_ADDRESS` - A porta principal do servidor HTTP (também usada para `BASE_URL`)
2. `MINIO_PORT` - Endpoint da API S3 do MinIO (também usado para `AWS_ENDPOINT_URL`)
3. `MINIO_CONSOLE_PORT` - Console web do MinIO
4. `MINIO_TEST_PORT` - Instância de teste do MinIO

### Varredura de Portas

Use o script fornecido para encontrar 4 portas disponíveis começando de **9090**:

```bash
# Execute o script de alocação de portas da skill worktrees
# Nota: Este script está localizado no diretório da skill worktrees, não no repositório do projeto
ports=($(bash scripts/allocate-ports.sh))

# Atribua portas às variáveis (usando indexação baseada em 1 para compatibilidade com zsh)
SERVER_PORT=${ports[1]}
MINIO_PORT=${ports[2]}
MINIO_CONSOLE_PORT=${ports[3]}
MINIO_TEST_PORT=${ports[4]}
```

**Importante:**
- As portas não precisam ser consecutivas - o script encontra a próxima porta disponível até ter 4
- O array usa indexação baseada em 1 (estilo zsh) para compatibilidade
- O diretório `scripts/` é parte da skill worktrees, não do seu repositório de projeto

### Atualizar Arquivo .env

Uma vez que as portas estejam alocadas, atualize o arquivo `.env` com os novos valores:

```bash
# Atualizar SERVER_ADDRESS (note o prefixo :)
sed -i '' "s|^SERVER_ADDRESS=.*|SERVER_ADDRESS=:${SERVER_PORT}|" .env

# Atualizar BASE_URL
sed -i '' "s|^BASE_URL=.*|BASE_URL=http://localhost:${SERVER_PORT}|" .env

# Atualizar AWS_ENDPOINT_URL para corresponder a MINIO_PORT
sed -i '' "s|^AWS_ENDPOINT_URL=.*|AWS_ENDPOINT_URL=http://localhost:${MINIO_PORT}|" .env

# Atualizar portas do MinIO
sed -i '' "s|^MINIO_PORT=.*|MINIO_PORT=${MINIO_PORT}|" .env
sed -i '' "s|^MINIO_CONSOLE_PORT=.*|MINIO_CONSOLE_PORT=${MINIO_CONSOLE_PORT}|" .env
sed -i '' "s|^MINIO_TEST_PORT=.*|MINIO_TEST_PORT=${MINIO_TEST_PORT}|" .env
```

## Iniciando Serviços

Após atualizar o arquivo `.env`, inicie os serviços:

### 1. Iniciar docker-compose (se docker-compose.yml existir)

```bash
docker compose up -d
```

Isso inicia os serviços MinIO em modo "detached" (em segundo plano) nas portas alocadas. O Docker Compose lerá automaticamente os valores das portas do arquivo `.env`.

### 2. Baixar Tailwind CSS (se necessário)

```bash
make tailwindcss
```

Isso baixa o binário do Tailwind CSS se ele não existir na worktree.

### 3. Iniciar monitoramento do Tailwind CSS (se tailwindcss existir)

```bash
./tailwindcss -i tailwind.css -o public/styles/app.css --watch &
```

Isso inicia a compilação do Tailwind CSS em modo de observação (watch mode) para reconstrução automática do CSS.

### 4. Iniciar watch.sh (se existir)

```bash
./watch.sh &
```

Isso inicia a aplicação com auto-reload do Air em segundo plano. O script `watch.sh` lerá `SERVER_ADDRESS` do arquivo `.env` e iniciará o servidor HTTP na porta alocada.

### 5. Informar o Usuário

Reporte ao usuário com:
- O caminho da worktree (`.worktrees/<nome-da-branch>`)
- As portas alocadas:
  - URL da App: `http://localhost:${SERVER_PORT}` (de BASE_URL)
  - MinIO S3: `http://localhost:${MINIO_PORT}`
  - Console MinIO: `http://localhost:${MINIO_CONSOLE_PORT}`
  - Teste MinIO: `http://localhost:${MINIO_TEST_PORT}`
- Confirmação de que os serviços estão rodando

## Parando Serviços

Ao parar o trabalho em uma worktree ou fazer a limpeza, use o script de encerramento:

```bash
cd .worktrees/<nome-da-branch>
bash scripts/shutdown-services.sh
```

**Nota:** O diretório `scripts/` é parte da skill worktrees, não do seu repositório de projeto.

Este script irá:
1. Parar e remover serviços do docker compose (se docker-compose.yml existir)
2. Parar o processo do servidor da aplicação pela porta (se .env existir)

## Rastreamento de Portas

As portas são armazenadas **apenas** no arquivo `.env` de cada worktree - não há um registro central. Quando você precisar descobrir quais portas uma worktree está usando, leia seu arquivo `.env`:

```bash
grep -E "^(SERVER_ADDRESS|MINIO_PORT|MINIO_CONSOLE_PORT|MINIO_TEST_PORT)=" .worktrees/<nome-da-branch>/.env
```

## Exemplo de Fluxo de Trabalho Completo

```bash
# 1. Criar worktree (do SKILL.md principal)
git worktree add .worktrees/feature-auth -b feature-auth
cd .worktrees/feature-auth

# 2. Copiar arquivos .env
cp ../../.env* .

# 3. Alocar portas usando o script
ports=($(bash scripts/allocate-ports.sh))
SERVER_PORT=${ports[1]}
MINIO_PORT=${ports[2]}
MINIO_CONSOLE_PORT=${ports[3]}
MINIO_TEST_PORT=${ports[4]}
# Exemplo de alocação: 9090, 9091, 9092, 9093

# 4. Atualizar .env com as portas alocadas
sed -i '' "s|^SERVER_ADDRESS=.*|SERVER_ADDRESS=:${SERVER_PORT}|" .env
sed -i '' "s|^BASE_URL=.*|BASE_URL=http://localhost:${SERVER_PORT}|" .env
sed -i '' "s|^AWS_ENDPOINT_URL=.*|AWS_ENDPOINT_URL=http://localhost:${MINIO_PORT}|" .env
sed -i '' "s|^MINIO_PORT=.*|MINIO_PORT=${MINIO_PORT}|" .env
sed -i '' "s|^MINIO_CONSOLE_PORT=.*|MINIO_CONSOLE_PORT=${MINIO_CONSOLE_PORT}|" .env
sed -i '' "s|^MINIO_TEST_PORT=.*|MINIO_TEST_PORT=${MINIO_TEST_PORT}|" .env

# 5. Iniciar serviços
docker compose up -d
make tailwindcss
./tailwindcss -i tailwind.css -o public/styles/app.css --watch &
./watch.sh &

# 6. Reportar ao usuário
echo "Worktree criada em .worktrees/feature-auth"
echo "App rodando em http://localhost:${SERVER_PORT}"
echo "MinIO S3 em http://localhost:${MINIO_PORT}"
echo "MinIO Console em http://localhost:${MINIO_CONSOLE_PORT}"
echo "Serviços iniciados com sucesso"

# Mais tarde: Parar serviços
cd .worktrees/feature-auth
bash scripts/shutdown-services.sh
cd ../..
```
