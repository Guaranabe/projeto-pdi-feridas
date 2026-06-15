# projeto-pdi-feridas
[Caso deseje abrir direto no Google Colab](https://colab.research.google.com/drive/1WSP4hDdEL6eX72d2FSijlH8I4yY7wolU?usp=sharing)

Este projeto implementa um pipeline de Processamento Digital de Imagens para identificação aproximada de regiões de feridas diabéticas. O sistema utiliza uma CNN baseada em MobileNetV2 para classificar patches como pele saudável ou ferida e, posteriormente, aplica janela deslizante, limiarização, morfologia matemática e classificação por cor em HSV para destacar tecidos aparentes da lesão.
O projeto foi desenvolvido para execução no Google Colab. O link do dataset utilizado se encontra na documentação e no próprio código. O projeto foi feito para ser executado célula a célula.

## Como executar

1. Abra o arquivo `DFU_Segmentacao_Colab.ipynb` no Google Colab.
2. Execute as células em ordem.
3. Quando solicitado, envie o arquivo `feridadiabetes.zip`.
4. O notebook irá extrair o dataset, treinar o modelo e gerar os resultados.
5. Quando solicitado novamente, caso deseje, insira uma imagem de ferida diabética qualquer.
