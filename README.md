# computer_vision_2
## Введение
Целью даной работы является изучение методов, определяющих ключевые точки на изображениях с целью определения траектории движения камеры. Для этого были взяты четыре изображения со сдвигом камеры, определены ключевые точки с помощью самостоятельно написанных функций, использующих детектор Харриса, функцию Лаплассиана, метод адаптивного радиуса для поиска наиболее важных ключевых точек. С помощью функции cKDTree дескрипторы, определенные для каждого изображения были сопоставлены с целью определения смещения камеры. Далее была построена траектория движения камеры.  
В работе будут использоваться четыре фотографии со сдвигом камеры:  
![крыса](https://github.com/LugenderGeist/computer_vision_2/blob/main/rat.png)

## Перевод изображений в Grayscale
Так как основной целью работы стоит написание метода поиска и сопоставления ключевых точек - переведем изображения в grayscale. Градации серого упрощают вычисления, так как вместо трех каналов (R, G, B) используется один канал интенсивности.
Для детектирования ключевых точек основным фактором является контрастность, а не цветовая информация. Поэтому цветные данные обычно не дают дополнительной пользы.  
![grayscale](https://github.com/LugenderGeist/computer_vision_2/blob/main/grayscale.png)  

## Выравнивание яркости  
Дескриптор ключевой точки (keypoint descriptor) — это числовое представление окрестности ключевой точки, которое описывает её уникальные характеристики. Дескриптор позволяет сравнивать ключевые точки между изображениями и определять их схожесть, даже если изображения подверглись изменениям.  
Разная яркость изображений может повлиять на формирование дескриптора, а при сопоставлении изображений многие алгоритмы опираются на сравнение дескрипторов ключевых точек. Если дескрипторы различаются из-за различий яркости - это может привести к ошибочным соответствиям или недостатку хороших совпадений. Поэтому выберем метод и выровняем яркость изображений.   
Для выравнивания яркости будем использовать метод линейной трансформации яркости и выровняем яркость второго изображения относительно первого.  
Этот метод изменит яркость и контраст второго изображения так, чтобы его среднее значение (mean) и стандартное отклонение (std) стали равны соответствующим значениям первого изображения.
- Для каждого изображения вычисляются среднее значение (mean) и стандартное отклонение (std).
- Вычисляется коэффициент масштабирования α=std1/std2, который регулирует контрастность.
- Вычисляется смещение β=mean1-αmean2, которое регулирует яркость.
- Каждый пиксель второго изображения преобразуется по формуле: new_pixel=α(pixel−mean2)+mean1
- После преобразования значения пикселей могут выходить за диапазон [0, 255], поэтому их необходимо нормализовать.  
 ![histogram](https://github.com/LugenderGeist/computer_vision_2/blob/main/histogramms.png) 

## Поиск ключевых точек 
В данном случае поиск ключевых точек осуществлялся с помощью детектора Харриса и Лаплассиана. Первоначально каждое изображение обрабатывается фильтром Гаусса, чтобы сгладить картинку и не тратить ресурсы на незначительные кандидаты в ключевые точки. Далее вычисляется Лапласиан: это оператор, вычисляющий сумму вторых производных интенсивности.
Он находит локальные максимумы и минимумы, которые являются областями с наибольшими градиентами. Из Лапласиана далее сформируется "маска", по которой будет работать детектор Харриса, что значительно упростит его вычисления.  
Далее вычисляются градиенты по осям X и Y, из которых будет сформирована матрица.  
![matrix](https://github.com/LugenderGeist/computer_vision_2/blob/main/matrix.PNG)  
Ответ Харриса - результат работы с матрицей, по значению которого можно определить, является ли точка ключевой.  
![harris](https://github.com/LugenderGeist/computer_vision_2/blob/main/harris_answer.PNG)  
Получим следующие ключевые точки на изображениях:  
![keypoints](https://github.com/LugenderGeist/computer_vision_2/blob/main/keypoints.png)  

## Фильтрация ключевых точек
Полученные точки будут отфильтрованы с помощью метода адаптивного радиуса: сначала все полученнные ключевые точки будут отсортированы по степени яркости. В заданном радиусе вокруг первой в списке точки будут убраны все остальные так, чтобы осталась самая значимая. Далее функция пройдется по всему списку, чтобы в заданном радиусе вокруг точек не осталось лишних. Таким образом, останутся самые значимые точки.  
![filtered](https://github.com/LugenderGeist/computer_vision_2/blob/main/filtered.png)  

## Сопоставление точек
Для сопоставления точек между изображениями необходимо посчитать дескрипторы: векторы признаков, которые описывают окрестность ключевой точки. Для каждой точки определяется окно анализа, в котором вычисляются градиенты, а после - их магнитуды и направления. Далее пиксели в выбранном окне "голосуют" за бин, соответствующий его направлению градиента, после чего формируется гистограмма ориентаций как массив значений каждого бина.
Сопоставление будет осуществляться с помощью cKDTree: этот метод строит дерево, разделяющее пространство пополам по одной из координат для поиска дескриптора, ближайшего к заданному. Вместо того чтобы сравнивать его со всеми дескрипторами последовательно, KDTree позволяет сделать это быстрее, выбирая только наиболее близкие кандидаты.  
Также в работе применяется тест Лоу для исключения ошибочных соответствий между дескрипторами: дескриптор ключевой точки может иметь несколько кандидатов на соответствие: в случае, если дескриптор на первом изображении в равной степени соответствует двум дескрипторам на следующем изображении - такую точку не стоит брать в качестве опоры для построения траектории движения камеры.  
Результат сопоставления ключевых точек между всеми изображениями:  
![keypoints_1](https://github.com/LugenderGeist/computer_vision_2/blob/main/matches.png)  

## Построение траектории движения камеры
По полученным ключевым точкчам построим траекторию движения камеры. Для этого необходимо определить несколько величин.
K — внутренняя матрица камеры [3x3]. Зададим ее как примерную матрицу для камеры с разрешением 1280х720p.  
Эссенциальная матрица E — это матрица [3×3], которая связывает соответствующие точки на двух изображениях, сделанных одной камерой с разных позиций. Она содержит информацию о положении и ориентации камеры между двумя кадрами. Математически, если точка x1 на первом изображении соответствует точке x2 на втором изображении, то их координаты удовлетворяют следующему уравнению:  
![essential](https://github.com/LugenderGeist/computer_vision_2/blob/main/essential.PNG)   
Здесь x1 и x2 — нормализованные координаты точек (в однородных координатах). E — эссенциальная матрица.  
Из эссенциальной матрицы можно получить матрицу поворота R и вектор перемещения t, которые описывают движение камеры между двумя кадрами.  
![trajectory](https://github.com/LugenderGeist/computer_vision_2/blob/main/trajectory.png)  

По результатам построения траектории видно, как именно была сдвинута камера. В результате выполнения работы был изучен принцип поиска ключевых точек с помощью детектора Харриса с применением функции Лаплассиана и фильтра Гаусса, способ фильтрации ключевых точек с помощью адаптивного радуиса. Был изучен принцип построения дескрипторов для ключевых точек изображения, а также метод сопоставления дескрипторов KDTree.
