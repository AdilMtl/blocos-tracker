# Guia de Interpretação do JSON exportado pelo Kcal.ix

Este documento consolida a lógica de programação para o backend do seu aplicativo, integrando as métricas do banco de dados (Kcal.ix) com os princípios técnicos e científicos defendidos pelo Lucas Campos.
Documento 4: interpretacao-kcalix-json.md
Este guia orienta como o algoritmo do "Nutricionista Virtual" deve interpretar os dados exportados para realizar ajustes precisos na dieta e no treino, baseando-se no comportamento real do usuário.
1. Campos do Profile e Metas Metabólicas
Os campos abaixo definem a base matemática para a prescrição individualizada.
Campo,Significado,Aplicação Algorítmica
blockSize.pG,Gramas de proteína/bloco,"Conversão para bater a meta de 1.6g a 2.5g/kg 1, 2."
bodyData.activity,Fator de atividade TDEE,Lucas sugere 1.4 a 1.5 para praticantes padrão 3-5.
bodyData.deficitPct,% de déficit/surplus,"Define o objetivo (Cut, Bulk ou Recomp) 6, 7."
2. Cálculo de Gasto Energético (TDEE)
O app deve utilizar as fórmulas citadas nas fontes para maior precisão:
BMR (Taxa Metabólica Basal): Utilizar Mifflin-St Jeor para usuários padrão ou Tinsley para usuários com maior massa muscular, conforme recomendado 5, 8.
TDEE (Gasto Energético Total): BMR × activity.
Meta Calórica: TDEE × (1 - deficitPct/100).
Nota: O valor da fórmula é uma estimativa inicial; o "padrão ouro" para ajuste é a oscilação do peso real 3, 9, 10.
3. Lógica de Aderência e Qualidade
A aderência é o preditor número um de sucesso no emagrecimento 11, 12.
Cálculo: aderênciaP (%) = (totalP × blockSize.pG) / goals.pG × 100.
Análise de Qualidade (foodLog): O algoritmo deve verificar se 80-90% das calorias vêm de "comida de verdade" (arroz, feijão, carnes magras, frutas e vegetais) 13-16. O foodLog identifica se o usuário está "beliscando" ou abusando de ultraprocessados, o que compromete a saciedade 17, 18.
4. Detecção Automática de Platô
Seguindo a regra prática de observação clínica do Lucas:
Critério: Variação de peso < 0.5kg em 14 dias 19, 20.
Condição de Validação: Aderência calórica média > 80%.
Ação do App: Se houver platô real, sugerir redução de 200-300 kcal ou aumento do gasto via aeróbico 20-22.
5. Padrões Comportamentais (Weekly Trends)
Frequência Alimentar: O app não deve forçar 6 refeições; a distribuição deve seguir a rotina do usuário (ex: 4 refeições), pois o resultado depende do balanço crônico, não de janelas agudas 23-25.
Alertas de Fim de Semana: Se a aderência cair consistentemente aos sábados/domingos (< 70%), o app deve sugerir a inclusão de alimentos palatáveis (ex: doce de leite, chocolate) de forma controlada para evitar o efeito rebote 13, 17, 26.
6. Unidades e Conversões Fisiológicas
Peso: kg | Medidas: cm (foco em circunferência abdominal como indicador de gordura visceral) 27, 28.
Gordura Corporal: % (Abaixo de 10-12% para homens exige déficit moderado para evitar perda de massa magra) 29-31.
Equivalência Energética: Carboidrato/Proteína (4 kcal/g) e Gordura (9 kcal/g) 32, 33.
Observação Técnica para o Desenvolvedor:
Esta estrutura permite que o seu "Nutricionista Virtual" não apenas conte calorias, mas realize ajustes dinâmicos. Por exemplo, se o bodyData.deficitPct for positivo (Cutting) e a aderência for alta, mas o peso não cair em 14 dias, o algoritmo "sabe" que a fórmula superestimou o gasto energético e deve disparar um ajuste corretivo automático de calorias 9, 20.

