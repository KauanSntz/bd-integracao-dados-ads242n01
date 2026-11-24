# AGENTS.md — Integração de Dados (ADS242N01)

> Leia este arquivo **e** `PROGRESSO.md` antes de qualquer ação. O `PROGRESSO.md` é o
> estado vivo; este arquivo é o contrato estável (contexto, regras, checklist).
> Última revisão deste arquivo: 24/11/2026.

## 1. Contexto da atividade

Disciplina: Banco de Dados (ADS242N01). Trabalho individual ou em dupla, entrega 24/11/2026.
Repositório base: <https://github.com/agnpreto/bd-integracao-dados-ads242n01>
Clone local usado: `C:\Users\kauan\OneDrive\Documentos\bd-integracao-dados`

Cenário: implantar o banco de uma rede de lojas a partir de uma planilha enviada pelo
cliente. O material **contém erros intencionais** no DDL, nos dados e no script de
importação. Encontrar, explicar e corrigir faz parte da avaliação.

Arquivos entregues pelo cliente:

| Arquivo | Papel | Pode alterar? |
| --- | --- | --- |
| `dados/base_teste.xlsx` | 5 abas: `venda`, `pedido_compra`, `entradas_mercadoria`, `produtos_filial`, `fornecedor` | **NÃO** |
| `docs/Atividade_Integracao_de_Dados.pdf` | Enunciado completo | Não |
| `sql/01_ddl_enunciado.sql` | DDL original com erros | **NÃO** |
| `scripts/importar.py` | Script do estagiário, com erros | Sim (corrigir) |
| `README.md` | Estrutura de entrega e regras do PR | Só a tabela "Equipe" |

Regra de negócio central: uma entrada de mercadoria se liga ao seu pedido de compra
pelo campo `ordem_compra`.

## 2. Estrutura de arquivos esperada (README)

```
dados/base_teste.xlsx               planilha do cliente (não alterar)
docs/Atividade_Integracao_de_Dados.pdf
docs/parte1_importacao.md           Etapa 1: documentação da importação
docs/parte4_validacao.md            Etapa 3: roteiro de validação
scripts/importar.py                 script do estagiário (corrigir aqui)
sql/01_ddl_enunciado.sql            DDL original, com erros (não alterar)
sql/01_ddl_postgres.sql             DDL corrigido (PostgreSQL)
sql/01_ddl_mysql.sql                DDL corrigido (MySQL 8+)
sql/02_consultas.sql                Parte 2
sql/03_transformacoes.sql           Parte 3 (inclui a trigger)
sql/04_validacao.sql                Parte 4
sql/validar.sql                     conferência da importação (contagens, órfãos, duplicidades)
AGENTS.md                           este arquivo
PROGRESSO.md                        estado vivo da execução
```

## 3. Ambiente

- PostgreSQL 16 via Docker:
  `docker run --name pg -e POSTGRES_PASSWORD=1234 -p 5432:5432 -d postgres:16`
- Banco: `postgres` / usuário `postgres` / senha de exemplo `1234` / porta 5432.
- Python (venv local `.venv/`, já ignorado pelo `.gitignore`):
  `pip install pandas psycopg2-binary openpyxl`
  (o `pypdf` foi instalado apenas para extrair o texto do enunciado PDF)

> **ATENÇÃO:** na máquina atual **não existe Docker, WSL, Podman nem PostgreSQL local**
> e a porta 5432 está fechada. Veja o bloqueio registrado em `PROGRESSO.md`.

## 4. Regras

1. **Nunca alterar** `dados/base_teste.xlsx` nem `sql/01_ddl_enunciado.sql`.
2. **Nunca inventar dado.** Dado suspeito/corrompido deve ser reportado em
   `docs/parte1_importacao.md`, com a consulta SQL que o detecta.
3. **Commits atômicos**: um commit por item do checklist, mensagem descritiva.
4. **Nunca commitar senha real.** Usar `os.environ` (`PGPASSWORD`, `PGHOST`, ...) ou
   valor de exemplo (`1234`). Nunca copiar credenciais para logs/docs.
5. Trabalhar na branch **`feat/etapa1`** (não commitar direto em `main`).
6. Não instalar SQL, pacotes ou serviços sem necessidade; **não instalar Docker/PostgreSQL
   sem perguntar ao usuário**.
7. Não começar a Etapa 2 antes de a Etapa 1 estar concluída e commitada.
8. Preservar o estilo do script original; corrigir, não reescrever do zero.
9. Toda afirmação sobre o banco deve ter sido **testada** (SELECT/saída real), não deduzida.

## 5. Checklist das 3 etapas

### Etapa 1 — Implantação (Parte 1)
- [ ] Ler `docs/Atividade_Integracao_de_Dados.pdf` e `README.md`
- [ ] Ler as 5 abas de `dados/base_teste.xlsx` (completas) e reportar linhas/colunas/anomalias
- [ ] Rodar `sql/01_ddl_enunciado.sql` no PostgreSQL e anotar cada erro (mensagem + tabela + causa)
- [ ] Criar `sql/01_ddl_postgres.sql` corrigido; rodar; confirmar as 5 tabelas
- [ ] Criar `sql/01_ddl_mysql.sql` corrigido (MySQL 8+, InnoDB, `DECIMAL` no lugar de `NUMERIC`/`FLOAT`)
- [ ] Corrigir `scripts/importar.py` (editar, não reescrever)
- [ ] Rodar a importação
- [ ] `SELECT COUNT(*)` por tabela e comparar com a planilha
- [ ] Criar `sql/validar.sql` (contagens, órfãos de FK, duplicidades de PK, faixas de data/ID)
- [ ] Preencher `docs/parte1_importacao.md`

### Etapa 2 — Consultas (Partes 2 e 3)
- [ ] `sql/02_consultas.sql`: total vendido (qtde e R$) por produto em fev/2025
- [ ] `sql/02_consultas.sql`: produtos requisitados e não recebidos
- [ ] `sql/03_transformacoes.sql`: concatenar `produto_id` + `descricao_produto` (`12345 - Detergente`)
- [ ] `sql/03_transformacoes.sql`: datas em `DD/MM/YYYY`
- [ ] `sql/03_transformacoes.sql`: produtos requisitados mais de 10 vezes (explicar a interpretação em comentário)
- [ ] `sql/03_transformacoes.sql`: trigger que gera `idfornecedor` numérico no produto, relacionado a `fornecedor`

### Etapa 3 — Validação (Parte 4)
- [ ] `docs/parte4_validacao.md`: roteiro da reunião de validação de fev/2025
- [ ] `sql/04_validacao.sql`: consultas prontas para a reunião
- [ ] Lista de **todos** os erros de dados, cada um com a consulta que o evidencia

## 6. Hipóteses já levantadas (VERIFICAR — não confiar)

LLM/análise prévia apontou os itens abaixo. Cada um precisa de teste real (rodar o DDL,
rodar o script) antes de ser dado como confirmado em `docs/parte1_importacao.md`.

### 6.1 DDL (`sql/01_ddl_enunciado.sql`)
- `entradas_mercadoria`: a PK cita `ordem_compra`, que **não está declarada** → `CREATE` falha.
- `produtos_filial`: falta vírgula antes de `CONSTRAINT produtos_filial_pkey`; a PK cita
  `idproduto`, que não existe (a coluna é `produto_id`) → falha. Também `decricao` (typo) e
  `idfonecedor` (typo) e `preco_unitario float8 DEFAULT '0'` (string em coluna numérica).
- `fornecedor`: PK cita `idproduto` inexistente; `idforncedor` (typo) → falha.
- `venda`: PK `pk_consumo` (nome enganoso) cita `produto_id varchar(25)`; `horariomov` é
  `varchar(8)` em vez de `TIME`; `qtde_vendida float8` para quantidade.
- `pedido_compra`: PK `(pedido_id, produto_id, item)` com `pedido_id float8`; `filial_id int4`
  sem FK; `ordem_compra float8` sem FK para o pedido.
- Nenhuma **FK** declarada no DDL original, embora o enunciado exija relacionamento
  (`entradas_mercadoria.ordem_compra` → `pedido_compra.ordem_compra`) e exista hierarquia
  `fornecedor` → `produtos_filial` → `venda`/`pedido_compra`.

### 6.2 Script (`scripts/importar.py`)
- `excel_para_data(int(Timestamp))` → `int()` de `Timestamp` lança exceção → `except: return None`
  → **todas as datas viram NULL**.
- `conn.rollback()` dentro do `except` de cada linha **desfaz o lote inteiro** já executado
  (e, sem SAVEPOINT, perde as linhas anteriores do lote).
- `int(row["fornecedor_id"])` é incompatível com `fornecedor.idfornecedor` (`text` `F1..F20`)
  → violação de tipo/FK. Precisa mapear `1..20` → `F1..F20`.
- `produtos_filial`: usa colunas `idproduto`/`descricao`, mas o DDL corrigido usa
  `produto_id`/`descricao`; o DDL original tem `decricao`.
- `pedido_compra`: `df.iloc[:, :12]` **descarta** as colunas extras M..W, que na verdade
  contêm os valores corretos das linhas 21..29 (ver 6.3).
- Contadores: imprime `len(df)` mesmo quando houve erro; não conta inserções reais.
- Não há `SAVEPOINT` por linha; um erro aborta a transação.
- `horariomov` é inserido como texto, mas deveria ser `TIME`.

### 6.3 Dados (`dados/base_teste.xlsx`)
- Vírgula decimal (`78,93`) — **não encontrada** nesta versão do arquivo (valores já são
  numéricos); manter conversão defensiva por segurança.
- Datas no padrão M/D/YYYY (`1/28/2025`) — nesta versão as células são datas reais do Excel;
  o risco existe se alguém ler como texto. Manter parser defensivo.
- `fornecedor_id` numérico (`1..20`) em `pedido_compra`, enquanto `fornecedor.idfornecedor` é
  `F1..F20` (`varchar`).
- **`pedido_compra` tem 23 colunas para 12 campos**: as colunas `M..W` (linhas 22..30 da
  planilha) contêm uma cópia deslocada do bloco `data_pedido..preco_compra` com os valores
  corretos; o bloco `A..L` dessas mesmas linhas está corrompido (`ordem_compra = 0`,
  `qtde_pedida`/`qtde_entregue` truncados). Ler o xlsx completo, não uma amostra.
- `venda.produto_id` tem `P1..P28`, mas `produtos_filial` só tem `P1..P20` → prováveis
  vendas órfãs.
- Possíveis `data_entrega` anteriores a `data_pedido`.
