# hse_hw1_meth

## Исходные данные

В данном задании будут проанализированы следующие 3 BS-Seq образца, полученные на разных стадиях развития эмбриона мыши:

Образец | Индекс файла | Описание
--- | --- | --- 
8 Cell | SRR5836473 | 8-клеточный эмбрион, примерно 2.25 дня после оплодотворения яйцеклетки
ICM | SRR5836475 | внутренняя клеточная масса бластоциста, примерно 3.5 дня после оплодотворения яйцеклетки
Epiblast | SRR3824222 | стадия эпибласта, примерно 6.5 дней после оплодотворения яйцеклетки

## Ссылка на Colab

[Ссылка](https://colab.research.google.com/drive/1CXUvfD5kTw2D6q5QcRZYvETQcW1kkbWL?usp=sharing)

## Выполнение

### 1
Сравним анализ прочтений epyblast и примера RNA-Seq секвенирования с индексом SRR3414631 (семестр 3 дз 3). Оба результата получены по Mus musculus (домашняя мышь).
![SRR3414631](https://github.com/ytken/hse_hw1_meth/blob/main/fastqc/ENA_SRR3414631.png) 



### 2а

Число ридов, закартированных на участки
Участок | 8 Cell | ICM | Epiblast
--- | --- | --- | ---
11347700-11367700 | 1090 | 1456 | 2328
40185800-40195800 | 464 | 630 | 1062

### 2b

Сколько процентов прочтений дуплицированно в каждом из образцов?
Образец | Процент дуплицирования (%)
--- | ---
8 Cell | 81.69
ICM | 90.92
Epiblast | 97.08

### 2d

Отчеты метилирования 3 образцов можно найти в папке проекта.

M-bias plot

#### 8 cell
![8 cell read 1](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/8cell_read1.png) 
![8 cell read 2](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/8cell_read2.png)
#### ICM
![ICM read 1](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/ICM_read1.png) 
![ICM read 2](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/ICM_read2.png)
#### Epiblast
![Epiblast read 1](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/epiblast_read1.png) 
![Epiblast read 2](https://github.com/ytken/hse_hw1_meth/blob/main/img_bismark/epiblast_read2.png)
#### Анализ
В соответствии с документацией Bismark Bisulfite Mapper:
> Начиная с Bismark v0.8.0, экстрактор метилирования Bismark также создает график смещения метилирования, который
> показывает долю метилирования в каждой возможной позиции в считывании. График также содержит абсолютное количество 
> вызовов метилирования (как метилированных, так и неметилированных) для каждой позиции.

Также из задания:
>  Считается, что при развитии эмбриона происходят так называемые волны деметилирования-метилирования, т.е. на ранних 
>  стадиях CpG метилирование уменьшается до некоторого минимума (около 25%), а затем по мере дифференцировки тканей, оно 
>  сильно увеличивается (около 90%) и остается таким на протяжении всей жизни организма.

По результатам анализа получены сделующие (приближенные) результаты метилирования:
Образец | Метилирование GpG динуклеотидов (%)
--- | ---
8 cell | 43
ICM | 23
Epiblast | 77

Результат соответствует ожидаемому и уточняет его.

Такой анализ может быть полезен:
> График M-смещения должен позволить исследователям принять обоснованное решение о том, следует ли оставлять смещение в
> окончательных данных или удалять его (например, используя опцию извлечения метилирования - игнорировать).

### 2e
Для визуализации распределения метилирования цитозинов по хромосоме напишем функцию. На вход используем файлы .bedgraph.
```python
def methylation_bedgraph_visualizer(filename):
  data = pd.read_csv(filename,  delimiter='\t', skiprows=1, header=None)
  plt.hist(data[3], bins=100, density=True)
  plt.title('Распределение GpG метилирования '+filename)
  plt.xlabel('Метилированные цитозины (%)')
  plt.ylabel('Частота')
  plt.show()
```
Результат
#### 8 cell
![8 cell](https://github.com/ytken/hse_hw1_meth/blob/main/img_meth/meth_8cell.png) 
#### ICM
![ICM](https://github.com/ytken/hse_hw1_meth/blob/main/img_meth/meth_ICM.png) 
#### Epiblast
![Epiblast](https://github.com/ytken/hse_hw1_meth/blob/main/img_meth/meth_epiblast.png) 
Заметим, что наиболее неравномерное распределение метилированных цитозинов для образцов 8 cell. Чуть меньше половины не метилированные, 
около четверти полностью метилированные и в остальном видно нормальное распределение с пиком в 50% метилирования. Для образцов ICM с вероятностью 0.65 встретятся
не метилированные, а также заметны максимумы в 50% и 100% частотой 0.1 . Для epiblast наибольшая вероятность ~0.47 встречи полностью метилированного образца, но также
есть вероятность ~0.12 встретить не метилированный.

Тенденция волн CpG деметилирования-метилирования наблюдается, но есть также и отклонения, которые нельзя отбросить.

### 2f
Визуализируем уровень метилирования и покрытия для каждого образца с помощью pyGenomeTracks. 

Полные треки
![](https://github.com/ytken/hse_hw1_meth/blob/main/img_cov/image_cov.png) 
Приближен наиболее интересный участок
![](https://github.com/ytken/hse_hw1_meth/blob/main/img_cov/image_cov_enlarged.png) 
