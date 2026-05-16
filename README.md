# Crypto Wallet Monitor

Sistema modular para coletar, armazenar e expor saldos de tokens (USDT) de carteiras específicas na rede Binance Smart Chain (BSC).

##  Arquitetura
- **Coletor (`collector.py`)**: Script em Python que consome a blockchain BSC via RPC pública e extrai saldos via chamadas de Contrato Inteligente (ERC-20/BEP-20).
- **Banco de Dados (`PostgreSQL`)**: Camada de persistência relacional mapeada via SQLAlchemy ORM.
- **API (`FastAPI`)**: Camada de entrega com endpoints RESTful assíncronos e validação estrita via Pydantic.

## Como Executar

### 1. Inicializar o Banco de Dados (Via Docker)
Se não tiver um banco PostgreSQL rodando localmente, suba um container oficial:
```bash
docker run --name wallet-postgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=wallet_monitor -p 5432:5432 -d postgres
```

### 2. Configurar o Ambiente Python
Crie e ative seu ambiente virtual, em seguida instale as dependências:
```bash
python -m venv venv
source venv/bin/activate  # No Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Crie o arquivo `.env` baseado no `.env.example` e ajuste suas credenciais de acesso se necessário.

### 3. Rodar o Coletor de Dados
Execute o script para buscar os saldos e alimentar a tabela do banco de dados:
```bash
python collector.py
```
*Dica: Você pode agendar este script no Cron (Linux) ou Task Scheduler (Windows) para rodar a cada X minutos.*

### 4. Subir a API HTTP
Inicie o servidor de desenvolvimento da API:
```bash
uvicorn main:app --reload
```

### 5. Acessar a Documentação Interativa
Abra o navegador e acesse a interface Swagger gerada automaticamente:
- **Swagger UI**: [http://127.0.0](http://127.0.0)

##  Erro comum: conexão recusada com PostgreSQL

Se ao executar `collector.py` você receber um erro parecido com:

```
psycopg2.OperationalError: connection to server at "localhost", port 5432 failed: Connection refused
```

Tente as seguintes soluções rápidas:

- **Iniciar o PostgreSQL localmente (Debian/Ubuntu):**
```bash
sudo systemctl start postgresql
sudo systemctl status postgresql
# ou, em algumas distros:
sudo service postgresql start
```

- **Subir um container PostgreSQL (recomendado para dev):**
```bash
docker run --name wallet-postgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=wallet_monitor -p 5432:5432 -d postgres:15
```

- **Ajustar a variável `DATABASE_URL` no arquivo `.env`:** a URL deve apontar para um servidor PostgreSQL acessível.

- **Teste rápido sem PostgreSQL (opcional):** altere temporariamente para SQLite no `.env`:
```
DATABASE_URL=sqlite:///./wallet_monitor.db
```

Depois de garantir que o banco esteja acessível, execute novamente:
```bash
python collector.py
```

### Observação importante sobre o arquivo `.env` (explicação simples)

O arquivo `.env` é onde o projeto guarda configurações que o programa precisa para funcionar — por exemplo, onde está o banco de dados e qual serviço RPC usar para acessar a blockchain.

Para quem não é desenvolvedor: pense no `.env` como uma ficha com as instruções que o aplicativo usa para se conectar a outros serviços. Essas instruções devem ser corretas e reais; se você colar valores que "parecem" gerados automaticamente (textos estranhos, nomes genéricos ou URLs inválidas), o programa não conseguirá se conectar.

O que NÃO fazer:
- Colar valores que pareçam gerados por IA (ex.: strings como "user_generated_by_ai", URLs falsas ou frases genéricas).
- Compartilhar o arquivo `.env` em repositórios públicos — ele pode conter senhas.

O que FAZER (exemplos fáceis de entender):

- Se você tem um banco PostgreSQL no seu computador ou na rede da empresa, use uma linha parecida com esta:

```text
DATABASE_URL=postgresql://user:password@localhost:5432/wallet_monitor
BSC_RPC_URL=https://bsc-dataseed1.binance.org
```

- Se você só quer testar sem instalar nada, use SQLite (útil para testar localmente):

```text
DATABASE_URL=sqlite:///./wallet_monitor.db
BSC_RPC_URL=https://bsc-dataseed1.binance.org
```

Como detectar problema comum:
- Ao rodar `python collector.py`, se aparecer erro com "connection refused" ou "OperationalError" relacionado a PostgreSQL, significa que o endereço/credenciais não estão acessíveis.

Opções simples para resolver sem programar:
1. Peça ao time de infraestrutura as credenciais corretas (usuário, senha, host, porta, nome do banco) e cole no `.env`.
2. Suba um PostgreSQL local com Docker (exemplo — copie e cole no terminal):

```bash
docker run --name wallet-postgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=wallet_monitor -p 5432:5432 -d postgres:15
```

3. Ou altere para SQLite (veja exemplo acima) para testes rápidos — isso cria um arquivo `wallet_monitor.db` na pasta do projeto.

Se precisar, eu posso:
- Aplicar automaticamente a fallback para SQLite (para você não precisar mudar nada manualmente).
- Ajudar a subir o container PostgreSQL passo a passo.

Segurança: nunca comite o `.env` com senhas em repositórios públicos e evite compartilhar credenciais em canais não seguros.

