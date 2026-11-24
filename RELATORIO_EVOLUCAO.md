# Relatório de Evolução e Resolução de Problemas — MarketGo

**Disciplina:** Programação para Dispositivos Móveis  
**Projeto:** MarketGo (Compras com entrega)  
**Instituição:** IESB  
**Data:** 24 de Novembro de 2026  

---

## 1. Visão Geral da Construção

O projeto **MarketGo** foi estruturado a partir do zero seguindo rigorosamente todas as etapas do guia oficial da disciplina, empregando arquitetura limpa em camadas:
- **Rotas / Navegação:** Expo Router com grupos protegidos `(auth)` e `(app)` e abas `(tabs)`.
- **Regras de Negócio Puras:** Lógica do carrinho (`cart.ts`), catálogo (`catalog.ts`), pedidos (`orders.ts`), validação de checkout (`checkout.ts`) e cálculos monetários em centavos (`money.ts`).
- **Segurança no Banco (Postgres + Supabase):** RLS ativado em todas as tabelas (`stores`, `products`, `orders`, `order_items`), impedindo escrita direta do app nas tabelas de pedidos. Toda escrita e cálculo financeiro ocorrem exclusivamente via funções armazenadas (RPC) `create_order`, `advance_order` e `cancel_order`.
- **Camada de Dados & Estado:** `@tanstack/react-query`, `AuthContext` e `CartContext` com persistência segura por usuário no `AsyncStorage`.

---

## 2. Erros e Desafios Encontrados no Processo & Soluções Aplicadas

Durante a configuração e verificação automática com as versões mais recentes das dependências (Node 25, React Native 0.88, React 19.3, TypeScript 6), surgiram alguns pontos técnicos que foram diagnosticados e solucionados:

| # | Sintoma / Erro | Causa Raiz | Solução Aplicada |
|---|---|---|---|
| **1** | `O token '&&' não é um separador de instruções válido nesta versão` | No terminal Windows PowerShell, encadear comandos com `&&` pode falhar dependendo da versão do interpretador. | Utilizado o separador de comandos compatível com PowerShell (`;`). |
| **2** | `FAIL supabase/tests/db.test.mjs: Must use import to load ES Module` ao rodar `npm run test:app` | O executor padrão do Jest tentou executar o arquivo `.mjs` de testes do PGlite que utiliza sintaxe ES Module nativa do Node. | Configurado `"testPathIgnorePatterns": ["/node_modules/", "/supabase/"]` no `jest` do `package.json`, garantindo que o Jest rode apenas os testes do app (`__tests__/`) e o `test:db` execute nativamente no Node. |
| **3** | `error TS2591: Cannot find name 'process'` no `src/lib/supabase.ts` | No TypeScript 6, os tipos de `@types/node` não são inferidos automaticamente sem a definição explícita no `tsconfig.json`. | Instalado `@types/node` em `devDependencies` e adicionado `"types": ["jest", "node"]` no `tsconfig.json` e referência no `expo-env.d.ts`. |
| **4** | `error TS2322: Type '____ColorValue_Internal' is not assignable to type 'string'` no `TabsLayout` | A tipagem do `react-native-screens` e `expo-router` no React 19 / RN 0.88 usa uma tipagem interna estrita para a prop `color` da função de renderização do ícone. | Ajustada a assinatura em `tabIcon` para aceitar a prop e fazer fallback seguro com `color ?? colors.muted`. |
| **5** | `error TS2769: RefreshControl onRefresh` incompatibilidade com retorno de `refetch()` | O `refetch` do TanStack Query retorna `Promise<QueryObserverResult>`, enquanto a prop `onRefresh` do `RefreshControl` do React Native espera `() => void \| Promise<void>`. | Envolvido o chamado em função anônima: `onRefresh={() => { refetch(); }}` em `index.tsx` e `orders.tsx`. |
| **6** | Bloqueio de e-mail / tela de confirmação após cadastro | Fluxo nativo com confirmação por e-mail ligada e validação de regex restrita no cliente. | Flexibilizada a regra do Zod para o campo e ajustado o `register.tsx` para realizar auto-login transparente. |

---

## 3. Resumo dos Testes Automatizados

A suíte completa de **50 testes automatizados** foi executada e aprovada com 100% de sucesso:

```
> npm test

PASS __tests__/cart.test.ts
PASS __tests__/catalog.test.ts
PASS __tests__/checkout.test.ts
PASS __tests__/money.test.ts
PASS __tests__/orders.test.ts

Test Suites: 5 passed, 5 total
Tests:       37 passed, 37 total (Jest)

▶ leitura e permissões (3 testes)
  ✔ usuário logado lê lojas e só os produtos ativos
  ✔ o app não consegue inserir pedidos diretamente
  ✔ visitante (anon) não executa as funções
▶ create_order (6 testes)
  ✔ calcula o total no servidor e guarda os itens (snapshot)
  ✔ respeita o pedido mínimo da loja
  ✔ recusa loja fechada
  ✔ recusa itens inválidos: outra loja, inativo, quantidade e repetidos
  ✔ valida carrinho vazio, login, endereço e pagamento
  ✔ troco: só vale em dinheiro e precisa cobrir o total
▶ privacidade e status (4 testes)
  ✔ cada usuário só vê os próprios pedidos e itens
  ✔ o pedido avança na ordem certa e para em entregue
  ✔ outro usuário não avança nem cancela o pedido alheio
  ✔ só cancela enquanto o pedido está recebido

Tests:       13 passed, 13 total (PGlite / Node test runner)

Total Geral: 50 testes passando (37 App + 13 Banco), 0 falhas!
Typecheck:   tsc --noEmit concluído sem nenhum erro.
Formatação:  Prettier executado e padronizado.
```

---

## 4. Próximos Passos para Execução no Celular (Expo Go)

1. **Configurar o Supabase:**
   - Crie o projeto no Supabase.
   - Execute no SQL Editor o script `supabase/schema.sql` e em seguida o `supabase/seed.sql`.
   - Copie a URL do Projeto e a anon key para o arquivo `.env`:
     ```env
     EXPO_PUBLIC_SUPABASE_URL=https://SEU-PROJETO.supabase.co
     EXPO_PUBLIC_SUPABASE_KEY=sua-chave-anon-publica
     ```
2. **Rodar no Celular:**
   - Execute no terminal:
     ```bash
     npx expo start -c
     ```
   - Abra o aplicativo **Expo Go** no celular e escaneie o QR Code (ou utilize `npx expo start --tunnel` caso esteja em rede que bloqueie conexão direta).
