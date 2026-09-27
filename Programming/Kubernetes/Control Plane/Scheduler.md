
## [Base](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)

**Планирование в Kubernetes** подразумевает обеспечение того, чтобы pod'ы были назначены node'ам и `kubelet` мог поднять их на этих node'ах.

**`Scheduler`** - это компонент `Control Plane`, который следит за наличием вновь созданных pod, которые еще не были назначены node'ам кластера, и для каждого такого pod'а выполняет поиск наилучшей node'ы в кластере.

**`Kube-scheduler`** - это стандартный планировщик для кластера Kubernetes. Он разработан таким образом, что при желании, есть возможность написать свой собственный компонент планирования и использовать его.

Поскольку контейнеры в pod'ах и сами pod'ы, могут иметь разные требования по ресурсам, планировщик отфильтровывает любые node'ы, которые не соответствуют конкретным потребностям pod'а в планировании. При назначении pod'а определенной node, `scheduler` опирается на `Resource request`(минимально выделяемое значение ресурсов для container'ов в pod'е), а не на реальное потребление этих ресурсов на определенной node. 

Например, если существует потребность назначить pod, то `scheduler` смотрит на request'ы для этого pod'а и ищет node'у, на которую есть возможность установить данный pod, опираясь на уже имеющиеся `Resource request'ы` у работающих pod'ов на этой node. То есть, если `scheduler` пытается назначить pod с `Resource request` = 2CPU, и при этом существует node с 4CPU и текущей утилизацией всего 1CPU, то это вовсе не означает, что `scheduler` сможет назначить pod на данную node. Т.к. на этой node могут находиться pod'ы с суммарным `Resource request` = 3CPU, несмотря на реальное потребление CPU только в размере одного ядра. 

В кластере узлы (node'ы), отвечающие требованиям планирования для pod'а называются допустимыми узлами (`feasible node`)
**Scheduler** находит все `feasible node'ы` и вызывает определенные оценочные функции, для того чтобы решить, на какую из этих node установить pod.

Таким образом, планирование в Kubernetes с помощью `kube-scheduler` происходит в 2 этапа:
1. Filtration (фильтрация)
	На данном этапе scheduler просеивает все узлы кластера с помощью определенных фильтров. Например, фильтр PodFitsResources проверяет, достаточно ли у узла-кандидата доступных ресурсов для удовлетворения конкретных запросов pod'а на ресурсы.
2. Scoring (подсчет очков)
	После того как список из возможных кандидатов сформирован, scheduler, с помощью вызова оценивающих функций, присваивает каждой node определенное кол-во очков. Node'е с наибольшим кол-вом очков присваивается pod.  Если таких node несколько, то выбирается одна случайным образом.

Существует два поддерживаемых способа настройки поведения планировщика при фильтрации и оценке:
	1. Политики планирования (`Scheduling Policies`) - через настройку предикатов и приоритетов (`Predicates and Priorities`)
	2. Профили планирования (`Scheduling Profiles`) - через настройку плагинов, реализующих различные этапы планирования, включающие: `QueueSort`, `Filter`, `Score`, `Bind`, `Reserve`, `Permit`, и другие.

--- 

## [Scheduling Framework](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)

**`Scheduling Framework`** - это современная архитектура планировщика `kube-scheduler`. Она позволяет разработчикам и администраторам легко расширять функциональность планировщика, с помощью добавления самописных плагинов *(Scheduling Plugins)*, при этом ядро планировщика остается легким и поддерживаемым. 
### [Scheduler configuration](https://kubernetes.io/docs/reference/scheduling/config/#profiles)

Существует возможность настраивать поведение `kube-scheduler`, создав конфигурационный файл и передав его путь в качестве аргумента командной строки.

Для конфигурирования планировщика ранее использовалась технология `Scheduling Policies`, но в силу её неповоротливости, была заменена на более актуальную и flexible архитектуру `Scheduling Framework`(о ней чуть позже).

**Устаревшая архитектура:**
```
Scheduler Configuration (старый формат)
└── Policy файл (--policy-config-file)
    ├── Predicates (фильтры)
    └── Priorities (веса)
```

**Обновленная архитектура:**
```
Scheduling Framework (концепция: "процесс планирования состоит из слотов")
│
└── Scheduler Configuration (конкретный файл, реализующий эту концепцию)
    │
    └── Profiles (для разных типов подов)
        │
        └── Profile: default-scheduler
            ├── Plugins (список того, что умеет этот профиль)
            │   ├── NodeResourcesFit (определенная логика поведения)
            │   └── ImageLocality (определенная логика поведения)
            │
            └── Extension Points (в какие этапы планирования эти плагины вставлять)
                ├── filter: сюда идет NodeResourcesFit
                └── score: сюда идут NodeResourcesFit и ImageLocality
```

#### [Scheduling Policies](https://kubernetes.io/docs/reference/scheduling/policies/)(decommissioned)

До версии Kubernetes 1.23 существовал такой функционал как `Scheduling Policies`. Настройка работы Scheduler определялась через отдельный файл (`--policy-config-file`), в котором были жестко прописаны:
1. **`predicates`** (предикаты) — функции для фильтрации узлов (аналог современного этапа `Filter`).
2. **`priorities`** (приоритеты) — функции для ранжирования узлов (аналог современного этапа `Score`).
Данный подход является весьма трудно изменяемым и сложно масштабируется. Также в данную систему нельзя было добавить несколько профилей планирования, поэтому ей на замену пришла новая архитектура планировщика - `Scheduling Framework`

Грубо говоря, `Scheduling policies` - это устаревший способ конфигурирования планировщика. Сейчас используется `Scheduling Framework` реализующий конфигурацию через `Plugins`(плагины) и `Extension points`(этапы планирования)

#### [Scheduling Profiles](https://kubernetes.io/docs/reference/scheduling/config/#profiles)

**`Scheduling Profiles` (Профили планирования)** — это наборы правил (плагинов), которые задают schduler'у: "как именно нужно распределять pod'ы". В новых версия Kubernetes у одного scheduler может быть несколько профилей планирования.

Профиль планирования позволяет настраивать различные этапы планирования (точки расширения). Существует возможность указать профили планирования, запустив команду `kube-scheduler --config <filename>`и используя структуру [KubeSchedulerConfiguration v1](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/)
###### Минимальная конфигурация:
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/srv/kubernetes/kube-scheduler/kubeconfig
```

Ранее scheduler работал по единым фиксированным правилам и для их изменения приходилось запускать отдельный измененный scheduler (второй бинарник). Очевидно, что это неудобно. Поэтому со временем была реализована концепция `Scheduling Framework`, позволяющая использовать несколько `Scheduling Profiles` и описать в одном едином конфигурационном файле (для одного scheduler'а) различные ***профили*** (правила) поведения при планировании.

Каждый **профиль планирования** состоит из "Extension Points" (точек расширения). Каждое **расширение** - это один конкретный этап процесса планирования. 

**Ключевые расширения:**
- **`queueSort`**: Определяет, в каком порядке поды стоят в очереди.
- **`filter`**: Отсеивает node'ы, которые не подходят (предикаты).
- **`score`**: Оценивает оставшиеся node'ы по баллам (приоритеты).
- **`bind`**: Привязывает под к выбранной node.
- И еще куча других: `preFilter`, `postFilter`, `preScore`, `reserve`, `permit` и т.д.

Plugins (плагины) - это кусочки кода, которые реализуют конкретную логику. Они подключаются к расширениям

Например:
- Плагин **`NodeResourcesFit`** работает на этапе `filter` (отсеивает node'ы, где мало ресурсов) и на этапе `score` (оценивает, где ресурсов больше).
- Плагин **`ImageLocality`** работает на этапе `score` (дает баллы node'ам, где уже есть нужный образ)
- Полный список плагинов можно посмотреть [тут](https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins)

Сборка уникального `Scheduling Profile` (профиля) предполагает, что мы назначаем конкретному этапу планирования (`Extension Point` - расширению) определенные plugin'ы, тем самым описываем поведение scheduler'а на данных этапах.
###### Пример конфигурационного файла scheduler'а с несколькими профилями:
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  # Профиль 1: Обычный (дефолтный)
  - schedulerName: default-scheduler
  
  # Профиль 2: Для тестов — вообще без скоринга
  - schedulerName: no-scoring-scheduler
    plugins:
      preScore:
        disabled:
        - name: '*'
      score:
        disabled:
        - name: '*'
  
  # Профиль 3: Кастомный — со своим плагином
  - schedulerName: my-custom-scheduler
    plugins:
      score:
        disabled:
        - name: TaintToleration  # Отключили стандартный
        enabled:
        - name: MySuperPlugin    # Включили свой
          weight: 10
```
*Как видно из примера, есть возможность указать свой самописный плагин (обычно на Go, но можно использовать и другие языки)*

---
## FAQ
#### Как pod выбирает необходимый ему профиль?
Для этого необходимо просто прописать какой профиль планирования использовать при развертывании данного pod'а в манифест-файле этого же pod'а
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: no-scoring-scheduler  # <-- Вот тут выбор профиля
  containers:
  - name: app
    image: nginx
```

#### Как scheduler считает scoring?

На каждом этапе планирования (точке расширения) планировщик ожидает ответ на определенный вопрос негласно закрепленный за каждым этапом. 

Например, на этапе **Filter** планировщик ожидает ответ на вопрос "Может ли под запуститься на этой node в целом?". Или на этапе **Score** на вопрос "Насколько хороша эта node для данного pod'а?" (оценка от 0 до 100).

Допустим после этапа **Filter** планировщик рассматривает неотсеянные node'ы:
- **Node А**: 16 CPU свободно, образа нет (надо качать), лейблы совпадают.
- **Node B**: 8 CPU свободно, образ уже скачан, лейблы совпадают.
- **Node C**: 2 CPU свободно, образ уже скачан, лейблов нет.
Планировщик должен выбрать наилучшую node из приведенного списка. Чтобы это сделать он высчитывает определенный `scoring` с помощью вызова плагинов для текущего этапа планирования (этап **Score**).

Также допустим, что к данному этапу и его обработке подключены 2 плагина:
```yml
plugins:
  score:   # <== этап планирования
    enabled: # <== массив включенных плагинов
    - name: NodeResourcesFit # <== плагин
      weight: 1 # <== вес плагина
    - name: ImageLocality # <== плагин
      weight: 2 # <== вес плагина
```

Каждый плагин имеет свой вес (поле `weight`), которое определяет значимость плагина и напрямую связано с его влиянием на итоговый набор очков (`scoring`). Как видим из данного примера, для нас большую важность имеет наличие установленного образа контейнера на node, нежели чем наличие большого кол-ва свободных ресурсов.

Каждый плагин в зоне своей ответственности оценивает node'у и выставляет определенный балл (от 0 до 100). 
*На самом деле, каждый плагин имеет свой собственный диапазон: от 0 до 10, от 0 до 10000 и другие. Однако существует специальный механизм нормализации баллов, реализованный через **`NormalizeScore()`** - метод, приводящий баллы к общему знаменателю (к шкале от 0 до 100)*

Вернемся к пример:
- `NodeResourceFit` - оценивает наличие свободных ресурсов на node
	**node A**: 100
	**node B**: 60
	**node C**: 20
	*После чего данные очки умножаются на **вес** плагина (т.е. на 1) и получаем итоговое кол-во очков которые плагин вкладывает в результирующий `scoring`*:
	**node A**: 100 × weight = 100 × 1 = 100
	**node B**: 60 × weight = 60 × 1 = 60
	**node C**: 20 × weight = 20 × 1 = 20

- `ImageLocality` - оценивает наличие скачанного образа для контейнера использующегося внутри планируемого к назначению pod'а
	**node A**: 0 × weight = 0 × 2 = 0
	**node B**: 100 × weight = 100 × 2 = 200
	**node C**: 100 × weight = 100 × 2 = 200

Важно заметить, что наличие label'ов не оценивается при планировании с данным профилем планирования, т.к. в этапе отсутствует плагин, рассчитывающих `scoring` по факту наличия необходимых label'ов на node. 

Итоговые результаты следующие:
	**node A**: 100 + 0 = 100
	**node B**: 60 + 200 = <mark style="background: #BBFABBA6;">260</mark>
	**node C**: 20 + 200 = 220
Наибольший `scoring` получила **node B**, а значит pod будет назначен данной node'е 

#### Как Affinity работает при планировании?

В Scheduling Framework Affinity работает на двух этапах:
1. **Filter** — для жестких правил (`requiredDuringScheduling`)
   *отфильтровывает всех неугодных*
2. **Score** — для мягких правил (`preferredDuringScheduling`)
   *позволяет неугодным поучаствовать в расчете scoring'а*

За соответствие требованиям указанным в Affinity отвечают Affinity-плагины: `NodeAffinity` и `InterPodAffinity`.

##### Пример:
Допустим, у нас есть кластер с тремя node'ами:

| **Node** |    **Labels**    | **CPU (free)** |       **Running pods**        |
| :------: | :--------------: | :------------: | :---------------------------: |
|  node1   | zone=A, env=prod |     8 CPU      | `frontend-abc` (app=frontend) |
|  node2   | zone=B, env=prod |     4 CPU      | `frontend-xyz` (app=frontend) |
|  node3   | zone=A, env=dev  |     16 CPU     |  `backend-123` (app=backend)  |
И мы хотим назначить одной из них pod с такой конфигурацией:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
    tier: frontend
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:  # Жесткое правило (Filter)
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - A
      preferredDuringSchedulingIgnoredDuringExecution:  # Мягкое правило (Score)
      - weight: 100
        preference:
          matchExpressions:
          - key: env
            operator: In
            values:
            - prod
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:  # Мягкое правило (Score)
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - frontend
          topologyKey: kubernetes.io/hostname
  containers:
  - name: app
    image: nginx
```

Изначально scheduler фильтрует подходящие для расчета scoring'а node'ы, опираясь на строгие правила. Как видно из примера, `requiredDuringSchedulingIgnoredDuringExecution` указывает на строгую необходимость при планировании назначать pod только тем node, у которых существует **label "zone: A"**. Следовательно, node2 будет отфильтрована и не будет участвовать в подсчете scoring'а на этапе `Score`.

Теперь scheduler будет подсчитывать баллы для "мягких" Affinity-правил:
1. Правило `preferredDuringSchedulingIgnoredDuringExecution` для `nodeAffinity`, ожидающее наличие **label "env: prod"** (плагин `NodeAffinity`):
	- **node1**: `env=prod` ✅ (совпадает) → 100 баллов × weight = 100×100 = **10000**
	- **node3**: `env=dev` ❌ (не совпадает) → 0 баллов × weight = 0×100 = **0**
2. Правило `preferredDuringSchedulingIgnoredDuringExecution` для `podAffinity`, нацеленное на установку текущего pod на node имеющую уже запущенный pod с **label "app: frontend"** (плагин `InterPodAffinity`):
	- **node1**: есть под с необходимым label (app=frontend) ✅ → 100 баллов × вес 50 = **5000**
	- **node3**: нет запущенного pod с подходящим label ❌ → 0 баллов × вес 50 = **0**
3. Суммирование баллов:
	**node1**: 10000 + 5000 = 15000
	**node3**: 0 + 0 = 0

Исходя из общего scoring'а, pod будет назначен node'е с номером 1.

Планирование опирающееся на **Taints/Tolerations** использует отдельный плагин `TaintToleration`. Он также может быть использован для жесткого отсеивания на этапе `Filter`, а также участвовать в подсчете баллов на этапе `Score`(если включен scoring для `Tolerations`).