# challenge-alura-telecomX_parte2
Desenvolver modelos preditivos capazes de prever quais clientes têm maior chance de cancelar seus serviços.

Relatório — Predição de Churn (Telecom X)
1) Objetivo

Desenvolver modelos preditivos para identificar clientes com maior probabilidade de evasão (churn) e, a partir dos resultados, entender os principais fatores associados ao cancelamento para orientar ações de retenção.

2) Preparação dos dados e premissas

A base foi normalizada a partir de um JSON com campos aninhados (customer/phone/internet/account), transformando-os em colunas tabulares.

Foram removidas colunas sem valor preditivo (ex.: customerID).

Variáveis categóricas foram transformadas via One-Hot Encoding.

O target (evasão) foi transformado em binário.

Foi identificado desbalanceamento moderado (~74% não churn / ~26% churn). Para aprofundar, foi aplicado SMOTE, resultando em base balanceada 50/50.

Foi aplicada padronização (StandardScaler) para modelos sensíveis à escala (ex.: Regressão Logística).

3) Evidências exploratórias (o que os dados mostraram)
3.1 Correlação com churn (principais sinais)

As variáveis com maior associação positiva com churn foram:

InternetService_Fiber optic

PaymentMethod_Electronic check

Charges.Monthly

PaperlessBilling_Yes

SeniorCitizen

Variáveis com baixa relevância na correlação:

gender_Male

PhoneService_Yes

Interpretação: churn se conecta mais a tipo de internet, forma de pagamento e custo mensal do que a atributos demográficos simples.

3.2 Tenure e Total gasto vs churn (boxplots + scatter)

Os gráficos mostraram padrões consistentes:

Clientes que evadem tendem a ter tenure menor (cancelam mais cedo).

Clientes que permanecem tendem a ter maior gasto total (efeito natural de permanecer mais tempo).

O scatter indicou uma relação forte: tenure ↑ → Charges.Total ↑, e churn se concentra em tenure mais baixo.

Interpretação: existe um “período crítico” no início do ciclo de vida do cliente onde a probabilidade de churn é mais alta.

4) Modelos e desempenho
4.1 Regressão Logística (com normalização)

Justificativa: modelo linear, interpretável e boa baseline. Precisa de normalização porque é sensível à escala.

Métricas (teste):

Accuracy: ~0.831

F1-score: ~0.83

Matriz de confusão:

TN=1339 | FP=255

FN=291 | TP=1354

Leitura dos erros:

FN (291) = churns não detectados (risco: perder cliente sem ação).

FP (255) = alertas falsos (custo: campanha de retenção desnecessária).

4.2 Random Forest (sem normalização)

Justificativa: captura relações não lineares e interações. Não é sensível à escala.

Métricas (teste):

Accuracy: ~0.842

F1-score: ~0.84

Matriz de confusão:

TN=1333 | FP=261

FN=249 | TP=1396

Leitura dos erros:

Reduziu FN (249 vs 291) e aumentou TP (1396 vs 1354), ou seja:

melhor para identificar churn de fato, que é geralmente o foco do negócio.

5) Comparação crítica: melhor modelo e sinais de over/underfitting
Melhor desempenho

O Random Forest foi superior no conjunto de teste:

Melhor accuracy e F1

Melhor recall para churn (menos churn “escapando” como FN)

Overfitting / Underfitting

Não há evidência forte de overfitting/underfitting apenas olhando as métricas de teste atuais.

Em geral:

A Regressão Logística pode underfit se existirem relações muito não lineares (ela é uma fronteira linear).

Random Forest pode overfit se crescer demais (árvores muito profundas), mas o desempenho de teste ficou estável.

Ajustes possíveis (se necessário):

Random Forest: max_depth, min_samples_leaf, min_samples_split, n_estimators

Logística: regularização (C), seleção de features, interações/polinômios

6) Principais fatores que influenciam churn (síntese)

Com base nas análises (correlação + padrões visuais + comportamento esperado em modelos):

Fatores que aumentam risco de churn

Internet fibra óptica
Indica possível sensibilidade a preço, qualidade, suporte ou expectativa de serviço.

Pagamento via electronic check
Pode representar fricção, perfil menos engajado ou risco transacional.

Cobrança mensal alta (Charges.Monthly)
Relação direta com percepção de custo/benefício.

PaperlessBilling e SeniorCitizen
Aparecem como sinais associados, mas podem ser mais “proxy” de perfil do que causa direta.

Fatores que reduzem risco de churn (retêm)

Tenure alto
Quanto mais tempo, menor o churn — reforça o “período crítico” inicial.

Maior gasto total (Charges.Total)
Normalmente acompanha tenure e estabilidade.

7) Estratégias de retenção orientadas por dados
Estratégia A — “Programa de retenção no início da jornada” (tenure baixo)

Público-alvo: clientes nos primeiros 1–3 meses
Ações:

Onboarding proativo (mensagens/educação de uso)

Check de qualidade de serviço e suporte “early life”

Oferta de upgrade/benefício temporário (ex.: desconto 1–2 meses) condicionado a permanência

Por que funciona: churn concentra-se no início; reter cedo aumenta LTV.

Estratégia B — “Reprecificação/Oferta para clientes com Charges.Monthly alto”

Público-alvo: clientes com cobrança mensal alta + sinais de risco
Ações:

Oferta de plano com melhor custo-benefício

Bundles (serviços agregados) com preço menor que o pacote atual

Revisão de fatura e transparência (reduzir surpresa/cobranças percebidas como indevidas)

Por que funciona: custo mensal alto correlaciona com churn.

Estratégia C — “Redução de fricção no pagamento (Electronic Check)”

Público-alvo: clientes em PaymentMethod_Electronic check
Ações:

Incentivar migração para métodos automáticos (débito/CC) com benefício

Comunicação sobre facilidade e segurança

Campanhas de “1 clique” para troca do método

Por que funciona: forma de pagamento está entre os sinais mais fortes.

Estratégia D — “Plano de ação específico para fibra óptica”

Público-alvo: InternetService_Fiber optic
Ações:

Diagnóstico de qualidade: quedas, velocidade percebida, chamados

Ajuste de SLA e suporte dedicado

Oferta de upgrade de modem/roteador ou visita técnica com critérios

Por que funciona: serviço de fibra aparece como forte sinal de churn — pode ser qualidade ou preço.

8) Recomendação final

Modelo recomendado: Random Forest
Motivo: melhor desempenho e maior capacidade de capturar padrões complexos, reduzindo falsos negativos (churn não detectado).

Uso sugerido na operação:

Gerar uma lista semanal de clientes com maior risco

Priorizar ações nos segmentos de maior impacto:

tenure baixo

cobrança mensal alta

payment electronic check

fibra óptica
