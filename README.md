# Introdução À Computação Gráfica 2018.1

### Introdução:
O objetivo deste projeto é a rasterização de primitivas usando o algoritmo de Brasenham. Para esta tarefa utilizamos um framework desenvolvido pelo Prof. Christian Pagot que emula o acesso direto a memória de vídeo do computador via o ponteiro FBptr.

### Rasterização de um ponto:
Desenvolvemos uma função chamada putPixel que recebe como parâmetro  dois inteiros representando as coordenadas de um pixel e um array de quatro inteiros que representam os valore de RGBA que queremos representar nessas coordenadas.

```C++
void putPixel(int x, int y, color c) {
	FBptr[x*4 + y*4*IMAGE_WIDTH] = c[0];
	FBptr[x*4 + y*4*IMAGE_WIDTH + 1] = c[1];
	FBptr[x*4 + y*4*IMAGE_WIDTH + 2] = c[2];
	FBptr[x*4 + y*4*IMAGE_WIDTH + 3] = c[3];
}
```
Cada pixel é representado por quatro bytes onde cada byte especifíca uma intensidade de uma das cores RGBA nesta ordem. Como esses bytes estão armazenados linearmente na memória de vídeo, cada pixel  está disposto quatro posições a frente do pixel anterior. A posição inicial de cada pixel é dada pela função Offset = 4*x + 4*y*w onde x e y são as coordenadas do pixel e w é o comprmento da tela.


### Rasterização de uma linha:
Para rasterizar linhas, utilizamos o algoritmo de Bresenham generalizando-o para todos octantes do plano cartesiano.

#### Algoritmo de Bresebam:
Esse algoritmo nos da uma função incremental para a rasterização de uma reta, mas em sua forma natural ele é limitado ao primeiro octante do plano cartesiano(retas com inclinação entre 0 e 1). Ele é implementado em código da seguinte forma:

```C++
void drawLine(pixel i,pixel f, color c) {
	// Coordenadas iniciais
	int x = i.x;
	int y = i.y;
	//Deslocamentos
	int dx = f.x - i.x;
	int dy = f.y - i.y;
	
	int d = 2*dy - dx;
	int e_inc = 2*dy;
	int ne_inc = 2*(dy - dx);

	putPixel(i,c);			
	while(x < f.x) {
		if(d <= 0){
			x++;
			d += e_inc;			
		}
		else{
			x++;
			y++;
			d += ne_inc;			
		}
		putPixel(x,y,c);
	}
}
```

#### Como foi feito a generalização:

Para podermos desenhar retas em todos os octantes, precisamos fazer algumas observações:

![alt text](https://github.com/avillasilva/CG/raw/master/Fotos/Sem%20t%C3%ADtulo.png "octantes")

Podemos observar que se invertermos o ponto inicial e ponto final de retas que se encontram no 5° octante teremos retas que se enquadram no 1° octante. Isso também ocorre entre o 3° e 7°, 4° e 8° , 6° e 2° octantes. Devido a isso basta implementar os casos de retas nos 1°, 2°, 7° e 8° octantes, todos os outros casos podem ser feitos por espelhamento.

Em sua Forma final, a função drawLine ficou assim:

```C++
void drawLine(pixel i,pixel f, color c) {
	// Coordenadas iniciais
	int x = i.x;
	int y = i.y;
	//Deslocamentos
	int dx = f.x - i.x;
	int dy = f.y - i.y;
	
	if(abs(dx) > abs(dy)){
		//1 Octante
		if(dx > 0 && dy > 0) {
			int d = 2*dy - dx;
			int e_inc = 2*dy;
			int ne_inc = 2*(dy - dx);
			
			putPixel(i,c);			
			while(x < f.x){
				if(d <= 0){
					x++;
					d += e_inc;			
				}
				else{
					x++;
					y++;
					d += ne_inc;			
				}
				putPixel(x,y,c);
			}
		}

		//4 e 5 Octantes 
		else if(dx < 0) {
			drawLine(f,i,c);
		}

		//8 Octante e Linhas Horizontais
		else {
			int d = 2*dy + dx;
			int e_inc = 2*dy;
			int se_inc = 2*(dy + dx);
			
			putPixel(i,c);			
			while(x < f.x){
				if(d <= 0){
					x++;
					y--;
					d += se_inc;			
				}
				else{
					x++;
					d += e_inc;			
				}
				putPixel(x,y,c);
			}
		}
	}
```



### Raterização de Trinagulos:
Para rasterizar essa primitiva, apenas desenhamos três retas ligando três pontos recebidos como parametro.

```C++
	void drawTriangle(pixel v1,pixel v2, pixel v3, color c) {
	drawLine(v1,v2,c);
	drawLine(v1,v3,c);
	drawLine(v2,v3,c);
}
```

### Interpolação de cores:
A interpolção de cores foi feita a parti de uma nova função que baseada num fator de porcentagem (distancia parcial percorrida entre ponto atual e ponto final divido pela distancia total) que define a intensidade da cor inicial e da cor final num dado ponto.
```C++
void interpolar(float per,color atual, color inicial, color fin) {
	for(int i = 0; i < 4; i++) {
		atual[i] = (per * inicial[i]) + ((1-per) * fin[i]);	
	}
} 
```

Esta função é chamada antes do posicionamento de cada pixel definindo a cor deste pixel;
```C++
...
interpolar(per,atual,cInicial,cFinal);
putPixel(x,y,atual);
...
```

###Resultados:
