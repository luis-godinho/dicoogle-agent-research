# Experiência LIDC: relatório de um LLM geral vs referências estruturadas

Estado: desenho provisório revisto em 2026-11-21; não implementado. Decisões condicionadas à auditoria dos dados locais. Prompts para agentes devem ser escritos em inglês.

## 1. Objetivo e âmbito

Avaliar se um LLM multimodal geral produz descrições estruturadas de achados a partir de imagens sem anotações visíveis, comparando-as com referências escondidas. O planeador de encaminhamento para modelos especializados fica para uma fase posterior. Esta fase não demonstra utilidade clínica nem eficácia de encaminhamento.

LIDC-IDRI é o primeiro candidato para validar o método, não um compromisso de dataset único. DICOM SR é um formato possível de referência, não um requisito universal nem uma garantia de diagnóstico confirmado. Cada dataset terá um mapeamento documentado para o esquema comum, com campos indisponíveis explicitamente assinalados.

## 2. Preservar o piloto anterior

Run `exp-lidc_idri_pilot-60994d3b`: dois pacientes, quatro pedidos, três sucessos e uma falha `connector_error:http_5xx`. Classificação benigno/maligno com e sem overlay, avaliada contra labels de paciente.

Só um par está completo. Balanced accuracy e AUROC de 1.0 no subset não validam desempenho nodular. LIDC-IDRI-0072 tem diagnóstico maligno de paciente e benigno para Nodule 1; não existe crosswalk clínico autorizado para as SEG identificadas como Nodule 1.

Preservar inputs, respostas, falha, accounting e métricas como evidência histórica. Não reinterpretar ou sobrescrever a execução.

## 3. Imagens, SEG e seleção de cortes

- CT renderizada sem overlays: input do modelo.
- SEG: seleção experimental dos cortes e apoio ao alinhamento espacial, quando verificado.
- SR/source annotations: referência escondida, nunca input do modelo.
- Diagnóstico de paciente: separado de características e de diagnóstico confirmado de lesão.

Seleção pela SEG é uma condição assistida por referência: avalia descrição em cortes relevantes, não deteção autónoma num estudo completo. Não constitui um limite superior matemático garantido.

Amostragem uniforme sem SEG e número de cortes são análises adicionais possíveis, não requisitos para implementar já. Não há comparação com/sem overlay no desenho principal. Custos, limites da API e desempenho com diferentes volumes de imagem devem ser avaliados separadamente.

## 4. Proveniência e anotações individuais

Auditar a cadeia source annotation -> grupo SR -> SEG/segmento -> CT -> lesão física. Não assumir que os SR estão descarregados.

Distinguir objeto SEG, segmento, anotação e lesão. Partilhar o rótulo Nodule 1 ou ter SOPInstanceUIDs distintos não prova identidade de lesão nem leitores diferentes. Geometria pode apoiar uma hipótese, mas não substitui proveniência explícita.

Há potencialmente várias anotações por lesão; não assumir quatro disponíveis em cada caso. Verificar identificadores de leitor/sessão e se são estáveis entre casos. Na ausência de identificação, escrever que a identidade não é recuperável dos ficheiros inspecionados, sem concluir que foi deliberadamente removida.

## 5. Núcleo de campos e extensões

| Campo | Papel e condição |
|---|---|
| Lista/contagem de achados | Núcleo; avaliar apenas no âmbito visual e de referência estabelecido |
| Tipo de achado | Núcleo; vocabulário compatível com a referência |
| Localização por painel/corte e posição aproximada | Núcleo; coordenadas rastreáveis; anatomia adicional apenas quando avaliável |
| Atenuação | Núcleo proposto: sólido, parcialmente sólido, não sólido; mapeamento pré-definido por dataset |
| Tamanho | Condicional: escala fornecida ao modelo e definição de medida compatível |
| Margem | Extensão; não confundir nitidez com espiculação/lobulação |
| Espiculação/lobulação | Extensões; verificar proveniência e eventuais correções de codificação |
| Calcificação | Extensão nominal, não ordinal |
| Suspeita de malignidade | Exploratória; avaliação subjetiva, não diagnóstico confirmado |

Subtlety, sphericity e internal structure ficam fora do núcleo inicial. Internal structure não deve ser tratada automaticamente como ordinal. Códigos numéricos não implicam ordem natural.

Não presumir que milímetros são recuperáveis de uma montagem redimensionada. Distinguir metadados da CT disponíveis ao avaliador da informação efetivamente fornecida ao modelo.

Cada campo admite not_assessable, motivo e evidência visual. Esquema final só após auditoria.

## 6. Regra das referências

Análise principal proposta:
1. Preservar todas as anotações ligadas com evidência à lesão.
2. Comparar a previsão com cada anotação disponível.
3. Resumir dentro da lesão, dando peso igual a cada lesão na agregação.
4. Separar casos com uma e várias anotações; tratar dependência entre lesões do mesmo paciente na incerteza estatística.

Reportar desacordo entre anotações. Não inventar leitores persistentes ou kappas por pares de leitores sem identificação válida.

Análise secundária agregada:
- Ordinais: mediana apenas quando definir uma categoria sem ambiguidade.
- Nominais: moda única.
- Empates/medianas entre categorias: referência ambígua, sem arredondamento escolhido após ver a previsão.

Não considerar automaticamente correta qualquer previsão dentro do intervalo dos leitores. A referência agregada não é diagnóstico verdadeiro.

## 7. Emparelhamento, omissões e achados extra

Definir antes dos resultados uma correspondência espacial um-para-um com critério de admissibilidade e limiar, não apenas escolher sempre o vizinho mais próximo. Usar coordenadas/cortes e outras evidências disponíveis; correspondências ambíguas ficam explícitas.

Comparar atributos nos pares admissíveis, mas reportar também:
- lesões de referência sem previsão (omissões);
- previsões sem correspondência;
- duplicações;
- erro de contagem, quando a referência suportar contagem no âmbito avaliado.

Uma previsão extra não desaparece por se escolher a melhor correspondência. Contudo, não anotado não significa ausente: chamar falso positivo confirmado apenas quando a referência/revisão o suportar. Separar achados extra não verificáveis.

A lógica sensibilidade + falsos positivos por exame tem precedente no LUNA16; não afirmar implementação equivalente ao protocolo FROC sem satisfazer os seus requisitos. Não estimar especificidade de exemplos apenas positivos ou sem unidade negativa definida.

## 8. Métricas

| Tipo | Avaliação proposta |
|---|---|
| Ordinal | Concordância exata, matriz de confusão, distância absoluta média em categorias; ±1 nível como análise secundária explícita |
| Nominal | Concordância exata e matriz de confusão |
| Contínuo | Erro absoluto, viés; Bland-Altman e ICC com especificação adequada quando amostra/desenho permitirem |
| Deteção | Sensibilidade, omissões e previsões sem correspondência, respeitando completude da referência |

Distâncias entre categorias ordinais são uma convenção a explicitar. Kappa ponderado linear é complemento contra referência agregada ordinal definida, sujeito a amostra/distribuição adequadas. Spearman mede associação e não substitui concordância. Não concatenar pares LLM-anotação como observações independentes de dois leitores fixos.

Apresentar intervalos de confiança apropriados à dependência por paciente, denominadores e baselines fixados sem usar resultados do teste. Repetições medem consistência, não substituem uma baseline preditiva.

Reportar em conjunto: concordância entre respostas avaliáveis, cobertura por campo, abstinências, falhas técnicas e referências sem resposta utilizável. Não ocultar omissões em scores apenas condicionais ao sucesso.

## 9. Literatura de apoio

- Vanbelle, Hernandez Engelhart e Blix (2024), A comprehensive guide to study the agreement and reliability of multi-observer ordinal data. DOI: 10.1186/s12874-024-02431-y. Base para concordância ordinal, ponderações e desenho multi-observador.
- Pedrosa et al., LNDb: A Lung Nodule Database on Computed Tomography. arXiv:1911.08434. Precedente para concordância de características nodulares e distinção entre campos nominais/ordinais; explicita limitações de identidade dos leitores do LIDC.
- Zhu et al., How Well Do Multi-modal LLMs Interpret CT Scans? An Auto-Evaluation Framework for Analyses. arXiv:2403.05680. GPTRadScore: comparação de descrições por parte do corpo, localização e tipo; não resolve por si só deteção exaustiva ou penalização de achados extra.
- Mao et al. (2025), Assessments of lung nodules by an artificial intelligence chatbot using longitudinal CT images. DOI: 10.1016/j.xcrm.2025.101988. Precedente para atributos, ICC e Bland-Altman. Fornece coordenadas/tamanho inicial para tracking e usa preparação com exemplos: não equivale ao nosso cenário sem pistas. Não desenhou experiência específica de múltiplos nódulos.
- Huang et al. (2025), Unveiling large multimodal models in pulmonary CT: A comparative assessment of generative AI performance in lung cancer diagnostics. DOI: 10.1002/VIW.20250077. Descrições por dimensão, repetições e discrepâncias com relatórios humanos.
- Khovanova et al. (2026), A comparative accuracy study of multimodal LLMs, VLM and agent-based framework for pulmonary nodule detection on chest radiographs. DOI: 10.3389/fdgth.2025.1674835. Começou com 100 exames, avaliou 83 após exclusões técnicas; alucinações dos relatórios não afetaram o endpoint binário. Não é comparação detalhada de relatórios.
- Suh et al. (2024), Comparing Diagnostic Accuracy of Radiologists versus GPT-4V and Gemini Pro Vision Using Image Inputs from Diagnosis Please Cases. DOI: 10.1148/radiol.240273. Diagnóstico diferencial com repetições; critérios de acerto não equivalem a qualidade de descrição.
- LUNA16, protocolo de avaliação: https://luna16.grand-challenge.org/Evaluation/ . Inspiração para avaliar deteção e achados extra, não protocolo adotado automaticamente.

Os cinco PDFs fornecidos foram consultados através do texto extraído; isso não equivale a validação integral de todos os suplementos e resultados. Confirmar tabelas, condições experimentais e referências antes da submissão. Não há ainda uma revisão sistemática nem uma alegação demonstrada de novidade.

## 10. Auditoria local e próximos passos

Executar primeiro uma inspeção em inglês, só de leitura, dos dois pacientes: CT/SEG/SR, anotações-fonte e manifestos locais. Sem inferência, rede, downloads, instalações, alterações ao pipeline, evaluate/report, staging ou commits. No máximo um novo relatório Markdown fora de artifacts/runs e artifacts/results; não sobrescrever nada.

O relatório deve incluir inventário, atributos crus/códigos/unidades, caminhos na árvore SR, ligações verificadas, identidade de leitores recuperável ou não, discrepância clínica, completude da referência e matriz de viabilidade dos campos. Separar verificado, inferido e não resolvido; continuar verificações independentes perante bloqueios. Não interpretar imagens clinicamente.

Depois: fixar esquema, correspondência, regra de referência e métricas antes de novas respostas do modelo. Só então alterar prompts/avaliador e planear tamanho amostral e eventual segundo dataset. Esta revisão documental não autoriza essas ações.
