# Conferir · Análise contabilística da picagem bancária

## 1. Como se faz, em contabilidade, a conferência de movimentos bancários

A **picagem** é o primeiro passo da **reconciliação bancária**. Compara duas
fontes independentes do mesmo dinheiro:

| Fonte | O que é | Sinal |
|---|---|---|
| **Extrato bancário** | O registo do banco (saldo e movimentos da conta à ordem) | Na óptica do banco, um *débito* é uma saída da empresa |
| **Razão da conta 12 – Depósitos à ordem** (SNC) | O registo da empresa | Na óptica da empresa, um *débito* da conta 12 é uma entrada |

Cada movimento do extrato é «picado» (ticado) quando está **suportado por um
documento** (fatura, fatura-recibo, fatura simplificada, recibo, nota de
crédito/débito, aviso bancário) e **lançado na contabilidade** com a
contrapartida certa (fornecedores 22, clientes 21, gastos 62, caixa 11, etc.).

O que não fica picado de um dos lados são os **itens de reconciliação**:

**Registado na contabilidade, ainda não no extrato**
- Cheques emitidos e ainda não apresentados a pagamento (cheques em circulação);
- Depósitos e remessas em trânsito;
- Transferências emitidas ainda não processadas pelo banco.

**No extrato, ainda não registado na contabilidade**
- Comissões, despesas de manutenção, imposto do selo (o suporte é o próprio extrato ou aviso);
- Juros credores e devedores;
- Débitos diretos, transferências recebidas não identificadas, liquidações de TPA.

**Erros**: valores trocados, transposição de dígitos (diferença divisível por 9),
lançamentos em duplicado, movimentos lançados na conta errada.

O resultado é o **mapa de reconciliação bancária**, que tem de fechar a zero:

```
  Saldo segundo o extrato
− Pagamentos registados e ainda não debitados pelo banco
+ Recebimentos registados e ainda não creditados pelo banco
= Saldo bancário ajustado                                   (A)

  Saldo da conta 12 segundo a contabilidade
+ Créditos do banco ainda não lançados
− Débitos do banco ainda não lançados
= Saldo contabilístico ajustado                             (B)

  Diferença por explicar  (A − B)  = 0
```

Também se controla o próprio extrato (saldo inicial + movimentos = saldo
final declarado; saldo corrido sem quebras). No fim, a conferência é **encerrada**
com identificação de quem a preparou e de quem a reviu, e fica arquivada.

Há ainda situações em que o valor do banco não é igual ao do documento, e que
são legítimas:
- **Retenção na fonte**: fatura de 984,00 € com 25 % de IRS retido → o banco
  recebe 738,00 €;
- Descontos financeiros, comissões cobradas na transferência, diferenças de câmbio, arredondamentos;
- **Pagamentos parciais** e liquidações de conta corrente: a fatura foi lançada
  noutro momento, e o movimento salda-a total ou parcialmente.

Em Portugal, os documentos de suporte têm **NIF**, **ATCUD** e **código QR**
(Portaria n.º 195/2020), e o QR contém NIF do emitente, tipo, data, número,
ATCUD, IVA, total e retenção.

## 2. O que a versão original cobria

| Nuance | Original |
|---|---|
| Importar extrato CSV (data, descrição, valor ou débito/crédito) | ✅ |
| Associar um ou vários talões a um movimento, com soma e diferença | ✅ |
| Sugestão automática (valor exato, janela de datas, entidade) com empates pendentes | ✅ |
| Duplicados na importação | ✅ |
| Saldo inicial e saldo final calculado | ✅ (só calculado) |
| OCR de talões e PDF | ✅ |
| Impressão da picagem com espaço para assinatura | ✅ |
| **Saldo corrido do extrato / saldo final declarado** | ❌ Sem controlo |
| **Lado da contabilidade (conta 12)** | ❌ Inexistente |
| **Cheques em circulação e depósitos em trânsito** | ❌ |
| **Mapa de reconciliação bancária** | ❌ |
| **Movimentos sem talão** (comissões, juros, imposto do selo, levantamentos, transferências entre contas) | ❌ Nunca podiam ficar verificados |
| **Diferenças justificadas** (retenção na fonte, desconto, câmbio, arredondamento) | ❌ Ficavam sempre «Divergência» |
| **Retenção na fonte no documento** | ❌ |
| **Classificação contabilística (conta de contrapartida)** | ❌ |
| **Estado «lançado na contabilidade»** e proposta de lançamentos | ❌ |
| NIF, tipo de documento, ATCUD, IVA, código QR da AT | ❌ |
| Documentos em duplicado (mesmo NIF/número ou ATCUD) | ❌ |
| Data-valor | ❌ |
| Encerramento (preparado por / revisto por) e bloqueio de alterações | ❌ |
| Histórico de alterações (trilho de auditoria) | ❌ |
| Extratos exportados por ordem decrescente | ❌ Saldo corrido ficava errado |

## 3. O que esta versão acrescenta

- **Separador «Reconciliação»** com o mapa A/B completo, a diferença por explicar,
  um alerta de transposição de dígitos quando a diferença é divisível por 9, o
  controlo do extrato (saldo final declarado, quebras de saldo corrido) e o encerramento.
- **Registos da contabilidade por refletir no banco** (cheques emitidos,
  depósitos em trânsito, transferências), que podem ser regularizados mais tarde
  com o movimento do extrato correspondente.
- **Justificação sem talão**: extrato/aviso bancário, transferência entre contas
  próprias, movimento de caixa, liquidação de conta corrente, registo já existente na contabilidade.
- **Diferenças justificadas**: retenção na fonte, desconto financeiro, encargo
  bancário, câmbio, arredondamento, pagamento parcial, outra.
- **Documento completo**: tipo (FT, FS, FR, RC, NC, ND, aviso), número, NIF com
  validação do dígito de controlo, ATCUD, IVA, retenção na fonte (o valor
  a liquidar é o total menos a retenção), deteção de duplicados.
- **Leitura do código QR da AT** a partir da imagem (BarcodeDetector nativo ou
  jsQR) ou do texto colado. Preenche NIF, tipo, data, número, ATCUD, IVA, total
  e retenção, e deduz se é compra ou venda a partir do NIF da empresa.
- **Conta de contrapartida (SNC)** com lista de referência e sugestão automática
  pela descrição; indicação «lançado na contabilidade»; **exportação da proposta
  de lançamentos** (débito/crédito) em CSV.
- **Importação**: colunas opcionais de saldo e data-valor, inversão automática de
  extratos em ordem decrescente, saldo inicial e final deduzidos do extrato.
- **Encerramento** com lista de verificação, identificação do preparador e do
  revisor, modo só de leitura e reabertura registada.
- **Histórico** de todas as ações.
- **Relatório impresso** completo: mapa, controlo, movimentos com débito,
  crédito e saldo, pendentes, documentos por associar e assinaturas.

## 4. Redesenho

- Visual empresarial sóbrio: navegação lateral escura com ícones vetoriais,
  barra superior fixa com empresa, IBAN e período, indicadores compactos e
  tabelas densas com cabeçalho fixo.
- Colunas contabilísticas: **Saídas (débito) / Entradas (crédito) / Saldo**,
  números tabulares alinhados à direita.
- Filtros em botões com contagem (incluindo «Por lançar»), pesquisa com o atalho `/`.
- Conferência num **painel lateral**, com o botão «Verificar e seguinte» para
  conferir em série sem voltar à tabela.
- Documentos em tabela (total, retenção, a liquidar, movimento, estado).

## 5. Limitações que ficam

- Um documento só pode pertencer a um movimento. Um segundo pagamento parcial
  justifica-se como «liquidação de conta corrente», sem associar outra vez o documento.
- Uma conta bancária por ficheiro e só em EUR.
- O plano de contas é uma referência simplificada do SNC. A classificação deve ser
  confirmada pelo contabilista certificado.
- A proposta de lançamentos é um ficheiro CSV genérico, não um formato de importação de um ERP concreto.
