# Relatório Técnico — Segmentação de Feridas Diabéticas

## 1. Introdução

Este projeto apresenta um pipeline de Processamento Digital de Imagens aplicado à análise de imagens de feridas relacionadas ao pé diabético. O objetivo é identificar automaticamente a região provável da ferida em uma imagem e, em seguida, destacar visualmente diferentes tecidos aparentes presentes na lesão.

O sistema foi desenvolvido para execução no Google Colab, utilizando Python, TensorFlow/Keras, OpenCV, NumPy, Pandas e Matplotlib.

## 2. Dataset

Foi utilizado o dataset Diabetic Foot Ulcer (https://www.kaggle.com/datasets/laithjj/diabetic-foot-ulcer-dfu), composto por imagens de pés diabéticos e patches organizados em duas classes principais:

* `Abnormal(Ulcer)`: patches contendo regiões de ferida ou úlcera;
* `Normal(Healthy skin)`: patches contendo pele saudável.

Como o dataset não possui máscaras pixel a pixel para segmentação semântica, não foi possível treinar diretamente um modelo do tipo U-Net ou DeepLab para segmentação supervisionada. Sendo assim, foi adotada uma abordagem híbrida, combinando Deep Learning para classificação de patches e técnicas clássicas de PDI para pós-processamento e classificação visual dos tecidos.

## 3. Pipeline

O pipeline final foi dividido nas seguintes etapas:

1. carregamento e extração do dataset;
2. preparação dos patches de treino e validação;
3. treinamento de um classificador binário;
4. aplicação do modelo em imagens completas por janela deslizante;
5. geração de um mapa de probabilidade da ferida;
6. criação de uma máscara binária da região provável da ferida;
7. pós-processamento morfológico da máscara;
8. classificação aproximada dos tecidos aparentes por cor;
9. visualização dos resultados.

Esse é o fluxo de funcionamento do código. Em duas instâncias, ele solicita input do usuário: ao início, para o carregamento do dataset mencionado anteriormente; e ao fim, para utilização de uma imagem que o usuário deseja testar.


## 4. Classificação com Deep Learning

Para a etapa de classificação dos patches, foi utilizada a arquitetura MobileNetV2 com transferência de aprendizado. Essa escolha foi feita porque a MobileNetV2 é uma rede convolucional leve, eficiente e adequada para aplicações em ambientes com recursos limitados, como Google Colab ou sistemas futuros baseados em dispositivos móveis.

A rede foi inicializada com pesos pré-treinados e adaptada para uma tarefa binária:

* classe 0: pele saudável;
* classe 1: ferida/úlcera.

A saída do modelo é uma probabilidade entre 0 e 1, indicando a chance de um patch conter região de ferida. Foi utilizada função de perda `binary_crossentropy`, adequada para classificação binária, e otimizador Adam.

Também foi usada uma seed fixa para reduzir a aleatoriedade na divisão dos dados, inicialização das camadas treináveis e embaralhamento das imagens. Isso contribui para maior reprodutibilidade dos resultados.

## 5. Segmentação aproximada da ferida

Como o dataset não fornece máscaras reais da ferida, a segmentação foi feita de forma aproximada. Para isso, o classificador treinado foi aplicado em imagens completas por meio de uma técnica de janela deslizante.

A imagem é percorrida por janelas de tamanho `224x224` pixels, compatíveis com a entrada da MobileNetV2. Em cada janela, o modelo estima a probabilidade de presença de ferida. As probabilidades obtidas são combinadas para formar um mapa de calor, no qual regiões mais claras indicam maior probabilidade de conter ferida.

Foi utilizado `stride=32` para melhorar a resolução espacial do mapa de probabilidade. Esse valor gera maior sobreposição entre as janelas e tende a produzir uma segmentação visualmente mais detalhada, embora aumente o tempo de processamento.

## 6. Pós-processamento da máscara

Após a geração do mapa de calor, foi aplicada uma etapa de pós-processamento para criar a máscara binária da ferida. As principais escolhas tomadas foram:

* suavização do mapa com filtro Gaussiano;
* limiarização geralmente com `threshold=0.80`;
* operações morfológicas de abertura e fechamento;
* remoção de componentes pequenos;
* manutenção da maior região conectada.

O threshold escolhido é razoavelmente alto, e foi a opção devido a iterações apresentarem melhores resultados para o código atual.

## 7. Classificação dos tecidos aparentes

Após a obtenção da máscara da ferida, a classificação dos tecidos foi feita apenas dentro da região segmentada. Essa etapa não utiliza Deep Learning, mas sim regras baseadas em cor no espaço HSV e em características RGB.

As classes finais utilizadas foram:

* `0`: fora da ferida;
* `1`: escara ou necrose escura;
* `2`: fibrina ou slough;
* `3`: tecido de granulação;
* `4`: epitelização;
* `5`: maceração;
* `6`: região incerta dentro da ferida.

A escara/necrose foi associada a regiões muito escuras ou marrom-escuras. A fibrina/slough foi associada a tons amarelados, bege ou creme. A granulação foi associada a regiões avermelhadas com predominância do canal vermelho. A epitelização foi associada a tons rosados claros próximos à borda da ferida. A maceração foi associada a regiões claras, pouco saturadas e esbranquiçadas, também próximas à borda.

A classe “incerto” foi incluída para evitar que pixels ambíguos fossem forçados a uma classe específica. Essa escolha torna o resultado mais cauteloso, especialmente considerando as variações de iluminação, foco, posição, cor de pele e qualidade das imagens.

## 8. Limitações

A principal limitação do projeto é a ausência de máscaras pixel a pixel no dataset utilizado. Por isso, a segmentação da ferida não é uma segmentação supervisionada real, mas uma aproximação gerada a partir de um classificador de patches.

Além disso, a classificação dos tecidos aparentes depende fortemente da cor da imagem. Assim, o resultado pode ser afetado por fatores como:

* iluminação inadequada;
* sombras;
* uso de flash;
* baixa resolução;
* pele com diferentes tonalidades;
* presença de curativos;
* fundo parecido com a pele;
* imagens desfocadas.

O sistema não possui finalidade diagnóstica e não deve ser usado para tomada de decisão médica. Trata-se de uma aplicação acadêmica dos conceitos aprendidos em PDI, com auxílio de técnicas de Deep Learning.

## 9. Conclusão

O projeto demonstrou a viabilidade de combinar técnicas de Deep Learning e Processamento Digital de Imagens para análise aproximada de feridas diabéticas. A MobileNetV2 foi utilizada para identificar regiões prováveis de ferida em patches, enquanto a técnica de janela deslizante permitiu transformar essa classificação em uma máscara aproximada sobre a imagem completa.

A etapa de pós-processamento morfológico melhorou a qualidade da máscara, reduzindo ruídos e falsos positivos. Por fim, a classificação por cor em HSV permitiu destacar visualmente diferentes tecidos aparentes da ferida, como necrose, fibrina, granulação, epitelização e maceração. Ainda assim, nota-se a complexidade da classificação e os vários obstáculos a serem vencidos para a realização de um projeto viável para uso real.

Apesar das limitações, o pipeline atende ao objetivo acadêmico do projeto, fornecendo uma solução funcional, interpretável e executável no Google Colab.

Em futuras iterações, deseja-se melhorar a caracterização das classes, e se possível, obter ou criar uma máscara feita por um especialista e assim utilizá-la para segmentação.
