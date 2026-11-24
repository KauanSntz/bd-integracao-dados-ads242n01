# PROGRESSO.md — estado vivo

**Última atualização:** 24/11/2026
**Branch:** `feat/etapa1`
**Etapa em andamento:** Etapa 2 (Consultas) — **CONCLUÍDA** ✅

---

## Feito (Etapa 2)

- [x] `sql/02_consultas.sql` criado e testado (2 consultas).
- [x] `sql/03_transformacoes.sql` criado e testado (4 blocos + trigger).
- [x] Total vendido em fev/2025: 14 produtos, P14 lidera (R$ 5.633,90).
- [x] Produtos requisitados e não recebidos: 9 (P12..P20, ordens sem entrada).
- [x] Concatenação `produto_id - descricao` (20 linhas).
- [x] Datas em DD/MM/YYYY (venda e pedido_compra).
- [x] "Mais de 10 vezes": COUNT(*) > 10 → 0 produtos neste dataset (justificado).
- [x] Trigger `trg_produtos_filial_idfornecedor` testado: NULL → F21 (ROLLBACK).

## Em andamento

Nada. Etapa 2 concluída.

## Falhou / Bloqueado

Nenhum.

## Próximo passo concreto

Etapa 2 encerrada. Aguardar autorização para iniciar a Etapa 3
(`docs/parte4_validacao.md` + `sql/04_validacao.sql`).

## Feito (Etapa 1)

- [x] Repositório clonado em `C:\Users\kauan\OneDrive\Documentos\bd-integracao-dados`
      (branch `main`, commit `8b8f089`).
- [x] `AGENTS.md` e `PROGRESSO.md` criados.
- [x] Enunciado lido: `docs/Atividade_Integracao_de_Dados.pdf` (3 páginas) e `README.md`.
- [x] Planilha inspecionada por completo (5 abas, pandas + openpyxl).
- [x] PostgreSQL 16 instalado (Chocolatey) e rodando como serviço `postgresql-x64-16`
      (porta 5432, usuário `postgres`, senha de exemplo `1234`).
- [x] `sql/01_ddl_enunciado.sql` rodado em `enunciado_teste` — 3 erros anotados:
      `entradas_mercadoria` (coluna `ordem_compra` ausente), `produtos_filial` (sintaxe
      + `idproduto` inexistente), `fornecedor` (`idproduto` inexistente). `venda` e
      `pedido_compra` criadas com problemas de modelagem.
- [x] `sql/01_ddl_postgres.sql` criado e validado: 5 tabelas em `atividade`.
- [x] `sql/01_ddl_mysql.sql` criado (MySQL 8+, InnoDB, DECIMAL).
- [x] `scripts/importar.py` corrigido (datas, vírgula decimal, hora TIME, fornecedor
      1..20 → F1..F20, nomes de coluna, SAVEPOINT, contador real).
- [x] Importação rodada: 20/20/29/20/33, 0 rejeitadas.
- [x] `fk_venda_produto` adicionada como `NOT VALID` (pós-importação): 13 órfãos
      P21..P28 mantidos; 3 FKs no banco (2 validadas, 1 NOT VALID).
- [x] `fk_entradas_pedido` impossível: `UNIQUE(ordem_compra,item,produto_id)` violada
      por 2 grupos `(0,1,P19)` e `(0,1,P20)`.
- [x] `sql/validar.sql` criado e executado (contagens OK, 2 entradas órfãs, 13 vendas
      órfãs, 20 datas invertidas, 11 ordens 0, 9 pedidos duplicados, 5 qtdes fracionadas,
      18 divergências de quantidade).
- [x] `docs/parte1_importacao.md` preenchido.
- [x] Script original testado em `script_original` para evidenciar seus erros reais
      (pedido_compra 0 por FK; venda 0 por datas nulas; contadores mentirosos).

## Em andamento

Nada. Etapa 1 concluída.

## Falhou / Bloqueado

Nenhum bloqueio ativo.

## Próximo passo concreto

Etapa 1 encerrada. Aguardar autorização para iniciar a Etapa 2
(`sql/02_consultas.sql`, `sql/03_transformacoes.sql` com trigger).

## Comandos que funcionaram

```powershell
# Banco
$env:PGPASSWORD="1234"; psql -h localhost -U postgres -d postgres -c "CREATE DATABASE atividade;"
$env:PGPASSWORD="1234"; psql -h localhost -U postgres -d atividade -f sql/01_ddl_postgres.sql
$env:PGPASSWORD="1234"; psql -h localhost -U postgres -d atividade -f sql/validar.sql

# Importação
$env:PGDATABASE="atividade"; $env:PGPASSWORD="1234"; .\.venv\Scripts\python.exe scripts\importar.py

# Teste do script original (evidência)
git show origin/main:scripts/importar.py | Set-Content -Encoding UTF8 .../importar_original.py
```

## Comandos que falharam

```powershell
docker --version                 # docker: não reconhecido (resolvido com PG nativo)
wsl -l -v                        # WSL não instalado
choco install postgresql16 ...    # instalador EDB travou ao registrar o serviço
                                  # (resolvido com New-Service + pg_ctl)
```

## Decisões tomadas

1. Venv local `.venv/` no repositório (já coberto pelo `.gitignore`).
2. `pypdf` instalado só para extrair o texto do enunciado.
3. PostgreSQL instalado nativamente (Chocolatey) em vez de Docker, por ausência de Docker.
4. Serviço `postgresql-x64-16` registrado manualmente (`New-Service`) porque o
   instalador EDB travou.
5. `produto_id` padronizado para `varchar(25)` nos dois DDLs.
6. FKs criadas apenas onde os dados permitem (`produtos_filial`→`fornecedor`,
   `pedido_compra`→`fornecedor`); as demais ficam documentadas em `validar.sql`.
7. Importação usa `SAVEPOINT` por linha e `Decimal` para quantidades/preços.
8. Persistência: um commit atômico por item do checklist.
9. Identidade dos commits: `Kauan Ferreira dos Santos <2021331680@ifam.edu.br>`.

## Estrutura de commits (Etapa 1)

| Commit | Conteúdo |
| --- | --- |
| 1 | `chore: adiciona AGENTS.md e PROGRESSO.md` |
| 2 | `docs: leitura do enunciado e inspeção da planilha` |
| 3 | `feat: DDL corrigido para PostgreSQL e MySQL` |
| 4 | `fix: corrige importar.py (datas, decimais, FK fornecedor, SAVEPOINT)` |
| 5 | `feat: adiciona sql/validar.sql` |
| 6 | `docs: preenche parte1_importacao.md` |
| 7 | `docs: preenche tabela Equipe no README` |
| 8 | `feat: fk_venda_produto NOT VALID + registro de fk_entradas_pedido impossivel` |
| 9 | `feat: adiciona sql/02_consultas.sql` |
| 10 | `feat: adiciona sql/03_transformacoes.sql (concat, datas, >10 vezes, trigger)` |
