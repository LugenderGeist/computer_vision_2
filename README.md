# computer_vision_2
## Введение
В данной работе Необходимо:
- Взять 2 изображения со сдвигом
- Подготовить изображения
- Определить на каждой фотографии ключевые точки
- Отфильтровать самые наилучшие
- Построить по каждой точке дескриптор (можете использовать любой, рекомендуется SIFT)
- Сопоставить два соседних изображения на предмет соответствия ключевых точек
- Построить траекторию движения камеры  
В работе будут использоваться две фотографии со сдвигом камеры:  
![крыса](https://github.com/LugenderGeist/computer_vision_2/blob/main/rat.png)  
## Перевод изображений в Grayscale
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
 ![brightness]([https://github.com/LugenderGeist/computer_vision_2/blob/main/brightness.png)
По гистограмме сложно сказать, что изображения стали ближе друг к другу по яркости. Однако, работа уже проделана, поэтому проверим, действительно ли такая обработка изображения в данном случае даст какой-то значимый результат при поиске общих ключевых точек.  
## Применение метода SIFT
![SIFT]([https://github.com/LugenderGeist/computer_vision_2/blob/main/keypoints.png)  
## Фильтрация ключевых точек
![filtered](https://github.com/LugenderGeist/computer_vision_2/blob/main/filtered_keypoints.png)  

## Сопоставление точек
FLANN (Fast Library for Approximate Nearest Neighbors) — это библиотека для быстрого поиска приближенных ближайших соседей в многомерных пространствах. FLANN решает задачу поиска ближайших соседей, которая заключается в нахождении точек из набора данных, наиболее близких к заданной точке. В контексте компьютерного зрения FLANN используется для сопоставления дескрипторов ключевых точек между изображениями.  
Сначала сопоставим точки между первым изображением и обработанным:  
![keypoints_1](https://github.com/LugenderGeist/computer_vision_2/blob/main/keypoints_1.png)  
Теперь сопоставим первое изображение с необработанным:
![keypoints_2](https://github.com/LugenderGeist/computer_vision_2/blob/main/keypoints_2.png)  
После сопоставления первого изображения с обработанным и необработанным получим разницу всего в одну ключевую точку. При изменении значений size и response в большую и меньшую стороны - количество ключевых точек все равно не отличается больше чем на 3. Из этого можно сделать вывод, что в данной ситуации, при незначительном отклонении камеры между двумя изображениями и фотографиями, отличающимися незначительно по яркости - не обязательно регулировать яркость изображения, так как она, видимо, находится в допустимом диапазоне.  
## Построение траектории движения камеры
По полученным ключевым точкчам построим траекторию движения камеры. Для этого необходимо определить несколько величин.
K — внутренняя матрица камеры [3x3]. Зададим ее как примерную матрицу для камеры с разрешением 1280х720p.  
Фундаментальная матрица F — это матрица [3×3], которая также связывает соответствующие точки на двух изображениях, но в пиксельном пространстве. Она вычисляется с использованием алгоритмов, таких как RANSAC, который отсеивает шумные соответствия.  
Эссенциальная матрица E — это матрица [3×3], которая связывает соответствующие точки на двух изображениях, сделанных одной камерой с разных позиций. Она содержит информацию о положении и ориентации камеры между двумя кадрами. Математически, если точка x1 на первом изображении соответствует точке x2 на втором изображении, то их координаты удовлетворяют следующему уравнению:  
x2^T⋅E⋅x1=0  
Здесь x1 и x2 — нормализованные координаты точек (в однородных координатах). E — эссенциальная матрица.  
Из эссенциальной матрицы можно получить матрицу поворота R и вектор перемещения t, которые описывают движение камеры между двумя кадрами.
![trajectory](https://github.com/LugenderGeist/computer_vision_2/blob/main/trajectory.png)  
