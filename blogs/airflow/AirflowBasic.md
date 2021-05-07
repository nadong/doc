# Airflow 核心概念
## DAGs & DAG Runs
DAG(Directed Acyclic Graph)即有向无环图， 包括一系列任务（TASK）的逻辑工作流。可以定义TASK 之间的依赖关系（拓扑），调度， 失败机制，所有者等。DAG Run 是DAGs的一次执行， 以execution date 为主键分开。

## Operator & TASK & Task Instances
DAG 主要定义了工作流如何运行， 实际的工作是由Operator来完成。Operator 用于描述工作流中的单个任务。DAG 保证Operator的执行顺序和依赖，Operators 可以独立运行，并且可以运行在不同/相同的机器上。那么问题来了，Operator之间如何共享信息呢？ 一般的return 是不能够在Operator 之间传递的，这时候需要的是XCom。Airflow支持多种Operators和定制化Operator，具体介绍会在接下来部分详述。
Operator被实例化后，就是一个Task。实例化过程中会传递T具体的任务定义，参数等，成为DAG中的一个节点，并且task实例会有一个运行状态，可以是running， success， failed，rerun 等等。Task Instances 是指Task 的一次执行实例。

## Trigger Rules
依赖项是指task 之间的依赖关系。一般情况下，所有直接上游的task 执行成功，就可以出发下游task 执行，但是Airflow允许更复杂的依赖项设置。 所有的Operator 都要一个trigger rule的参数。参数可以为all_success（default）， all_fail， one_success ， 或者one_fail等。有一点需要注意的是，这些参数可以喝depends_on_pass 共同使用。
  * Trigger rule:
    * ALL_SCCESS/ALL_FAILED/ALL_DONE:
    * ONE_SUCCESS/ONE_FAILED:
    * NONE_FAILED/NONE_FAILED_OR_SKIPPED/NONE_SKIPPED:
    * DUMMY

## Queue & Pool
Queue 是BaseOperator的一个属性，用来执行发送任务的队列。因此任何task可以分配给任何队列。Airflow Worker可以指定监听的任务队列。所以通过queue的设置，可以指定worker 的类型。比如一个pipeline 有需要在EC2 上执行或者在EMR上执行的， 定义不同的queue，task 就会找到相应的机器执行。
Pool 可以定义任何task 的并发，可以在Airlow UI上设置。比如一台EMR 资源只能限制一个task 执行，就可以保证在EMR 上的task 只有一个并发。pool参数可和priority_weight结合使用，以定义队列中的优先级，默认情况下，priority_weight 是来自所有下游任务的priority_weight值相加。

## Schedule & Trigger
DAG 可以定义有schedule 和No Schedule 两种方式。 设置schedule的情况下， 有几个参数定义 start_date&end_date和schedule interval。No Schedule的方式， 将DAG的schedule 设置为None， 这种情况是external trigger DAG。

## XComs & Variables
Xcoms专门用于task之间的通信，以key，value的形式定义，用于task之间传递变量。通过xcom_push 推送，通过xcom_pull并且指定task_id来拉取.
Variables 用于global设置，可以是任意内容或者配置key,value形式。可以在代码或者UI 中定义，通过Variable.get()来读取。
