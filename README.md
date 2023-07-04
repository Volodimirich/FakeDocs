# FakeDocs

Данный проект содержит модели для генерации фейковых текстов на основе датасета MS MARCO и ему подобных. 
Главная задача, научиться генерировать текста, оцениваемые моделями ранжированиями выше, 
чем релевантные текста для данного запроса. 

Для оценки использовалась бибилиотека [docs-ranking-metrics](https://pypi.org/project/docs-ranking-metrics/).

Подробнее о проведенных экспериментах смотрите в каталоге `description_experiments`


# Запуск

**Все скрипты необходимо запускать из корневого каталога**

## Установка зависимостей

Перед запуском необходимо установить все зависимости, выполнив следующие команды:

```commandline
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

```commandline
pip install -r requirements.txt
```

Логирование метрик осуществляется в `wandb`. Поэтому перед запуском
необходимо авторизироваться, выполнив команду:

```commandline
wandb login
```

## Конфигурирование

Для обучения моделей доступны два скрипта: 
* `src/train.py` для обучения T5, 
* `src/train_gpt_clm.py` для обучения GPT на задаче CLM


Настройка параметров модели для обучения осуществляется в файле `configs/train_config.yaml`
Описание параметров:

```
Logging metrics:
* wandb_project --- Название проекта в `wandb` default="MadeFakeDocs"

Model
* tokenizer_name --- Название токенизатора для модели
* model_name --- Название модели
* local_path --- Путь до модели на локальным диске
* use_local --- True если загружать модель с диска, False загружать pretrain модель
* path_save_model --- *На данный момент не используется*
* input_max_length --- Максимальная длина входного текста в токенах
* target_max_length --- Моксимальная длина таргета модели в токенах
* total_samples --- Количество используемых примеров для обучения из датасета 
* type_of_training --- Тип обучения, доступны варианты CLM, TEACHER

Dataset
* save_dir --- Путь к папке для обучения
* dataset_name --- Название датасета для обучения. Доступны вариации part_data, full_data, made_data, made_valid_data
* block_size --- Размер блока текста для GPT
* mlm --- Следует ли использовать маскированное языковое моделирование или нет. Если установлено значение "False", метки совпадают с входными данными

TrainingArguments
* pre_trained --- *На данный момент не используется*
* output_dir --- Каталог с результатами обучения
* overwrite_output_dir --- True если перезаписывать каталог с результатами, иначе False
* num_train_epochs --- Количество эпох обучения
* per_device_train_batch_size ---  Размер батча для обучения
* per_device_eval_batch_size --- Размер батча при валидации
* warmup_steps --- Количество шагов  для планировщика скорости обучения
* gradient_accumulation_steps --- Для увеличения размера "виртуального" пакета

Optimizer
* lr --- Скорость обучения 
```

Для подсчета метрик ранжирования необходимо использовать скрипт `src/evaluate.py`

Конфигурирования осуществляется в файле `configs/evaluate_config.yaml`

```
Model
* tokenizer_name --- Название токенизатора для модели
* model_name --- Название модели
* local_path --- Путь до модели на локальным диске
* use_local --- True если загружать модель с диска, False загружать pretrain модель
* path_save_model --- *На данный момент не используется*
* input_max_length --- Максимальная длина входного текста в токенах
* target_max_length --- Моксимальная длина таргета модели в токенах
* total_samples --- Количество используемых примеров для обучения из датасета 
* type_of_training --- Тип обучения, доступны варианты CLM, TEACHER

result:
*  launch_description --- Описание эксперимента на основе которой будет назван каталог с результатами оценки
*  path_to_save_result --- Путь для сохранения рехультата
*  save_examples --- True если надо сохранять примеры генерации, иначе False
*  number_examples_batch --- Сколько батчех сохранять в качестве примеров
*  gpt_postprocessing --- True если обрезать текст в начале генерации на длину запроса, иначе False (Используется только для моделей на базе GPT)

Dataset
* save_dir --- Путь к папке для обучения
* dataset_name --- Название датасета для обучения. Доступны вариации part_data, full_data, made_data, made_valid_data
* block_size --- Размер блока текста для GPT
* mlm --- Следует ли использовать маскированное языковое моделирование или нет. Если установлено значение "False", метки совпадают с входными данными
```

Для генерации текстов необходимо использовать скрипт `src/predict.py`

Конфигурирования осуществляется в файле `configs/predict_config.yaml`. Файл конфигурации идентичен `configs/evaluate_config.yaml`
