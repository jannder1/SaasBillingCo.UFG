
# SaaS Billing Co. - Refatoração do Núcleo Financeiro

**Disciplina:** Implantação e Entrega de Software - 2026/2  
**Professora:** Sofia Costa Paiva  
**Instituição:** Instituto de Informática - UFG  
**Curso:** Engenharia de Software  

---

## 👥 Integrantes

* **Aluno:** Jannderson Oliveira

---

## 📌 Visão Geral do Projeto

Este repositório contém a refatoração do núcleo financeiro da plataforma **SaaS Billing Co.**[cite: 2]. O sistema é responsável por:
* Cálculo mensal de assinaturas[cite: 2].
* Aplicação de regras de descontos (fidelidade, ONGs e cupons de promoção)[cite: 1, 2].
* Aplicação de juros e multas por atraso[cite: 1].
* Processamento de cobrança via cartão de crédito[cite: 1, 2].
* Emissão de recibos por e-mail[cite: 1, 2].

### 🎯 Objetivo da Refatoração
Eliminar o débito técnico severo acumulado na solução legado sem alterar nenhuma regra ou comportamento de negócio existente, garantindo que a esteira de Integração Contínua (CI) execute 100% dos testes de unidade de forma determinística e isolada de infraestrutura externa (sem depender de PostgreSQL ou servidores SMTP locais)[cite: 2].

---

## 🛠️ Arquitetura e Decisões de Refatoração

A refatoração seguiu os princípios da **Arquitetura Hexagonal (Portas e Adaptadores)** e do **Domain-Driven Design (DDD)**[cite: 2]:

1. **Eliminação de Primitive Obsession e Imutabilidade:**
   * **`Money` (Value Object):** Encapsula operações de adição, subtração, multiplicação e validações numéricas utilizando `BigDecimal` para evitar erros de ponto flutuante[cite: 2].
   * **`BillingCommand` (Record):** Encapsula os parâmetros de entrada em um objeto imutável de intenção/comando[cite: 2].

2. **Desacoplamento de I/O e Inversão de Dependência:**
   * Extração das portas (interfaces): `CustomerRepository`, `PaymentGateway` e `NotificationService`[cite: 2].
   * Injeção de dependência de `java.time.Clock` para garantir que o tempo em testes seja 100% determinístico[cite: 2].

3. **Isolamento de Regras de Negócio e Simplificação:**
   * **`PricingCalculator`:** Motor puro de precificação contendo toda a lógica matemática[cite: 3].
   * **`SubscriptionBillingService`:** Atua como uma fachada/orquestradora de alto nível reduzida a menos de 20 linhas[cite: 3].

---

## 📊 1. Matriz de Code Smells e Violações Arquiteturais

| # | Trecho / Linha Afetada | Code Smell / Princípio Violado | "Juros" Decorrentes (Impacto Prático) |
|---|---|---|---|
| **1** | `double basePrice, double calculatedPrice`[cite: 1] | **Primitive Obsession**[cite: 2] | Operações com `double` geram imprecisão em casas decimais de valores financeiros[cite: 2]. |
| **2** | Assinatura com 8 parâmetros soltos[cite: 1] | **Long Parameter List**[cite: 2] | Dificulta a manutenção e aumenta o risco de inversão inadvertida de argumentos do mesmo tipo[cite: 2]. |
| **3** | `DriverManager.getConnection(...)` e `new SimpleEmail()`[cite: 1] | **Dependência Direta de I/O / Violação do SRP**[cite: 2] | Impede a execução de testes unitários na esteira de CI sem subir banco PostgreSQL e SMTP local[cite: 2]. |
| **4** | `LocalDate.now()` estático[cite: 1] | **Hardcoded Time Dependency**[cite: 2] | Torna os testes temporais não determinísticos, pois dependem do relógio do sistema em tempo de execução[cite: 2]. |
| **5** | Método único acumulando mais de 80 linhas[cite: 1] | **Long Method / God Method**[cite: 2, 3] | Alta complexidade ciclomática, dificultando manutenção e aumentando o risco de regressões[cite: 3]. |
| **6** | `catch (Exception e)` abrangente[cite: 1] | **Swallowed / Masked Exceptions**[cite: 3] | Mascara erros graves de infraestrutura ao apenas registrar em lista de logs sem tratamento[cite: 1, 3]. |

---

## 📈 2. Tabela Comparativa de Métricas (Antes vs. Depois)

| Métrica / Critério | Antes da Refatoração (Legado) | Depois da Refatoração | Impacto na Manutenibilidade e CI |
|---|---|---|---|
| **Maior Complexidade Ciclomática** | `14` (método único) | `2` (método orquestrador) | Fluxo de execução simples, linear e testável. |
| **Tamanho do Método Orquestrador** | ~85 linhas[cite: 1] | ~18 linhas | Método coeso atuando como fachada de alto nível[cite: 3]. |
| **Execução em CI (Infraestrutura)** | Dependia de PostgreSQL e SMTP[cite: 2] | **100% isolada** (via Mocks/Stubs)[cite: 2] | Testes rápidos e independentes de ambiente[cite: 2]. |
| **Determinismo em Testes Temporais** | Não determinístico (`LocalDate.now()`)[cite: 1, 2] | Determinístico (via `java.time.Clock`)[cite: 2] | Permite simulação precisa de datas e cupons[cite: 1, 2]. |
| **Contrato de Entrada** | 8 parâmetros primitivos[cite: 1, 2] | 1 objeto de comando (`BillingCommand`)[cite: 2] | Garante a integridade e validação na entrada[cite: 2]. |
| **Precisão Financeira** | `double` (risco de arredondamento)[cite: 1, 2] | `Money` Value Object (`BigDecimal`)[cite: 2] | Precisão monetária garantida[cite: 2]. |

---

## 📜 3. Histórico Estruturado no Git

O repositório segue a evolução em 4 commits estruturados[cite: 3]:

* `commit 1`: Adição do código legado base[cite: 3].
* `commit 2`: Introdução dos Value Objects e remoção de Primitive Obsession[cite: 3].
* `commit 3`: Extração das interfaces de infraestrutura e injeção de dependência[cite: 3].
* `commit 4`: Refatoração do motor de precificação e validação da suíte verde[cite: 3].

```
