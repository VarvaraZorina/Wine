# Модель обучения нейронной сети распознаванию этикеток на бутылках вина по фотографиям

## Структура проекта
```
WINE/
├── data.yaml                   # Конфигурационный файл
├── README.dataset.txt          # Информационный файл
├── README.roboflow.txt         # Информационный файл
├── test/                       # Тестовый каталог
│   ├── images/                 # Подкаталог изображений
|   |   └── ...                 
│   └── labels/                 # Подкаталог меток
|       └── ...                 
├── train/                      # Обучающий каталог
│   ├── images/                 # Подкаталог изображений
|   |   └── ...                 
│   └── labels/                 # Подкаталог меток
|       └── ...                 
└── valid/                      # Валидационный каталог
    ├── images/                 # Подкаталог изображений
    |   └── ...                 
    └── labels/                 # Подкаталог меток
        └── ...                 
```
## Основные этапы работы
1. **Загрузка данных**: 
    - Подготовка размеченного набора данных в формате, необходимом для YOLOv11. Делаем это с помощью Roboflow.
    - Загружаем данные для преобразования: используем набор изображений из открытых источников.
2. **Маркировка данных**: 
   - Определение классов, используемых для маркировки.
   - Маркировка объектов на изображении в соответствии с классами.
3. **Создание набора данных**: 
   - Устанавка необходимых параметров и генерация набора данных для дальнейшего хранения и обработки.
   - Экспорт датасета.
4. **Обучение модели**: Обучение модели YOLOv11х и визуализация результатов обучения.
5. **Развертывание**: Разворачиваем обученную модель в Roboflow.

# Отчет
### 1.Введение

#### 1.1 Цель проекта

<h4>
    Обучение нейронной сети распознаванию этикеток на бутылках вина, применяемое в приложениях для поиска и оценки вин.
</h4>

#### 1.2 Задачи проекта

1. Создание набора данных с помеченными классами; 
2. Экспорт набора данных для использования в обучении модели; 
3. Обучение модели в среде Google Colab; 
4. Развертывание и вывод данных;
5. Анализ полученных данных.

#### 1.3 Актуальность

Культура потребления вина в России с каждым годом набирает все большую популярность. Теперь потребителю становится важно разбираться в вине и пробовать новое, находить для себя любимые сорта и страны. Для облегчения выбора потребителя актуальна разработка приложения для оценки и помощи в подборе вин. 
С помощью приложения, использующего нейросеть для распознавания этикеток, пользователи могут быстро получать информацию о вине, включая характеристики, рекомендации по сочетанию с блюдами и отзывы.

### 2. Основная часть
#### 2.1 Получение рабочей директории:
```bash
import os
   HOME = os.getcwd()
   print(HOME)
```

Здесь мы получаем текущую рабочую директорию, что необходимо для создания директорий или работы с файлами.
#### 2.2 Установка необходимых библиотек
```bash
!pip install "ultralytics<=8.3.40" supervision roboflow
```

Устанавливаем библиотеки, которые понадобятся для работы с моделями.
#### 2.3 Создание папки для данных
```bash
!mkdir {HOME}/datasets
```

#### 2.4 Получение ключа API и загрузка проекта

Подключаемся к Roboflow с вашим API-ключом и загружаем нужный проект.
Создаем папку для хранения датасетов, что поможет организовать файлы.
```bash
from roboflow import Roboflow
rf = Roboflow(api_key="Ваш_API_Ключ")
project = rf.workspace("ваш_проект").project("ваш_подпроект")
dataset = project.version(1).download("yolov11")
```
#### 2.5 Настройка данных

Удаляем ненужные строки из файла `data.yaml`. Это позволяет устранить возможные ошибки, и вносим измененные пути к изображениями.
```bash
!sed -i '$d' {/content/datasets/Вино-1/data.yaml}/data.yaml   # Удаляет последнюю строку
!echo 'test: ../test/images' >> {/content/datasets/Вино-1/test/images}/data.yaml
!echo 'train: ../train/images' >> {/content/datasets/Вино-1/train/images}/data.yaml
!echo 'val: ../valid/images' >> {/content/datasets/Вино-1/valid/images}/data.yaml
```
#### 2.6 Обучение модели

Запускаем процесс тренировки модели. Здесь `epochs=50` указывает количество эпох (число полных проходов по набору данных), а `imgsz=640` - размер изображений.
```bash
!yolo task=detect mode=train model=yolo11s.pt data={dataset.location}/data.yaml epochs=50 imgsz=640 plots=True
```
#### 2.7 Предсказание объектов

После обучения модели, теперь мы можем использовать ее для распознавания объектов на новых изображениях.
```bash
!yolo task=detect mode=predict model={HOME}/runs/detect/train/weights/best.pt conf=0.25 source={dataset.location}/test/images save=True
```
#### 2.8 Отображение результатов

Отображаем результаты на первых трех изображениях из последних предсказаний.
```bash
import glob
import os
from IPython.display import Image as IPyImage, display

latest_folder = max(glob.glob('/content/runs/detect/predict*/'), key=os.path.getmtime)
for img in glob.glob(f'{latest_folder}/*.jpg')[:3]:
    display(IPyImage(filename=img, width=600))
    print("\n")
```
#### 2.9 Подключение и использование модели

И, наконец, подключаемся к развернутой модели для выполнения инференса (вывода) на тестовых изображениях. Сначала установим библиотеку `inference`.
```bash
!pip install inference
```

Затем импортируем необходимые библиотеки и запускаем инференс на случайных изображениях.
```bash
import cv2
import supervision as sv
from inference import get_model

model_id = project.id.split("/")[1] + "/" + dataset.version
model = get_model(model_id, "<ROBOFLOW_API_KEY>")

test_set_loc = dataset.location + "/test/images/"
test_images = os.listdir(test_set_loc)
for img_name in random.sample(test_images, min(4, len(test_images))):
    print("Running inference on " + img_name)
    image = cv2.imread(os.path.join(test_set_loc, img_name))
    results = model.infer(image, confidence=0.4, overlap=30)[0]
    detections = sv.Detections.from_inference(results)
    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()
    annotated_image = box_annotator.annotate(scene=image, detections=detections)
    annotated_image = label_annotator.annotate(scene=annotated_image, detections=detections)
    _, ret = cv2.imencode('.jpg', annotated_image)
    i = IPyImage(data=ret)
    display(i)
```

### 3. Выводы
Проект демонстрирует процесс обучения модели для распознавания этикеток на бутылках вина. Мы получили визуализацию результатов для дальнейшего анализа данных.
В ходе работы была проверена эффективность обучения модели и возможность дальнейшей работы с новыми данными.
