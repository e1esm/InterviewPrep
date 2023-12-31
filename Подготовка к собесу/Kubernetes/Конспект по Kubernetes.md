### Контейнеры vs Виртуальные машины
Контейнеры - это сущность, которая содержит все необходимые для запуска приложения зависимости.

Основное отличие в том, что виртуальные машины виртуализируют весь компьютер/сервер вплоть до аппаратного уровня. На виртуальную машину можно поставить любую гостевую ОС, и она может быть отличной от ОС хоста.

Контейнеры виртуализируют компоненты, находящиеся выше уровня операционной системы. Они делят друг с другом ядро ОС, установленной на сервере, благодаря чему они занимают гораздо меньше ресурсов и быстрее запускаются, но при этом изолированность контейнеров ниже, чем у ВМ.


### Stateful vs Stateless:
Stateful приложение - это приложение, сохраняющее при работе данные внутри себя - сессии пользователей, которые хранятся на сервере, например.
Такие приложения сложнее масштабировать горизонтально - чтобы развернуть несколько экземпляров - нужно переносить состояния на новые машины и синхронизировать их.

Stateless приложение - любой запрос к приложению уникален, а его ответ не зависит от какого-либо состояния приложения. Stateless-приложения легко масштабируются горизонтально, упрощают автоматизированное тестирование, так как нет состояния, которое нужно воспроизводить.


### Docker container и runtime - разница.
Есть Docker - стандарт, по которому описывается контейнер, а есть Docker-движок - runtime, что запускает контейнер.

В Kubernetes засчет его Container Runtime Interface (CRI) API в контейнерах можно запускать разные runtime, например CRI-O, Containerd.

Docker-runtime старше, чем Kubernetes - он не отвечает стандартам CRI -> Docker runtime не поддерживается в Kubernetes.



### Kubernetes:
Kubernetes - это open-source платформа для автоматизированного запуска, масштабирования и управления контейнеризированными приложениями.

С помощью него можно:
- запускать приложение в контейнере на нескольких серверах/площадках. Если приложения работают на 2-3 серверах - можно обойтись без Kubernetes, но если их десятки и сотни - дальнейшее масштабирование будет сложнее без инструмента оркестрации, которым предстоит Kubernetes.
- автоматически развертывать, апдейтить, откатывать назад обновления и управлять состоянием контейнеров.
- управлять нагрузкой и оперативно масштабироваться в большую или меньшую сторону


### Соотношение Kubernetes и Docker:
Docker - стандарт контейнеризации.
Kubernetes помогает настраивать сетевую связанность Docker-контейнеров, запущенных на разных хостах и оркестрировать их.
`Docker - контейнер, Kubernetes - платформа для управления контейнерами и их оркестрации.`

### Компоненты архитектуры Kubernetes:
- Master ноды - координируют все активности кластера. Распределяют и резервируют ресурсы, управляют состоянием контейнеров, масштабируют и раскатывают обновления. Компоненты мастер-ноды:
	- kube-apiserver - точка входа в панель управления master-ноды. Отвечает за взаимодействие между master и worker-нодами, отслеживает состояние worker-узлов и оповещает master о важных изменениях.
	- kube-scheduler - отвечает за распределение нагрузки на рабочие узлы, постоянно отслеживая сколько ресурсов доступно в момент X и сколько из них задейстованы под нагрузку в каждом узле. Решает, на каком узле запускать новый Pod.
	- Controller Manager - отвечает за разработку контроллеров: Deployment, ReplicaSet, StatefulSets, DaemonSet, Jobs, CronJob
	- ETCD: хранит информацию о настройках и состоянии кластера, его метаданные. Представляет собой распределенную базу данных в формате ключ-значение.
- Nodes (workers) - рабочие узлы в кластере. На них запускаются поды с контейнерами. На каждой worker-ноде Kubernetes работают:
	- kubelet - процесс, который запускает, удаляет и обновляет поды с контейнерами
	- kube-proxy - конфигурирует правила сети на рабочих узлах
![[Pasted image 20230922182418.png]]


### Pod
Под - это самая маленькая сущность Kubernetes, в которой запускаются контейнеры. Контейнеров внутри одного пода может быть несколько.
Помимо контейнеров, у каждого пода есть:
- Уникальный IP-адрес, который позволяет подам общаться друг с другом
- Хранилище PV
- Данные конфигурации, которые определяют, как контейнер должен запускаться.
![[Pasted image 20230922182550.png]]

### Namespaces:
Пространства имен позволяют разделять кластер на виртуальные кластеры, в которых можно объединять приложения в изолированные  группы по нужному принципу. Это позволяет создать приложение в двух разных неймспейсах с одинаковым именем.
Группировка по такому принципу упрощает работу с k8s. В одном пространстве можно разместить приложение мониторинга, в другом - приложения, связанные с ИБ и.т.д.

### ReplicaSet:
Задача RS - поддерживать работу определенного количества экземпляров подов в кластере Kubernnetes. Используется для запуска Stateless-приложений. RS используется для обеспечения доступности приложения. Если какие-то из подов покрашатся, то кубер автоматически с помощью RS запустит новые экземпляры подов, чтобы заменить вышедшие из строя. Без RS пришлось бы их запускать вручную.


### Deployment
Deployment - абстракция более высокого уровня относительно RS. Deployment помогает делать декларативные апдейты подов, используя ReplicaSet. Когда для группы контейнеров нужно обновить версии или откатиться к предыдущей - используется Deployment

### StatefulSet:
StatefulSet управляет развертыванием и масштабированием группы подов, давая возможность сохранять состояния и характеристики подов.
Если нужно, чтобы поды запускались в определенном порядке на тех же нодах, чтобы при каждом запуске у каждого было хранилище PVC или специальные сетевые идентификаторы - используется StatefulSet.


### DaemonSet:
Используется, когда надо запустить один или несколько подов на всех рабочих узлах кластера. При запуске новых нод разработчику не потребуется вручную запускать поды, которые долны там быть для каких-то служебных задач. Например, можно запустить Prometheus Node Exporter для мониторинга, collect, logstash и.т.д.



### Работа с хранилищем:
У Kubernetes есть volumes. Нативный - emptydir. Часть из них - stateless, т.е. они живут пока жив под. 
Для Stateful приложений используются постоянные хранилища - Persistent Volumes (PV). PV - это единицы хранения, которые были выделены кластеру Kubernetes его администратором. Это могут быть локальные диски, СХД, внешние дисковые полки. Никак не зависят от жизненного цикла подов.

Persistent Volume Claim (PVC) - это запрос на выделение PV определенных характеристик - тип хранилища, объем хранилища, типа доступа. Для описания подробных характеристик доступных PV используются Storage Classes.
![[Pasted image 20230922235716.png]]


### Способы предоставления трафика к подам по сети:
Для этого требуются сервисы (Services)
- ClusterIP - сущность, позволяющая маршрутизировать запросы к подам на статичный IP-адрес. Благодаря ClusterIP у нас будет неизменная точка входа, даже если поды будут крашиться и восстанавливаться снова.
- NodePort - делает сервис доступным извне через статический порт на каждом узле кластера. Любой трафик, отправленный на этот порт будет перенаправлен на сервис. При этом ClusterIP создается автоматически.
- LoadBalancer - публикует сервис вовне и заводит трафик от балансировщика облачного провайдера внутрь кластера.
- External name - сопоставляет сервис с DNS-именем(например, example.com). Создает CNAME-запись, которая соединяет DNS-имя с определенным именем внутри кластера. Выступает кк прокси, которое позволяет пользователю перенаправлять запросы сервису, находящемуся внутри или за пределами кластера.


### Ingress:
Ingress позволяет настраивать правила маршрутизации для трафика от внешних источников до сервисов внутри кластера.
В Ingress описываются сами правила маршрутизации к сетевым сервисам, а контроллер Ingress отвечает за их выполнение. Контроллер не поставляется в Kubernetes, но можно использовать одно из сторонних решений.


### Запуск приложений в Kubernetes с помощью kubectl:
1) Приложение должно быть упаковано в контейнер, поэтому сначала нужно будет поместить его туда
2) Нужно запустить контейнер в виде набора реплик (подов) - используем Deployment
3) Для того, чтобы приложение было доступно в интернете и к нему можно было подключиться - нужно настроить сервис LoadBalancer - который позволяет присвоить публичный IP-адрес и подключиться к кластеру из внешней сети.
4) Чтобы маршрутизировать пришедший через балансировщик трафик до приложения, в кластере должен быть создан Ingress, описывающий правила маршрутизации и запущен контроллер.

Проделать все это можно через kubectl, используя командную строку. Это императивный и самый простой способ.
Второй способ, используемый в пром индустрии - это управление через декларативные манифесты, в которых мы описываем желаемое состояние, а k8s сам решает, какие действия для этого нужно сделать. 



### Дебаггинг приложения:
Причины по которым приложение не работает:
- Под отсутствует
- Под не запускается (Pending)
- Под запускается, но с ошибкой (CrashLoopBackOff)
- Под работает (Running), но недоступен по сути

1) Нужно убедиться, что манифест действительно выполнился и все объекты были созданы
2) Если поды находятся в статусе Pending - Scheduler не может найти подходящую ноду для запуска. Возможные причины: недостаток ресурсов, невозможность скачивания образа и т.д. Найти причину помогут события, связанные с подом.
3) Если под был назначен ноде, но при запуске возникла ошибка (CrashLoopBackOff) - кластер будет предпринимать попытки по повторному запуску. Обычно это происходит из-за ошибки в самом приложении, а найти причину помогут логи.
4) Если поды недоступны по сети из других подов - проверка Service на необходимый Selector, а также и проверка на одинаковый namespace.

Если сервисы существуют, а доступа нет - возможная ошибка: NetworkPolicy, ограничивающий доступ сервисов друг к другу.
Если сервис должен быть доступным извне кластера - необходимо проверить наличие Ingress правила, описывающего маршрутизацию L-7 трафика на выбранный сервис.

Также, для прохождения запроса внутрь кластера должна существовать точка входа, через который запрос попадает на Ingress Controller - Service NodePort для кластеров on-premise инфраструктуры и LoadBalancer для кластеров в облаке.



### RBAC в Kubernetes:
RBAC (Role-Based access control) - это система распределения прав доступа к различным объектам в кластере Kubernetes.
Объекты в кластере k8s - это YAML-манифесты, а права доступа определяют, какому пользователю можно только просматривать манифесты, а кто может их создавать, изменять или даже удалять.

Пользователи кластера k8s - это все, кто шлет запросы в API-сервер, не только администраторы и разработчики, но и различные скрипты CI/CD, компоненты control plane, kubelet, kube-proxy на узлах, а также приложения, запущенные в кластере.
![[Pasted image 20230923194845.png]]

### ServiceAccount:
Самый простой тип пользователей - ServiceAccount. Kubernetes ничего не знает о пользователях в том виде, в котором мы привыкли их видеть в других системах ограничения доступа, где у пользователей есть логин, пароль, группы и т.д. Внутри своих объектов k8s никаких списков логинов, паролей, групп не хранит, но при этом он обладает механизмами вызова внешних сервисов проверки паролей, есть вариант проверки пользовательского сертификата.

ServiceAccount - был создан в первую очередь для ограничения прав ПО, работающего в кластере. Все общение идет через запросы к API-серверу, и каждый такой запрос авторизируется специальным JWT-токеном, который автоматически генерируется при создании объекта типа ServiceAccount и кладется в secret.

Этот Secret монтируется внутрь контейнера, где работает наше приложение и если оно знает и умеет в k8s - оно берет токен из файлы, смонтированного из secret и с ним делает запросы в API.

kubelet монтирует токен внутрь контейнеров, затем приложение решает - если ему нужно обращаться к API кластера - будет ли оно брать токен из смонтированного секрета или возьмет другой токен, например, запросит у пользователя.


### Role:
Для того, чтобы сервис аккаунту дать права - используются две сущности: Role и RoleBinding.

Role - это YAML-манифест, который описывает набор прав на объекты кластера K8S. Role ничего и никому не разрешает - это список.

Объекты кластера - YAML-манифесты - которые хранятся в etcd. все права проверяются API-сервером и относятся к запросам, которые принимает API-сервер.

Если разработчику было запрещено выполнять kubectl exec, но у него есть доступ на воркер-ноду, то зайти на нее и сделать docker exec в контейнер RBAC запретить никак не сможет.

В манифесте Role есть раздел Rules - список правил, описывающих права досутпа.
В каждом правиле есть следующие параметры:
![[Pasted image 20230923200425.png]]
- apiGroups - описывает API-группу манифеста - то, что написано в поле apiVersion до слеша. Если в apiVersion указана только версия, без группы, например как в манифесте пода - у манифеста корневая группа (core-group). В роли корневая группа указывается как пустая строка.
- resources - список ресурсов, к которым мы описываем доступ.
- verbs - список действий, которые можно сделать с ресурсами, описанными выше - получить, посмотреть список, следить за изменением, отредактировать, удалить и т.п.
- resourceNames - с помощью этого поля можно указать название объекта и например дать права на изменение не всех configmap, а только одного конкретного с именем X.

Role - это просто список правил, объединенных в одну абстракцию, никаких прав он не дает, для этого служит следующий объект.

### RoleBinding:
Манифест RoleBinding:
![[Pasted image 20230923201330.png]]
- roleref - права, из какой роли будут разрешены
- subjects - кому будут разрешены эти права

Таким образом, API-сервер кластера при получении HTTP-запроса на действие с объектом кластера увидит в заголовке JWT-токен. Проверит что токен подписан корневым сертификатом кластера, возьмет из него название сервис аккаунта и неймсмейс, провверит соответствующие RoleBinding в кластере, чтобы узнать, разрешено ли производить запрошенное действие над объектом из запроса.



### ClusterRole:
Role описывает права в неймспейсе, сущность Role неймспейс-зависимая и мы можем создавать роли с одинаковым именем в разных неймспейсах.

Cluster Role - это кластерный объект, сущность, описывающая права на объекты во всем кластере.

Стандартные кластерные роли:
- admin
- edit
- view
Эти права разрешают администрирование, редактирование и просмотр сущностей.


### ClusterRoleBinding:
RoleBinding - дает доступ только к тем сущностям, которые находятся в том же неймспейсе, что и манифест RoleBinding. ClusterRoleBinding позволяет выдать доступ к сущностям во всех неймспейсах кластера сразу.



### RBAC: обобщенно
Role в неймспейсах привязывается через RoleBinding в неймспейсе - юзер получает определенные права исключительно в пределах неймспейса.
ClusterRole привязывается через ClusterRoleBinding - юзер получает определенные права на весь кластер.