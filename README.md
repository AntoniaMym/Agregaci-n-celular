# Agregaci-n-celular

%%%%%Ag75%%%%%% 

%El código funciona bien para 75 y 100 de plasma% 


% Directorio
cd('C:\Users\user\Desktop\Seminario\Medidas\Ag_75%');

% Filtramos y ordenamos solo las imagenes
S = dir('TC-*');
S = S(~[S.isdir]);
[~,idx] = sort([S.datenum]);
S = S(idx);

% Fondo sin celulas
fondo = rgb2gray(imread('TC1.tif'));
%% PASO 1
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% SELECCION DE ZONA A ANALIZAR + ECUALIZACION DE LA IMAGEN 
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

clc;close all

% Primero analizaremos sólo 1 imagen. Si funciona se automatiza para todo
% el directorio

% Abro imagen (i) y selecciono zona a analizar
for i=1:447
I=rgb2gray(imread(S(i).name));
I_cropped = I(200:900,300:1600);
I_eq = adapthisteq(I_cropped);

% Muestro en pantalla Imagen, selección y su ecualización

% % Seteo tamaño y posición de figura 
% figure,set(gcf,'Position',[100 500 1200 400])

% % Creo dos ventanas
% subplot(1,2,1),imagesc(I),colormap gray
% rectangle('Position', [300,200,1300,700], 'EdgeColor', 'k', 'LineWidth', 2);
% title('Imagen Original')
% subplot(1,2,2),imagesc(I_eq),colormap gray
% title('Selección central + ecualización')

%% PASO 2
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% BINARIZACIÓN + MEJORAMIENTO DE IMAGEN
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% Hay 2 formas de binarizar (probar la que de mejor resultado)
bw = imbinarize(I_eq, graythresh(I_eq));
%bw = imbinarize(I_eq,'adaptive','ForegroundPolarity','dark','Sensitivity',0.5);

% Invierto imagen
bw = ~bw;

% Elimino objetos pequeños (< 20 px) de la imagen binarizada
bw = bwareaopen(bw, 20);

% % % Muestro analisis
% figure,set(gcf,'Position',[100 200 1200 600])
% subplot(2,2,1),imagesc(I_eq),colormap gray
% % title('Selección central + equalización')
% subplot(2,2,2),imagesc(bw)
% % title('binarización bruta')

% relleno agujeros
bw2 = imfill(bw,'holes');
% subplot(2,2,3),imagesc(bw2), colormap gray
% title('Relleno de agujeros')

% Elemento morfologico (relleno con una forma predeterminada)
se1 = strel('disk', 1);
%bw3 = imdilate(bw2, ones(1,1));

% Dilatación de imagen con elemento anterior
bw3 = imdilate(bw2,se1);

% Relleno agujeros
bw4 = imfill(bw3,'holes');
% subplot(2,2,4),imagesc(bw4),colormap gray
% title('Dilatacion + relleno')
% %BWp_dilated = imdilate(BWp,ones(3,3),'ispacked');

%% PASO 3
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% BINARIZACIÓN + MEJORAMIENTO DE IMAGEN
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% Calcula la distancia (de la imagen invertida)
D = -bwdist(~bw4);
% figure,imshow(D,[])

% Transformación divisoria (WATERSHED)
% Esta es la clave y hay que estudiarla bien. Basicamente segmenta regiones
% continuas dada las diferencias de intensidad en la imagen
Ld = watershed(D);
% 
% % Muestro resultado (previamente se debe convertir Ld)
% figure,imshow(label2rgb(Ld))

% Elimino las distancia nulas en Ld
bw5 = bw4;
bw5(Ld == 0) = 0;
% figure,imshow(bw5)

% Correccion con el minimo
mask = imextendedmin(D,1.4);
% figure,imshowpair(bw,mask,'blend')

% Repito prcedimiento imponiendo el minimo
D2 = imimposemin(D,mask);
Ld2 = watershed(D2);
bw5 = bw4;
bw5(Ld2 == 0) = 0;

% Relleno agujeros
bw5 = imfill(bw5,'holes');

% % Comparo figuras
% figure,imshow(bw5)
% figure,imshowpair(I_cropped,bw5)

% Otros Intentos
% Extended-maxima transform
%mask_em = imextendedmax(I_eq, 60);
%mask_em = imclose(mask_em, ones(1,1));
%mask_em = imfill(mask_em, 'holes');
%mask_em = bwareaopen(mask_em, 20);
%figure,imshow(mask_em)

%overlay2 = imoverlay(I_eq, bw5 | mask_em, [.3 1 .3]);
%imshow(overlay2)

%% Paso 4 a partir del area, filtro los objetos pequeños

%cells = imcomplement(bw5);

% Detecta objetos conectados
CC = bwconncomp(bw5);

% Mide propiedades de distintas regiones de una imagen
% le pido el centroide, area y perimetro
stats = regionprops('table',CC,'Perimeter','Area','Centroid');

% Filtra manchas y base
% area minima=300, area max=550
idx = find([stats.Area] > 250 & [stats.Area] < 550);

% Indice con los que no cumplieron la condición
idx2 = setdiff(1:length(stats.Area), idx)'; 

% Elimino de la imgen los elementos filtrados
BW2 = ismember(labelmatrix(CC),idx);

% Posiciones dada esta condicion
x_cell = stats.Centroid(:,1);
y_cell = stats.Centroid(:,2);
% % Grafico y comparo los resultados
% figure,set(gcf,'Position',[100 200 1200 400])
% subplot(1,2,1),imagesc(bw5),colormap gray
% subplot(1,2,2),imagesc(BW2),colormap gray

% Finalmente grafico centroide de las que cumplen la condicion en rojo
% % % o sea las celulas
% figure,imagesc(bw5),colormap gray
% hold,plot(x_cell(idx),y_cell(idx),'r.')
% % y las agregadas en azules
% plot(x_cell(idx2),y_cell(idx2),'b.')

%%Cantidad de elementos que cumplen 
n(i)=numel(idx2); %células solas
z(i)=numel(idx); %agregaciones 

t=n+z;

T(i)=i;


end

T1(i)=T(i)*10;

plot(T1,n)
% title('número de células desagregadas en el tiempo')
plot(T1,z)
% title('RBCs aggregations over time')

%%TABLA%%

tabla= table(({'time' ; 'Individual cells' ; 'Number of aggregations'}),[T*10 ; n ; z])

%Calcular cantidad de células por rouleaux%
       
