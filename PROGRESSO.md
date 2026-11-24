# PROGRESSO.md — estado vivo

**Última atualização:** 24/11/2026
**Branch:** `feat/etapa1`
**Etapa em andamento:** Etapa 1 (Implantação) — **BLOQUEADA no passo 3 (banco)**

---

## Feito

- [x] Repositório clonado em `C:\Users\kauan\OneDrive\Documentos\bd-integracao-dados`
      (branch `main`, commit `8b8f089`).
- [x] `AGENTS.md` criado (contexto, regras, ambiente, checklist, hipóteses).
- [x] `PROGRESSO.md` criado.
- [x] Enunciado lido: `docs/Atividade_Integracao_de_Dados.pdf` (3 páginas) e `README.md`.
- [x] Planilha inspecionada por completo (5 abas, via pandas + openpyxl, dados reais de célula).

## Em andamento

- Etapa 1, passo 3: rodar `sql/01_ddl_enunciado.sql` no PostgreSQL. **Sem banco disponível.**

## Falhou / Bloqueado

**BLOQUEIO DE AMBIENTE (decisão do usuário necessária)**

- `docker` não existe (nem Docker Desktop em `C:\Program Files\Docker`).
- Não há PostgreSQL local (`C:\Program Files\PostgreSQL` inexistente, `psql` fora do PATH,
  nenhum serviço `*postgres*`).
- Porta 5432 fechada (`Test-NetConnection localhost:5432` → `False`).
- `wsl` não instalado; `podman` inexistente.
- Consequência: passos 3, 7 e 8 da Etapa 1 (rodar DDL, importar, `COUNT(*)`) estão bloqueados.
  Não instalei nada de banco, conforme a regra "perguntar antes de instalar".

## Próximo passo concreto

1. Usuário escolhe como disponibilizar o PostgreSQL 16 (Docker Desktop, instalador nativo,
   ou informar um banco já existente).
2. Criar o banco/confirmar conexão e rodar `sql/01_ddl_enunciado.sql`, anotando erro por erro.
3. Seguir o checklist da Etapa 1 em `AGENTS.md`.

## Levantamento da planilha (passo 2 — concluído)

| Aba | Linhas | Colunas | Observação |
| --- | --- | --- | --- |
| `venda` | 33 | 9 | `venda_id` 1..33 único; `filial_id` 1..3; `horariomov` sempre `08:00:00`; `item` sempre 1; `unidade_medida` sempre `UN`; datas 2025-01-10..2025-03-28; 28 produtos distintos (`P1..P28`) |
| `pedido_compra` | 29 | **23** | 12 campos + 11 colunas extras `Unnamed: 12..22`. `pedido_id` 1..29 único; `filial_id` sempre 1; `ordem_compra` 0 nas linhas 21..29 (suspeito) |
| `entradas_mercadoria` | 20 | 9 | `nro_nfe` `NFE1..NFE20`; `ordem_compra` 1..20; `item` sempre 1; `filial_id` sempre 1 |
| `produtos_filial` | 20 | 8 | colunas reais: `filial_id, idproduto, descricao, estoque, preco_unitario, preco_compra, preco_venda, idfornecedor`; `P1..P20`; `idfornecedor` `F7..F20` (17 distintos) |
| `fornecedor` | 20 | 2 | `idfornecedor` `F1..F20`, `razao_social` "Fornecedor N LTDA" |

### Anomalias confirmadas na planilha (detalhe em `AGENTS.md` §6.3)

1. **`pedido_compra` com 23 colunas.** Existe um bloco duplicado em `M..W` nas linhas
   22..30 do Excel (pedidos 21..29). O bloco `A..L` dessas linhas está corrompido:
   `ordem_compra = 0`, `qtde_pedida` pequena, `qtde_entregue = 0`. O bloco `M..W` é
   internamente consistente (`qtde_pendente = qtde_pedida - qtde_entregue` fecha em todas
   as 9 linhas) e casa com `entradas_mercadoria.ordem_compra` 12..20. **Decisão pendente
   de registro final**: importar os valores de `M..W` (reconciliados, `fornecedor_id` vem de
   `L`) e documentar a correção + consulta de detecção.
2. **`venda.produto_id` vai até `P28`**, mas `produtos_filial` só tem `P1..P20` → prováveis
   vendas órfãs (8 produtos).
3. `pedido_compra.fornecedor_id` é `1..20` (inteiro) e `fornecedor.idfornecedor` é `F1..F20`
   (texto) → exige mapeamento.
4. Hipóteses de vírgula decimal e de datas M/D/YYYY **não se confirmaram** nesta versão:
   as células já são numéricas e datas reais do Excel. Manter parser defensivo.

## Comandos que funcionaram

```powershell
git clone https://github.com/agnpreto/bd-integracao-dados-ads242n01.git "C:\Users\kauan\OneDrive\Documentos\bd-integracao-dados"
cd "C:\Users\kauan\OneDrive\Documentos\bd-integracao-dados"
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install pandas openpyxl psycopg2-binary pypdf
.\.venv\Scripts\python.exe -c "import pandas, openpyxl, psycopg2, pypdf; print('ok')"
```

Extração do PDF (necessário porque o modelo não lê PDF direto):

```powershell
.\.venv\Scripts\python.exe -c "from pypdf import PdfReader; r=PdfReader('docs/Atividade_Integracao_de_Dados.pdf'); open('C:/Users/kauan/AppData/Local/Temp/opencode/enunciado.txt','w',encoding='utf-8').write('\n\n'.join(p.extract_text() for p in r.pages))"
```

## Comandos que falharam

```powershell
docker --version                 # docker: não reconhecido
docker ps -a                     # docker: não reconhecido
wsl -l -v                        # WSL não instalado
Test-NetConnection localhost 5432  # False (porta fechada)
python -c "import pandas"        # ModuleNotFoundError (antes do venv)
```

## Decisões tomadas

1. Venv local `.venv/` no repositório (já coberto pelo `.gitignore`), em vez de instalar
   pacotes no Python global.
2. `pypdf` instalado só para extrair o texto do enunciado (não é dependência da entrega).
3. Persistência: **um commit atômico por item** do checklist; estado sempre no `PROGRESSO.md`
   antes de avançar.
4. Nada de banco será instalado sem autorização explícita do usuário.

## Estrutura de commits

| Commit | Conteúdo |
| --- | --- |
| 1 | `chore: adiciona AGENTS.md e PROGRESSO.md` |
| 2 | `docs: leitura do enunciado e inspeção das 5 abas da planilha` (se ainda não incluído no 1) |
| ... | um por item do checklist |
