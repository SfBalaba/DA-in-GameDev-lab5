# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #5 выполнил(а):
- Балаба Софья Николаевна
- РИ210940
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Интегрировать экономическую систему в проект Unity и обучить ML-Agent.

## Задание 1
### Измените параметры файла. yaml-агента и определить какие параметры и как влияют на обучение модели.

Ход работы:

- Нужно открыть проект в Unity, который представлен в методических указаниях, и ознакомиться с ним.

- С помощью Anaconda Prompt нужно активировать новый ML-агент и скачать новые библиотеки: mlagents 0.28.0, torch 1.7.1. Вводим такие команды:

```py
conda create -n MLAgents python=3.6
conda activate MLAgents
```

![image](https://user-images.githubusercontent.com/101496751/204228722-34969e3c-bf78-49ff-8744-6f085dc70b01.png)

```py
pip install mlagents==0.28.0
```

![image](https://user-images.githubusercontent.com/101496751/204228879-5a948438-9e82-404a-b91c-a040be2d59c3.png)

```py
pip install torch~=1.7.1 -f https://download.pytorch.org/whl/torch_stable.html
```

![image](https://user-images.githubusercontent.com/101496751/204228774-d3fbcc80-03c9-47e2-9be4-dc9fe7d1c7c6.png)


- Затем необходимо перейти в папку с проектом и начать обучение модели.

Файл Move.cs выглядит вот так: 

```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class Move : Agent
{
    [SerializeField] private GameObject goldMine;
    [SerializeField] private GameObject village;
    private float speedMove;
    private float timeMining;
    private float month;
    private bool checkMiningStart = false;
    private bool checkMiningFinish = false;
    private bool checkStartMonth = false;
    private bool setSensor = true;
    private float amountGold;
    private float pickaxeСost;
    private float profitPercentage;
    private float[] pricesMonth = new float[2];
    private float priceMonth;
    private float tempInf;

    // Start is called before the first frame update
    public override void OnEpisodeBegin()
    {
        // If the Agent fell, zero its momentum
        if (this.transform.localPosition != village.transform.localPosition)
        {
            this.transform.localPosition = village.transform.localPosition;
        }
        checkMiningStart = false;
        checkMiningFinish = false;
        checkStartMonth = false;
        setSensor = true;
        priceMonth = 0.0f;
        pricesMonth[0] = 0.0f;
        pricesMonth[1] = 0.0f;
        tempInf = 0.0f;
        month = 1;
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(speedMove);
        sensor.AddObservation(timeMining);
        sensor.AddObservation(amountGold);
        sensor.AddObservation(pickaxeСost);
        sensor.AddObservation(profitPercentage);
    }

    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        if (month < 3 || setSensor == true)
        {
            speedMove = Mathf.Clamp(actionBuffers.ContinuousActions[0], 1f, 10f);
            Debug.Log("SpeedMove: " + speedMove);
            timeMining = Mathf.Clamp(actionBuffers.ContinuousActions[1], 1f, 10f);
            Debug.Log("timeMining: " + timeMining);
            setSensor = false;
            if (checkStartMonth == false)
            {
                Debug.Log("Start Coroutine StartMonth");
                StartCoroutine(StartMonth());
            }

            if (transform.position != goldMine.transform.position & checkMiningFinish == false)
            {
                transform.position = Vector3.MoveTowards(transform.position, goldMine.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == goldMine.transform.position & checkMiningStart == false)
            {
                Debug.Log("Start Coroutine StartGoldMine");
                StartCoroutine(StartGoldMine());
            }

            if (transform.position != village.transform.position & checkMiningFinish == true)
            {
                transform.position = Vector3.MoveTowards(transform.position, village.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == village.transform.position & checkMiningStart == true)
            {
                checkMiningFinish = false;
                checkMiningStart = false;
                setSensor = true;
                amountGold = Mathf.Clamp(actionBuffers.ContinuousActions[2], 1f, 10f);
                Debug.Log("amountGold: " + amountGold);
                pickaxeСost = Mathf.Clamp(actionBuffers.ContinuousActions[3], 100f, 1000f);
                Debug.Log("pickaxeСost: " + pickaxeСost);
                profitPercentage = Mathf.Clamp(actionBuffers.ContinuousActions[4], 0.1f, 0.5f);
                Debug.Log("profitPercentage: " + profitPercentage);

                if (month != 2)
                {
                    priceMonth = pricesMonth[0] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[0] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }
                if (month == 2)
                {
                    priceMonth = pricesMonth[1] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[1] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }

            }
        }
        else
        {
            tempInf = ((pricesMonth[1] - pricesMonth[0]) / pricesMonth[0]) * 100;
            if (tempInf <= 6f)
            {
                SetReward(1.0f);
                Debug.Log("True");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
            else
            {
                SetReward(-1.0f);
                Debug.Log("False");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
        }
    }

    IEnumerator StartGoldMine()
    {
        checkMiningStart = true;
        yield return new WaitForSeconds(timeMining);
        Debug.Log("Mining Finish");
        checkMiningFinish = true;
    }

    IEnumerator StartMonth()
    {
        checkStartMonth = true;
        yield return new WaitForSeconds(60);
        checkStartMonth = false;
        month++;

    }
}
```

- Обучим модель и с помощью графиков и посмотрим на результат обучения:


```py
mlagents-learn Economic.yaml --run-id=Economic –-force
```

![image](https://user-images.githubusercontent.com/101496751/204237070-1ca94a82-f23f-4ee6-9970-1ec7de61130c.png)
![image](https://user-images.githubusercontent.com/101496751/204244449-8618f7df-5902-4059-8e0a-f1c68c46725b.png)


Установим TensorBoard для оценки результатов обучения:
```py
pip install tensorflow
```
![image](https://user-images.githubusercontent.com/101496751/204229940-db7a5d4d-de37-49e4-bfe6-65e90a26548c.png)

Вот такие результаты можно будет увидеть после выполнения команды, перейдя на локальный хост - http://localhost:6006/:

```py
tensorboard --logdir=results\Economic
```

![image](https://user-images.githubusercontent.com/101496751/204245238-ac1938ad-45d9-4a8d-9ac6-2931ce61ff99.png)

В первом запуске использовались следующие параметры в .yaml файле:

```py
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 1.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-2
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 3      
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

Cumulative Reward должно увеличиваться во время успешной тренировки (наш график немонотонный), а Policy Loss - уменьшаться (наш график этому соответствует).

В следующих запусках мы будем менять значение одного из параметров и смотреть, как он будет влиять на обучение модели.

- При втором запуске будем менять параметр strength: увеличим его в двое. Это фактор, на который можно умножить вознаграждение, получаемое от окружающей среды. 

```py
strength: 2.0
``` 

![image](https://user-images.githubusercontent.com/101496751/204290208-e0dc8b3d-20f6-4e1c-b43e-ff49d4586846.png)
![image](https://user-images.githubusercontent.com/101496751/204301788-03ea388f-bd32-41f3-937d-5a7479f0ba9c.png)

Cumulative Reward всегда равно единице, а Policy Loss - растет вверх. Мы не можем назвать это успешной тренировкой.

- При третьем запуске  мы увеличим параметр epsilon на 0.1. Этот параметр влияет на то, насколько быстро политика может развиваться во время обучения. Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении с градиентным спуском. Установка этого малого значения приведет к более стабильным обновлениям, но также замедлит процесс обучения. В нашем случае, он приведет к менее стабильным обновлениям

```py
epsilon: 0.3
```

![image](https://user-images.githubusercontent.com/101496751/204298885-80ffacce-9331-45f4-b869-65bd1af76e16.png)
![image](https://user-images.githubusercontent.com/101496751/204301839-7cd35be5-9e50-4a30-bd78-5698c14a7d94.png)

Cumulative Reward идет вниз, а Policy Loss - также идет вниз. Мы не можем назвать это успешной тренировкой.

- При четвертом запуске поработаем с параметром learning_rate: уменьшим его в три раза. Этот параметр - это начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если тренировка нестабильна, а вознаграждение постоянно не увеличивается. 

```py
learning_rate: 1.0e-4
```

![image](https://user-images.githubusercontent.com/101496751/204308718-6594c2f0-3b01-4f35-80aa-a48767b9b499.png)
![image](https://user-images.githubusercontent.com/101496751/204308868-51cc39d7-85b8-499a-89a5-6bf8aa986cde.png)

Cumulative Reward всегда равно единице, а Policy Loss - уменьшается. То есть, среднее нашего вознаграждения вседа равно единице, то есть оно однообразно, что не есть интересно.

- И при пятом запуске мы снова поработаем с epsilon: уменьшим его до 0.1 для стабилизации обновлений.

```py
epsilon: 0.1
```

![image](https://user-images.githubusercontent.com/101496751/204296952-60153e9a-c250-4d92-a0d0-abb4f3a8139c.png)
![image](https://user-images.githubusercontent.com/101496751/204302058-abf4192d-9dbc-4bed-9ac7-97c1ab112ea8.png)

Cumulative Reward всегда увеличивается, а Policy Loss - уменьшается. Это самая успешная тренировка из пяти.

- Посмторим на графики всех пяти тренировок вместе:

![image](https://user-images.githubusercontent.com/101496751/204309239-5020cdea-5bd1-4c49-9cd4-ba1a134c7b4d.png)

![image](https://user-images.githubusercontent.com/101496751/204493471-20c92548-2f3d-4bcf-b636-cddc877fc49a.png)


Можно сделать небольшой вывод: параметры epsilon, learning_rate и strength оказывают влияние на обучение модели. Лучший результат показало уменьшение параметра epsilon.

## Задание 2
### Опишите результаты, выведенные в TensorBoard.

- График Cumulative Reward: среднее совокупное вознаграждение за эпизод по всем агентам. Должно увеличиваться во время успешной тренировки. О значении на каждой тренировке было сказано в задании 1.

- График Episode Length: Средняя продолжительность каждого эпизода в окружающей среде для всех агентов. Во всех пяти тренировках был одинаковый.

- График Policy Loss: Средняя величина функции потерь. Соответствует тому, насколько сильно меняется политика (процесс принятия решений о действиях). Этот график должен уменьшаться во время успешной тренировки.

- График Value Loss: Средняя потеря функции обновления значения. Соответствует тому, насколько хорошо модель способна предсказать значение каждого состояния. Этот график должен увеличиваться, пока агент учится, а затем уменьшаться, как только вознаграждение стабилизируется. Во всех пяти тренировках график идет монотонно вниз.

- Графики в разделе Policy показывают изменение некоторых параметров, которые указаны в .yaml файле
    - График Beta: У всех пяти тренировок одинаковый: идет монотонно вниз.

    - График Entropy: Показывает насколько случайны решения модели. Должен медленно уменьшаться во время успешного тренировочного процесса. Уменьшается при первой и второй попытках, в остальных - увеличивается. (значит, удачная пятая попытка не такая уж и успешная).

    - График Epsilon: во всех пяти случаях уменьшается (графики имеют разные начальные значения, так как мы их меняли сами, например при 3 и 5 тренировках).

    - График Extrinsic Reward: Этот график соответствует среднему совокупному вознаграждению, полученному от окружающей среды за эпизод. Во втором графике значение 2, так как мы увеличивали параметр strength в два раза.

    - График Extrinsic Value Estimate: Оценка среднего значения для всех положений, посещенных агентом. Должно увеличиваться во время успешной тренировки. При 3 и 4 попытках графики сначал растут, а потом уменьшаются, в остальных случаях - наоборот.

    - График Learning Rate: все пять графиков идут вниз (4 график имеет другое начальное значение, так как мы сами его таким задали).

С помощью графиков можно понять, какие параметры стоит изменить или оставить для лучшей тренировки модели.

    
## Выводы
В ходе лабораторной работы интегрировала экономическую систему в проект Unity и обучила ML-Agent. Поработала с параметрами в .yaml файле: меняла значения для изменения обучения модели. В ходе этого выяснила, как некоторые параметры могут влиять на тренировку модели: strength - коэффициент, на который умножается вознаграждение, epsilon - влияет на стабильность обновлений, learning_rate - начальная скорость обучения. С помощью графиков, выведенных в TensorBoard анализировала попытки обучения модели, какие-то были более успешными, а другие - менее. Для поиска наилучшего обучения можно эксперементировать с параметрами в .yaml файле и при этом анализировать графики. 


| Plugin | README |
| ------ | ------ |
| GitHub | [plugins/github/README.md][PlGh] |
| Unity | [plugins/unity/README.md][PlU] |
| Anaconda Prompt | [plugins/anacondaprompt/README.md][PlAP] |
| TensorBoard | [plugins/tensorboard/README.md][PlTB] |
| Visual Studio Code | [plugins/visualstudiocode/README.md][PlVSC] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
