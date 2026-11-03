# 🚀 Instalação e configuração do projeto

Este documento apresenta as etapas necessárias para configurar o ambiente de desenvolvimento, instalar as dependências e executar a aplicação FastAPI com PostgreSQL e SQLAlchemy.

## 1. Pré-requisitos

Antes de começar, instale as seguintes ferramentas:

- **Python 3.12 ou superior**
- **PostgreSQL**
- **Git**
- **Visual Studio Code** (opcional)

Verifique se o Python e o Git estão instalados:

```bash
python3 --version
git --version
```

Verifique também a instalação do PostgreSQL:

```bash
psql --version
```

## 2. Clonar o repositório

Caso o projeto já esteja no GitHub, clone o repositório:

```bash
git clone URL_DO_REPOSITORIO
cd NOME_DO_PROJETO
```

Substitua `URL_DO_REPOSITORIO` e `NOME_DO_PROJETO` pelos dados reais do projeto.

Se estiver começando do zero, crie a pasta do projeto:

```bash
mkdir meu-projeto
cd meu-projeto
```

## 3. Criar o ambiente virtual

O ambiente virtual mantém as dependências do projeto isoladas de outros projetos Python.

No Linux ou macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

No Windows (PowerShell):

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

Após a ativação, o terminal normalmente exibirá `(.venv)`.

Atualize o instalador de pacotes:

```bash
python -m pip install --upgrade pip
```

## 4. Criar a estrutura de pastas

Na raiz do projeto, execute:

```bash
mkdir -p app/core
mkdir -p app/domain/entities
mkdir -p app/repositories
mkdir -p app/services
mkdir -p app/routers
mkdir -p app/schemas
mkdir -p tests/unit
mkdir -p tests/integration
mkdir -p tests/e2e
```

Crie os arquivos Python e de configuração:

```bash
touch app/__init__.py
touch app/main.py

touch app/core/__init__.py
touch app/core/config.py
touch app/core/database.py
touch app/core/exceptions.py

touch app/domain/__init__.py
touch app/domain/exceptions.py
touch app/domain/entities/__init__.py
touch app/domain/entities/produto.py

touch app/repositories/__init__.py
touch app/repositories/produto_repository.py

touch app/services/__init__.py
touch app/services/produto_service.py

touch app/routers/__init__.py
touch app/routers/produto_router.py

touch app/schemas/__init__.py
touch app/schemas/produto.py

touch tests/__init__.py
touch tests/conftest.py
touch tests/unit/test_produto_service.py
touch tests/integration/test_produto_repository.py
touch tests/e2e/test_produto_router.py

touch requirements.txt
touch .env
touch .env.test
touch .gitignore
touch pytest.ini
```

> Os comandos de criação de pastas acima são para Linux e macOS. No Windows, você pode criar a mesma estrutura pelo VS Code ou pelo explorador de arquivos.

## 5. Configurar o arquivo `requirements.txt`

O arquivo `requirements.txt` contém as bibliotecas necessárias para executar e testar o projeto.

Adicione:

```text
fastapi
uvicorn[standard]
sqlalchemy
psycopg[binary]
pydantic-settings
pytest
httpx
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Depois de validar o ambiente, gere um registro das versões instaladas:

```bash
pip freeze > requirements-lock.txt
```

Esse arquivo pode ser utilizado para reproduzir as versões instaladas no ambiente atual. Para projetos maiores, considere usar um gerenciador de dependências com lockfile, como o Poetry ou o uv.

## 6. Configurar o PostgreSQL

Certifique-se de que o serviço do PostgreSQL esteja funcionando.

No Ubuntu, por exemplo:

```bash
sudo systemctl status postgresql
```

Se necessário, inicie o serviço:

```bash
sudo systemctl start postgresql
```

Acesse o PostgreSQL com um usuário administrador autorizado:

```bash
sudo -u postgres psql
```

Crie o banco de dados da aplicação:

```sql
CREATE DATABASE loja;
```

Crie também um banco exclusivo para os testes de integração:

```sql
CREATE DATABASE loja_test;
```

Saia do terminal do PostgreSQL:

```sql
\q
```

Configure um usuário e uma senha apropriados para a aplicação, com permissões limitadas aos bancos utilizados. Evite utilizar o superusuário `postgres` na aplicação em produção.

## 7. Configurar as variáveis de ambiente

No arquivo `.env`, adicione a URL de conexão do banco principal:

```env
DATABASE_URL=postgresql+psycopg://usuario:senha@localhost:5432/loja
```

No arquivo `.env.test`, configure a conexão exclusiva para testes:

```env
DATABASE_URL=postgresql+psycopg://usuario:senha@localhost:5432/loja_test
```

Substitua `usuario` e `senha` pelas credenciais configuradas no PostgreSQL.

**Importante:** o código de configuração precisa carregar explicitamente o arquivo `.env.test` ao executar os testes. Apenas criar o arquivo não altera automaticamente a conexão utilizada pela aplicação.

Adicione ao `.gitignore`:

```gitignore
# Ambiente virtual
.venv/
venv/

# Variáveis de ambiente e segredos
.env
.env.test

# Cache Python
__pycache__/
*.py[cod]
.pytest_cache/

# Ambientes e ferramentas
.vscode/
.idea/

# Relatórios de cobertura
htmlcov/
.coverage
```

Nunca publique credenciais reais no repositório.

## 8. Configurar o pytest

No arquivo `pytest.ini`, adicione:

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

Isso permite que o pytest localize os testes automaticamente.

## 9. Implementar os arquivos da aplicação

Preencha os arquivos criados conforme a arquitetura do projeto:

- `app/core/config.py`: configurações e variáveis de ambiente.
- `app/core/database.py`: engine, pool de conexões e sessões SQLAlchemy.
- `app/core/exceptions.py`: tratamento centralizado de erros.
- `app/domain/entities/`: entidades do domínio.
- `app/repositories/`: consultas e persistência de dados.
- `app/services/`: regras de negócio.
- `app/routers/`: endpoints HTTP do FastAPI.
- `app/schemas/`: validação de entrada e saída com Pydantic.
- `app/main.py`: inicialização da aplicação.

A separação de responsabilidades deve ser mantida:

```text
Router → Service → Repository → PostgreSQL
                 ↑
               Domain
```

O Router recebe a requisição, o Service executa as regras de negócio, o Repository acessa os dados e o Domain representa os conceitos do negócio.

## 10. Executar a aplicação

Com o ambiente virtual ativado, execute na raiz do projeto:

```bash
uvicorn app.main:app --reload
```

Por padrão, a aplicação estará disponível em:

- API: http://127.0.0.1:8000
- Documentação Swagger: http://127.0.0.1:8000/docs
- Documentação ReDoc: http://127.0.0.1:8000/redoc

O parâmetro `--reload` deve ser utilizado apenas durante o desenvolvimento.

## 11. Executar os testes

Para executar todos os testes:

```bash
pytest
```

Para executar somente os testes unitários:

```bash
pytest tests/unit/
```

Para executar os testes de integração:

```bash
pytest tests/integration/
```

Para executar os testes dos endpoints:

```bash
pytest tests/e2e/
```

Antes de executar os testes de integração e os testes ponta a ponta, configure o ambiente de testes para utilizar o banco `loja_test`, nunca o banco principal.

## 12. Verificar a instalação

Após iniciar a aplicação, acesse:

```text
http://127.0.0.1:8000/docs
```

Se a documentação interativa carregar e os endpoints estiverem registrados, a aplicação estará acessível.

Antes de considerar a instalação concluída, confirme que:

- [ ] O ambiente virtual está ativo.
- [ ] Todas as dependências foram instaladas.
- [ ] O PostgreSQL está funcionando.
- [ ] A variável `DATABASE_URL` está configurada.
- [ ] As tabelas ou migrações do banco foram aplicadas.
- [ ] A API inicia sem erros.
- [ ] Os testes são executados com sucesso.

## 13. Encerrar o ambiente virtual

Quando terminar de trabalhar, execute:

```bash
deactivate
```

Para voltar a trabalhar no projeto, entre na pasta e ative novamente o ambiente virtual.

---

**Pronto!** Com essas etapas, o ambiente estará preparado para desenvolver, executar e testar a aplicação FastAPI com PostgreSQL e SQLAlchemy, seguindo uma arquitetura organizada em camadas.