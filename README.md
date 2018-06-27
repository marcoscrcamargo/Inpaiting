<h1 align="center">Inpainting</h1>


## Autores

| [![victorxjoey](https://avatars1.githubusercontent.com/u/13484548?s=230&v=4)](https://github.com/VictorXjoeY/) |               [![marcoscrcamargo](https://avatars0.githubusercontent.com/u/13886241?s=230&v=4)](https://github.com/marcoscrcamargo/) |
|:-----------------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------:|
|[Victor Luiz Roquete Forbes](https://github.com/VictorXjoeY/)|[Marcos Cesar Ribeiro de Camargo](https://github.com/marcoscrcamargo/)|
| 9293394 | 9278045|

# Introdução
*Inpainting* é o processo de reconstrução digital de partes perdidas ou deterioradas de imagens e vídeos, também conhecida como interpolação de imagens e vídeos. Refere-se à aplicação de algoritmos sofisticados para substituir partes perdidas ou corrompidas da imagem (principalmente pequenas regiões ou para remover pequenos defeitos).

Nesse projeto estudamos e implementamos técnicas de *inpainting* para a remoção automática de rabiscos inseridos artificialmente em imagens. Para realizarmos a detecção automática da região que devemos fazer *inpainting* usamos do fato de que os rabiscos são feitos com cores contrastantes que ocorrem com alta frequência nas imagens.

Aplicamos também os métodos utilizados para a remoção de objetos em imagens. Para isso é necessário desenhar em cima do objeto a ser removido com um pincel duro com uma cor contrastante como, por exemplo, o vermelho (255, 0, 0).

# Conjunto de imagens
Parte do conjunto de imagens utilizado é apresentado abaixo.

## Imagens Originais
As imagens abaixo estão em sua forma original.

|<img src="./Project/images/original/dogo2.bmp"   width="200px" alt="dogo2"/>|<img src="./Project/images/original/horse_car.bmp"   height="200px" alt="horse_car"/>|<img src="./Project/images/original/forbes.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/original/momo_fino.bmp"   width="200px" alt="momo_fino"/>|
|------------|------------|------------|------------|
| Cachorro ([retirada da internet](https://pbs.twimg.com/profile_images/948761950363664385/Fpr2Oz35_400x400.jpg)) | Texto em foto (retirada de um [artigo](http://www.inf.ufrgs.br/~oliveira/pubs_files/inpainting.pdf)) | Forbes | Professor Moacir |

## Imagens Deterioradas
As imagens abaixo foram rabiscadas artificialmente. A única imagem que não inserimos rabiscos foi a segunda imagem, que foi retirada de um [artigo](http://www.inf.ufrgs.br/~oliveira/pubs_files/inpainting.pdf).

|<img src="./Project/images/deteriorated/dogo2.bmp"   width="200px" alt="dogo2"/>|<img src="./Project/images/deteriorated/horse_car.bmp"   height="200px" alt="horse_car"/>|<img src="./Project/images/deteriorated/forbes.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/deteriorated/momo_fino.bmp"   width="200px" alt="momo_fino"/>|
|------------|------------|------------|------------|
| Cachorro ([retirada da internet](https://pbs.twimg.com/profile_images/948761950363664385/Fpr2Oz35_400x400.jpg)) | Texto em foto (retirada de um [artigo](http://www.inf.ufrgs.br/~oliveira/pubs_files/inpainting.pdf)) | Forbes | Professor Moacir |

# Algoritmos de extração da máscara

O primeiro passo para realizar *inpainting* é definir a região deteriorada. Para isso implementamos dois métodos simples para extrair a máscara automaticamente. Para essa etapa do trabalho queremos realizar *inpainting* em cores específicas, ou seja, assumimos como sendo parte da região deteriorada todos os *pixels* que possuem certas cortes determinadas pelos métodos descritos a seguir.

O primeiro passo de ambos os métodos é calcular o número de ocorrências de cada tripla de cor RGB. Optamos por ignorar cores muito próximas do branco absoluto (255, 255, 255) pois alguns dos experimentos foram feitos com imagens com excesso de luz. Ignoramos então as cores que possuem valor maior ou igual a 250 nos três canais.

Vale notar que os métodos podem ser melhorados para que a máscara obtida represente não só algumas cores pré-determinadas pelos métodos, mas que represente também a "penumbra" que muitas ferramentas de edição inserem nas bordas dos traços. Para isso podemos extrair a máscara usando os métodos abaixo e depois preencher todos os *pixels* não preenchidos adjacentes a um ou mais *pixels* preenchidos que possuem um tom parecido, utilizando uma certa medida de distância. Para esse projeto tentamos usar uma medida de distância usando os canais H e S do espaço de cores HSV, mas, como não obtivemos muita precisão, optamos por usar apenas riscos "duros" no projeto para focar na parte principal: o Inpainting.

|<img src="./Project/images/deteriorated/momo.bmp"   width="200px" alt="momo"/>|<img src="./Project/images/masks/momo.bmp"   width="200px" alt="momo"/>|
|:-----------------------------------:|:-----------------------------------:|
| Foto deteriorada | Máscara extraída |

## *Most Frequent*
Nesse método assumimos que apenas uma cor deve ser removida: a mais frequente. Esse método funciona muito bem quando existe rabiscos de apenas uma cor.

## *Minimum Frequency*

Nesse método definimos um *threshold* e assumimos que todas as cores que ocorrem mais vezes do que esse *threshold* fazem parte da região de *inpainting*. Para essa etapa definimos o *threshold* como sendo 1% dos *pixels* da imagem, ou seja, se alguma cor ocorrer em mais do que 1% da imagem, ela é considerada "rabisco". Esse método não funciona tão bem quando existe rabiscos de apenas uma cor, mas é um método necessário para remover rabiscos de diferentes cores.

## *Red*

Nesse método a máscara será composta por todos os pixels vermelhos (255, 0, 0). Esse método é importante para melhorar a precisão da extração da máscara para a aplicação extra do projeto de remover objetos indesejados. Para isso basta o usuário pintar de vermelho (255, 0, 0) os objetos que deseja remover da imagem.

# Algoritmos de *Inpainting*
## Gerchberg Papoulis
O algoritmo Gerchberg-Papoulis é um algoritmo de *inpaiting* por difusão que funciona por meio de cortes nas frequencias obtidas pela Transformada Discreta de Fourier (DFT), zerando parte das frequências das imagens.

Considerando uma Máscara **M** que possui valor 0 nos locais em que a imagem é conhecida e 255 nas regiões deterioradas, o algoritmo com **T** iterações é aplicado da seguinte maneira:

1. *g_0* = Imagem inicial
2. Obtenção da DFT de M.
3. Para cada iteração k = 1, ..., T
	+  *G_k* = DFT de *g_{k-1}*
	+  Filtra *G_k* zerando coeficientes das frequências relativos a:
		+  *G_k* >= 0.9 * max(*M*)
		+  *G_k* <= 0.01 * max(*G_k*)
	+  Convolução de *G_k* com um filtro de média *k x k*.
	+  *g_k* = IDFT de *G_k*
	+  Normalização de *G_k* entre 0 e 255.
	+  Inserção dos pixels de *gk* somente na região referente a máscara.
		+  *g_k* = (1 - M/255) * *g_0* + (M/255) * *g_k*

Ao final do processo é obtida a imagem *G_k* restaurada.

Abaixo estão alguns dos resultados do algoritmo.

|<img src="./Project/images/inpainted/Gerchberg Papoulis/dogo2.bmp"   width="200px" alt="dogo2"/>|<img src="./Project/images/inpainted/Gerchberg Papoulis/horse_car.bmp"   height="200px" alt="horse_car"/>|<img src="./Project/images/inpainted/Gerchberg Papoulis/forbes.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/inpainted/Gerchberg Papoulis/momo_fino.bmp"   width="200px" alt="momo_fino"/>|
|------------|------------|------------|------------|
| Cachorro (retirada da internet) | Texto em foto (retirada de um artigo) | Forbes | Professor Moacir |

## *Inpainting* por exemplos
Os algoritmos de *Inpainting* por exemplos utilizados consistem em substituir cada *pixel* deteriorado *Pd* por um *pixel* não deteriorado *P* cuja janela *K*x*K* centrada em *P* maximiza uma certa medida de similaridade em relação a janela *K*x*K* centrada em *Pd*.

O *K* é definido automaticamente levando em consideração a "grossura" do rabisco da seguinte forma: Para cada *pixel* deteriorado *Pd* calcula-se sua distância de Manhattan para o pixel não-deteriorado mais próximo usando uma *Multi-Source Breadth-First Search*. Ao recuperar o máximo de todos esses valores, multiplica-lo por 2 e somar 3, obtemos um valor para *K* grande o suficiente para a região deteriorada nunca conter completamente uma janela *K*x*K*.

A medida de distância utilizada foi similar ao RMSE, mas calculado apenas entre *pixels* não-deteriorados. Vale dizer que para todo o projeto assumimos que os *pixels* fora da imagem são pretos (0, 0, 0).

### *Brute Force*
Nesse algoritmo a busca pelo *pixel* *P* é feita em toda a imagem. Seu tempo de execução é altíssimo e, portanto, apenas conseguimos rodar para as imagens dogo1.bmp (100x100), dogo2.bmp (400x400), momo_fino.bmp (280x280) e momo.bmp (280x280).

|<img src="./Project/images/inpainted/Brute Force/dogo1.bmp"   width="200px" alt="dogo1"/>|<img src="./Project/images/inpainted/Brute Force/dogo2.bmp"   height="200px" alt="dogo2"/>|<img src="./Project/images/inpainted/Brute Force/momo_fino.bmp"   width="200px" alt="momo_fino"/>|<img src="./Project/images/inpainted/Brute Force/momo.bmp"   width="200px" alt="momo"/>|
|------------|------------|------------|------------|
| Cachorro 100x100 reconstruído | Cachorro 400x400 reconstruído | Moacir com rabiscos finos reconstruído | Moacir com rabiscos grossos reconstruído |

### *Local Brute Force*
Nesse algoritmo fazemos a suposição de que as janelas mais similares não estão muito longe da região deteriorada, portanto a busca pelo *pixel* *P* é feita apenas em uma região 101x101 centrada em *Pd*. Isso permite que façamos *inpainting* em imagens maiores em tempo hábil, como a imagem abaixo.

|<img src="./Project/images/deteriorated/forbes.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/inpainted/Local Brute Force/forbes.bmp"   width="200px" alt="forbes"/>|
|------------|------------|
| Forbes 961x1280 deteriorado | Forbes 961x1280 reconstruído |

### *Local Dynamic Brute Force*
Aprimorando um pouco a ideia do Brute Force Local, percebemos que atribuir a média entre os *pixels* cuja janela *K*x*K* mais se assemelham a janela do *pixel* *Pd* reduz o RMSE e resulta, em geral, em restaurações mais suaves. Usamos os 5 mais semelhantes para imagens menores e os 10 mais semelhantes para imagens maiores.

Além disso, ao começarmos a tentar remover rabiscos mais grossos ou objetos maiores observamos que poderíamos usar um valor de *K* dinâmico, ou seja, um valor de *K* para cada *pixel* deteriorado *Pd* com o objetivo de obter janelas mais representativas e reduzir o tempo de execução. Esse método se mostrou especialmente útil em máscaras mais grossas, ou seja, máscaras que produzem um valor de *K* elevado nos outros métodos. Seu tempo de execução é mais baixo pois o *K* escolhido para as bordas é menor do que o *K* escolhido para o centro de regiões deterioradas.

|<img src="./Project/images/other/forbes_profile.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/deteriorated/forbes_profile.bmp"   width="200px" alt="forbes"/>|
|<img src="./Project/images/masks/forbes_profile.bmp"   width="200px" alt="forbes"/>|<img src="./Project/images/inpainted/Local Dynamic Brute Force/forbes_profile.bmp"   width="200px" alt="forbes"/>|
|------------|------------|------------|------------|
| Forbes-Perfil 934x1280 original | Forbes-Perfil 934x1280 "deteriorado" | Forbes-Perfil 934x1280 máscara | Forbes-Perfil 934x1280 restaurado |

### *Smart Brute Force*

Nesse algoritmo rodamos uma *Depth-First Search* (DFS) a partir de cada componente conexa de pixels deteriorados. Para o primeiro pixel deteriorado *Pd* de uma componente fazemos a busca pelas 50 janelas *K*x*K* mais similares em uma região 101x101 centrada em *Pd* e guardamos em uma lista de candidatos. Passamos essa lista para os vizinhos de *Pd* de tal forma que os vizinhos precisem apenas calcular a distância para 50 janelas na maior parte das vezes. Quando a janela mais similar ao *pixel* atual não for tão similar (sua distância é maior que um *threshold* que definimos como sendo 5.0 para a maioria das imagens), buscamos uma lista com os 50 melhores candidatos novamente. Por fim atribuímos a cada *pixel* deteriorado a média dos 5 *pixels* cujas janelas são as mais similares dentre os candidatos.

A suposição feita para o desenvolvimento desse algoritmo se deve ao fato de que *pixels* deteriorados vizinhos devem ser similares entre si e, portanto, similares a uma mesma lista de candidatos.

Podemos ver pela imagem *horse_car.bmp* que usar o *pixel* cuja janela *K*x*K* possui distância mínima não é sempre a melhor escolha. A média entre os 5 melhores candidatos resulta em um *inpainting* mais suave, removendo parte do ruído produzido pelos outros métodos de força bruta descritos.

|<img src="./Project/images/deteriorated/horse_car.bmp"   width="200px" alt="horse_car_deteriorated"/>|
<img src="./Project/images/masks/horse_car.bmp"   width="200px" alt="horse_car_mask"/>|
<img src="./Project/images/inpainted/Local Brute Force/horse_car.bmp"   width="200px" alt="horse_car_local"/>|
<img src="./Project/images/inpainted/Smart Brute Force/horse_car.bmp"   width="200px" alt="horse_car_smart"/>|
|------------|------------|------------|------------|
| Imagem deteriorada | Máscara | Local Brute Force | Smart Brute Force |

# Resultados

A avaliação dos resultados foi feita visualmente e pela raiz do erro quadrático médio (RMSE), calculado apenas nos pixels da região deteriorada (pixels da máscara). Além disso foram geradas imagens representando a diferença entre as imagens originais e as imagens restauradas.

## Remoção de rabiscos em imagens

Em geral observamos que os melhores resultados vieram do algoritmo *Smart Brute Force* enquanto os piores resultados vieram do *Gerchberg Papoulis*, visto que esse gera regiões visivelmente mais borradas (apesar de possuir o tempo de execução mais baixo) principalmente para imagens com vários detalhes como rostos.

A tabela abaixo sumariza os resultados obtidos na remoção dos rabiscos.

| Brute Force | RMSE | Tempo|
| :---: | :---: | :---: |
| dogo1.bmp (100x100) | 08.340 | 00m07s |
| dogo2.bmp (400x400) | 13.222 | 32m19s |
| momo.bmp (280x280) | 23.721 | 30m24s |
| momo_fino.bmp (280x280) | 13.735 | 06m50s |

Em seguida apresentamos resultados para algumas imagens com comparativos visuais e métricos, além do tempo de execução de cada algoritmo.

### Professor Moacir (desenho com bordas grossas)

A foto do professor Moacir com o desenho de bordas grossas (momo.bmp) tem dimensões 280x280.

Comparação das imagens:

|<img src="./Project/images/inpainted/Smart Brute Force/momo.bmp"   width="200px" alt="momo_inpainted_brute"/>|
<img src="./Project/images/difference/Smart Brute Force/momo.bmp"   width="200px" alt="momo_diff_brute"/>|
<img src="./Project/images/inpainted/Gerchberg Papoulis/momo.bmp"   width="200px" alt="momo_inpainted_gerchberg"/>|
<img src="./Project/images/difference/Gerchberg Papoulis/momo.bmp"   width="200px" alt="momo_diff_gerchberg"/>|
|------------|------------|------------|------------|
| Smart Brute Force | Imagem da diferença Smart Brute Force | Gerchberg Papoulis | Imagem da diferença Gerchberg Papoulis |

Comparação do RMSE e tempo de execução para cada algoritmo:

| Algoritmo | RMSE | Tempo de execução |
|-----------|------|-------------------|
|Gerchberg Papoulis| 45.094 |00m05s|
|Brute Force|23.721|30m24s|
|Local Brute Force|23.456|03m23s|
|Smart Brute Force|21.352|01m57s|

### Professor Moacir (desenho com bordas finas)

A foto do professor Moacir com o desenho de bordas finas (momo_fino.bmp) tem dimensões 280x280.

Comparação das imagens:

|<img src="./Project/images/inpainted/Smart Brute Force/momo_fino.bmp"   width="200px" alt="momo_fino_inpainted_brute"/>|
<img src="./Project/images/difference/Smart Brute Force/momo_fino.bmp"   width="200px" alt="momo_fino_diff_brute"/>|
<img src="./Project/images/inpainted/Gerchberg Papoulis/momo_fino.bmp"   width="200px" alt="momo_fino_inapinted_gerchberg"/>|
<img src="./Project/images/difference/Gerchberg Papoulis/momo_fino.bmp"   width="200px" alt="momo_fino_diff_gerchberg"/>|
|------------|------------|------------|------------|
| Smart Brute Force | Imagem da diferença Smart Brute Force | Gerchberg Papoulis | Imagem da diferença Gerchberg Papoulis |

Comparação do RMSE e tempo de execução para cada algoritmo:

| Algoritmo | RMSE | Tempo de execução |
|-----------|------|-------------------|
|Gerchberg Papoulis|30.238|00m05s|
|Brute Force|13.735|06m50s|
|Local Brute Force|14.350|00m47s|
|Smart Brute Force|12.567|00m24s|


### Cachorro

A imagem do cachorro retirada da internet (dogo2.bmp) tem dimensões 400x400.

Comparação das imagens:

|<img src="./Project/images/inpainted/Smart Brute Force/dogo2.bmp"   width="200px" alt="dogo2_inpainted_brute"/>|
<img src="./Project/images/difference/Smart Brute Force/dogo2.bmp"   width="200px" alt="dogo2_diff_brute"/>|
<img src="./Project/images/inpainted/Gerchberg Papoulis/dogo2.bmp"   width="200px" alt="dogo2_inapinted_gerchberg"/>|
<img src="./Project/images/difference/Gerchberg Papoulis/dogo2.bmp"   width="200px" alt="dogo2_diff_gerchberg"/>|
|------------|------------|------------|------------|
| Smart Brute Force | Imagem da diferença Smart Brute Force | Gerchberg Papoulis | Imagem da diferença Gerchberg Papoulis |

Comparação do RMSE e tempo de execução para cada algoritmo:

| Algoritmo | RMSE | Tempo de execução |
|-----------|------|-------------------|
|Gerchberg Papoulis|23.039|00m09s|
|Brute Force|13.222|32m19s|
|Local Brute Force|12.456|01m50s|
|Smart Brute Force|11.384|00m36s|


### Texto

A imagem do carro de cabalos (horse_car.bmp) tem dimensões 438x297, e pelo tamanho da mascara obtida ser muito grande o tempo de execução do algoritmo de brute force seria muito alto, por isso não foi executado.

Comparação das imagens:

|<img src="./Project/images/inpainted/Smart Brute Force/horse_car.bmp"   width="200px" alt="horse_car_inpainted_brute"/>|
<img src="./Project/images/difference/Smart Brute Force/horse_car.bmp"   width="200px" alt="horse_car_diff_brute"/>|
<img src="./Project/images/inpainted/Gerchberg Papoulis/horse_car.bmp"   width="200px" alt="horse_car_inapinted_gerchberg"/>|
<img src="./Project/images/difference/Gerchberg Papoulis/horse_car.bmp"   width="200px" alt="horse_car_diff_gerchberg"/>|
|------------|------------|------------|------------|
| Smart Brute Force | Imagem da diferença Smart Brute Force | Gerchberg Papoulis | Imagem da diferença Gerchberg Papoulis |

Comparação do RMSE e tempo de execução para cada algoritmo:

| Algoritmo | RMSE | Tempo de execução |
|-----------|------|-------------------|
|Gerchberg Papoulis|20.856|00m09s|
|Local Brute Force|25.752|10m11s|
|Smart Brute Force|20.614|07m48s|



### Forbes

A imagem do forbes (forbes.bmp) tem dimensões 961x1280, e pelo seu tamanho ser muito grande o tempo de execução do algoritmo de brute force seria muito alto, por isso não foi executado.

Comparação das imagens:

|<img src="./Project/images/inpainted/Smart Brute Force/forbes.bmp"   width="200px" alt="forbes_inpainted_brute"/>|
<img src="./Project/images/difference/Smart Brute Force/forbes.bmp"   width="200px" alt="forbes_diff_brute"/>|
<img src="./Project/images/inpainted/Gerchberg Papoulis/forbes.bmp"   width="200px" alt="forbes_inapinted_gerchberg"/>|
<img src="./Project/images/difference/Gerchberg Papoulis/forbes.bmp"   width="200px" alt="forbes_diff_gerchberg"/>|
|------------|------------|------------|------------|
| Smart Brute Force | Imagem da diferença Smart Brute Force | Gerchberg Papoulis | Imagem da diferença Gerchberg Papoulis |

Comparação do RMSE e tempo de execução para cada algoritmo:

| Algoritmo | RMSE | Tempo de execução |
|-----------|------|-------------------|
|Gerchberg Papoulis|20.856|01m31s|
|Local Brute Force|09.140|19m07s|
|Smart Brute Force|08.533|06m10s|



## Remoção de objetos em imagens

Para verificar a funcionalidade dos algoritmos de *inpainting* implementados para a remoção de objetos mais largos e em contextos diferentes, foram realizados os testes abaixo.
Em cada imagem é mostrada a imagem original, imagem com adição da máscara em vermelho (imagem deteriorada) e o resultado do *Smart Brute Force*.

### Remoção de pessoa em frente a faixada de zoológico

|<img src="./Project/images/other/zoo.bmp"   width="300px" alt="zoo_original"/>|
<img src="./Project/images/deteriorated/zoo.bmp"   width="300px" alt="zoo_deteroprated"/>|
<img src="./Project/images/inpainted/Smart Brute Force/zoo.bmp"   width="300px" alt="zoo_inpainted_brute"/>|
|------------|------------|------------|
| Imagem Original | Imagem deteriorada | Smart Brute Force |

### Remoção de irregularidades na pele

|<img src="./Project/images/other/forbes_profile.bmp"   width="300px" alt="forbes_profile_original"/>|
<img src="./Project/images/deteriorated/forbes_profile.bmp"   width="300px" alt="forbes_profile_deteriorated"/>|
<img src="./Project/images/inpainted/Smart Brute Force/forbes_profile.bmp"   width="300px" alt="forbes_profile_inpainted_brute"/>|
|------------|------------|------------|
| Imagem Original | Imagem deteriorada | Smart Brute Force |

### Remoção de desenho em tinta sobre a pele

|<img src="./Project/images/other/gabi_star.bmp"   width="300px" alt="gabi_star_original"/>|
<img src="./Project/images/deteriorated/gabi_star.bmp"   width="300px" alt="gabi_star_deteriorated"/>|
<img src="./Project/images/inpainted/Smart Brute Force/gabi_star.bmp"   width="300px" alt="gabi_star_inpainted_brute"/>|
|------------|------------|------------|
| Imagem original | Imagem deteriorada | Smart Brute Force |

### Remoção de um colar

|<img src="./Project/images/other/team.bmp"   width="300px" alt="team_original"/>|
<img src="./Project/images/deteriorated/team.bmp"   width="300px" alt="team_deteroprated"/>|
<img src="./Project/images/inpainted/Smart Brute Force/team.bmp"   width="300px" alt="team_inpainted_brute"/>|
|------------|------------|------------|
| Imagem original | Imagem deteriorada | Smart Brute Force |

# Instruções para execução do código

O algoritmo de *Gerchberg Papoulis* foi implementado em python, enquanto os algoritmos de força bruta foram implementados em C++. É necessário ter o [OpenCV](https://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html) instalado para a execução do código. As imagens utilizadas estavam no formato Bitmap (.bmp).

A imagem de entrada deve estar na pasta project/images/deteriorated/, a máscara será salva em project/images/masks/ e a imagem de saída na pasta project/images/deteriorated/<inpainting_algorithm>/.

A compilação do código em C++ foi feita utilizando o cmake com o arquivo CMakeLists.txt, então para gerar o Makefile e compilar o executável é preciso executar os comandos dentro da pasta Project:

	cmake .
	make


A execução do código em **C++** é feita pelo comando:

	./main <image_in.bmp> <image_out.bmp> <mask_extraction_algorithm> <inpainting_algorithm> (compare)?

É possivel também executar somente a comparação, utilizando o comando:

	./main compare <path/original.bmp> <path/inpainted.bmp> <path/mask.bmp>

A execução do código em **Python** é feita pelo comando:

	python3 main.py <image_in.bmp> <image_out.bmp> <mask_extraction_algorithm> (compare)?

O código em **Python** só contém a implementação do algoritmo *Gerchberg Papoulis*, por isso não é necessário escolher o algoritmo de *inpainting*.

Os argumentos dos programas são:
 * <image_in.bmp> - Nome da imagem de entrada.
 * <image_out.bmp> - Nome da imagem de saída.
 * <mask_extraction_algorithm> - Algoritmo de extração da máscara (*most_frequent*, *minimum_frequency* ou *red*).
 * <inpainting_algorithm> - Algoritmo de *inpainting* (*brute*, *local*, *smart* ou *dynamic*).
 * (compare) - Opcional. Realiza a comparação entre a imagem original, se houver, e a imagem restaurada produzindo o RMSE e a imagem da diferença.
