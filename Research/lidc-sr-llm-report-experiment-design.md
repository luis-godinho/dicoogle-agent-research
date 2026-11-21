# Experiência LIDC: relatório de um LLM geral vs SR de referência

Estado: rascunho de desenho, ainda não implementado. Os resultados da literatura abaixo vêm de excertos de pesquisa, não de leitura integral dos artigos; confirmar nos PDFs antes de citar.

## 1. Porquê mudar o piloto

O piloto `exp-lidc_idri_pilot-60994d3b` (2 pacientes, 4 pedidos: 3 sucessos, 1 `connector_error:http_5xx`) testou classificação benigno/maligno com e sem overlay SEG, comparada com labels ao nível do paciente.

Limitações:

- Os labels são de paciente; o prompt pedia classificação das regiões nodulares. Não há crosswalk autorizado entre diagnósticos nodulares e SEG (ex.: LIDC-IDRI-0072 é maligno ao nível do paciente, mas Nodule 1 é benigno; as três SEG são Nodule 1).
- Métricas com n=2 (balanced accuracy 1.0, AUROC 1.0) não são evidência de desempenho.
- Só existe um par completo sem/com overlay; não permite concluir sobre benefício do overlay.
- O objetivo real não é benigno/maligno nem overlay.

O piloto fica como teste técnico do pipeline. Não foi feita validação diagnóstica.

## 2. Objetivo

Medir a viabilidade de um LLM geral produzir, a partir de estudos sem anotações, um relatório estruturado de nódulos pulmonares que concorde com as anotações dos radiologistas (SR do LIDC). O planeador que encaminha para modelos especializados fica para uma fase posterior.

## 3. Papel de cada artefacto

| Artefacto | Papel |
|---|---|
| Imagens CT (cortes) | Input do LLM |
| SEG | Escolher cortes e alinhar cada relatório ao nódulo do SR; nunca é input do LLM |
| SR | Referência escondida, usada só na avaliação |
| Diagnóstico do paciente | Referência separada e mais fraca; não equivale a diagnóstico do nódulo |

O overlay SEG sai do desenho principal, pois o fluxo final usa estudos sem anotações. Só seria estudado à parte.

## 4. Seleção de cortes (oráculo)

No fluxo real não há SEG. Na experiência, escolher cortes com a SEG é um oráculo: mede a qualidade do relatório quando o LLM recebe cortes relevantes (limite superior), não a deteção em estudo completo.

Condições sugeridas:

- A: cortes escolhidos pela SEG.
- B: amostragem uniforme de cortes, sem SEG.

A diferença entre A e B indica a dependência de uma etapa de seleção. Testar também o número de cortes como variável.

Deteção em estudo completo não é impossível só por custo/limites de pedido: também há possível degradação com mais imagens e a referência é incompleta (o SR só inclui nódulos de 3 mm ou mais; um achado sem anotação pode ser real). Confirmar na documentação os limites atuais de imagens/tokens do modelo.

## 5. Referência: o que o SR contém

DICOM-LIDC-IDRI-Nodules converte as anotações do LIDC em SEG e SR. O pylidc expõe nove características por anotação: subtlety, internalStructure, calcification, sphericity, margin, lobulation, spiculation, texture e malignancy. A maioria usa escala ordinal 1 a 5 (internalStructure: 1 a 4). `malignancy` é avaliação subjetiva do radiologista, não diagnóstico confirmado. Cada nódulo pode ter até quatro leituras, que podem divergir.

Passo prévio, só leitura: inspecionar os SR já descarregados (campos, ligação às SEG, se os valores por leitor estão preservados).

## 6. Saída estruturada do LLM

Campos derivados dos atributos realmente disponíveis no SR, por exemplo: nódulo presente, diâmetro estimado, textura, margem, espiculação, lobulação, calcificação, suspeita de malignidade, incerteza/não avaliável. Os campos e valores finais devem sair da inspeção dos dados.

Permitir `não avaliável` como resposta válida.

## 7. Comparação de resultados

1. Alinhar cada relatório a um nódulo do SR (via SEG); definir regra para múltiplos nódulos descritos.
2. Métricas por tipo de campo:

| Tipo | Exemplo | Métrica |
|---|---|---|
| Presença | Nódulo presente | Sensibilidade, especificidade |
| Contínuo | Diâmetro | Erro absoluto (mm), viés |
| Ordinal | Espiculação, margem, textura | Diferença absoluta média; % dentro de ±1 nível |
| Categórico | Calcificação | Accuracy, matriz de confusão |
| Suspeita de malignidade | malignancy 1-5 | Spearman, diferença absoluta |

3. Leitores: escolher a regra antes de ver resultados (por defeito, mediana dos leitores na análise principal; variação inter-leitor como contexto). Medir a concordância de cada radiologista com os restantes como linha de base humana.
4. Mostrar intervalos de confiança; incluir baselines (resposta mais comum por campo; repetição do mesmo prompt para consistência).
5. Falhas e `não avaliável` não contam como acerto nem erro; reportar cobertura à parte.

## 8. Contribuição esperada

- Perfil de concordância por campo (o que o LLM descreve bem e mal).
- Comparação com a variabilidade entre radiologistas.
- Efeito do número de cortes.
- Fiabilidade do relatório (não avaliável, contradições, achados inventados, consistência entre repetições).

Não encontrei, numa pesquisa curta, um estudo que compare campo a campo o relatório de um LLM geral com o SR do LIDC em TC pulmonar; verificar com pesquisa dirigida antes de afirmar novidade.

## 9. Literatura inicial (a confirmar nos artigos)

- Comparação de MLLMs na deteção de nódulos em radiografias de tórax (PMC12887855): Gemini 2.0 Pro Experimental com accuracy 0.602 e sensibilidade 0.316.
- Large multimodal models em TC pulmonar (VIEW, 2025; doi.org/10.1002/VIW.20250077): AUC 0.61 com um corte para Gemini e Claude; accuracy desceu com mais cortes ou história clínica; houve dados inventados.
- GPTRadScore (arXiv 2403.05680; PMC12478437): GPT-4V e Gemini Pro Vision melhores que outros modelos, mas localização de 17.1% e 14%.
- GPT-4o em nódulos pulmonares longitudinais (Cell Reports Medicine, 2025): accuracy 0.88 para malignidade e ICC 0.91 para tamanho; desenho longitudinal, não diretamente comparável.
- RSNA Radiology, radiologistas vs GPT-4V e Gemini Pro Vision (10.1148/radiol.240273): resultados ainda por ler.

Por pesquisar: revisão sistemática de LLMs multimodais em imagiologia; literatura sobre encaminhamento/triagem de exames.

## 10. Próximos passos

1. Inspecionar SR e SEG descarregados (só leitura, sem chamadas ao modelo).
2. Fixar campos, regra de leitores e regra de alinhamento antes de ver resultados.
3. Ler os artigos da secção 9.
4. Só depois: novos prompts, schema de saída e avaliador.
5. Fase posterior: planeador.
