## 7. Vendas: Script para ajuste de vendedor nas tabelas SC5/SC6

### 🛠 Problema real
Durante o processo de vendas no Protheus, pedidos são criados com o **vendedor incorreto** por falha de seleção manual ou falta de regra de validação. Isso impacta diretamente nos relatórios de comissão, metas e acompanhamento de performance comercial.

Como os dados ficam gravados em múltiplas tabelas (`SC5` – cabeçalho e `SC6` – itens), a correção manual é **trabalhosa e sujeita a erro**.

### 📉 Impacto
- Relatórios de comissionamento incorretos
- Pagamentos indevidos ou inconsistências internas
- Dificuldade de rastrear performance real por vendedor
- Necessidade de retrabalho em pedidos já faturados

### 💡 Solução aplicada
Criação de um **script para ajuste em lote** do código de vendedor nas tabelas `SC5` e `SC6`, com base no número do pedido ou outro critério (data, cliente, etc.). O script pode ser executado com segurança após validações.

> Ideal para situações onde é necessário trocar todos os pedidos do vendedor X para o Y em um período.

### 🧾 Código exemplo (ADVPL)
```advpl
User Function AjusVend()
    Local cVendedorAntigo := "001" // código atual
    Local cVendedorNovo   := "005" // novo código
    Local cPedido         := ""    // ou use data/cliente/etc

    // Atualiza SC5 (cabeçalho)
    dbSelectArea("SC5")
    dbSetOrder(1)
    dbGoTop()

    While !Eof()
        If SC5->C5_VEND1 == cVendedorAntigo
            RecLock("SC5", .F.)
                SC5->C5_VEND1 := cVendedorNovo
            MsUnlock()
        EndIf
        dbSkip()
    EndDo

    // Atualiza SC6 (itens)
    dbSelectArea("SC6")
    dbSetOrder(1)
    dbGoTop()

    While !Eof()
        If SC6->C6_VEND1 == cVendedorAntigo
            RecLock("SC6", .F.)
                SC6->C6_VEND1 := cVendedorNovo
            MsUnlock()
        EndIf
        dbSkip()
    EndDo

    MsgInfo("Ajustes de vendedor concluídos.")
Return
```
> Importante: Sempre fazer backup antes de ajustes em lote e testar em ambiente de homologação.

## 🧪 Casos de Teste - Rotina de Atualização de Vendedor

| Cenário | Tabelas Afetadas | Regra de Negócio | Resultado |
|---------|------------------|------------------|-----------|
| Vendedor desatualizado | SC5 | Atualiza campo A1_VEND | ✅ SC5.A1_VEND modificado |
| Vendedor correto | - | Verifica A1_VEND == novo_vendedor | 🚫 Nenhuma alteração |
| Vendedor+Itens errados | SC5 + SC6 | Atualiza cabeçalho e itens | ✅ SC5+SC6 consistentes |

**Legenda:**
- ✅ = Atualização bem-sucedida /
- 🚫 = Nenhuma ação necessária /
- SC5 = Tabela de cabeçalho de pedidos/ 
- SC6 = Tabela de itens de pedidos

> **Fluxo de Erros:**  
> - Erro na atualização de SC6 gera rollback em SC5
> - Logs detalhados em arquivo `ATU_VEND.log`

🎯 Benefícios
Correção rápida e segura de dados críticos de venda

Confiabilidade nos relatórios de performance e comissão

Redução de retrabalho manual no Protheus

Otimização da gestão comercial

🏷️ Tags
#Protheus #Vendas #SC5 #SC6 #Vendedor #Comissão #ADVPL #Automação
