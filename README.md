# Scheduling with Apache Airflow
#   <span style="color:blue;font-weight:bold">Technical Architecture</span>
---
##   <span style="color:blue;font-weight:bold">Solution overview</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Apache Airflow is an open-source platform for programmatically authoring, scheduling, and monitoring workflows. It allows to organize, execute, and monitor sequences of tasks called **Directed Acyclic Graphs (DAGs)**, which describe how tasks relate to each other and in what order they should be executed.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Airflow is written in Python, which allows for rapid development and deployment of workflows. It is highly scalable and extensible, enabling to define operators (custom logic for task execution) and executors (mechanisms to handle task execution environments) according to the needs of tasks.

_**Diagram : Apache Airflow components**_

![image](https://github.com/user-attachments/assets/39de4aeb-2ad0-404b-a9f4-dd4aba15d14c)



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Airflow's design is based on a set of core components that work together to provide workflow management system (_Diagram : Apache Airflow components_). 
Below is a detailed description of its components, their roles, and dependencies:


- [ ] **Web Server**

_**Description**_: The Web Server is a Flask-based application providing the Airflow Web UI. It allows inspect, trigger, and debug DAG runs and task instances, as well as manage and monitor the Airflow environment.
_**Role**_: Serves as the interface for users to interact with the Airflow environment.
_**Dependencies**_: It interacts with the metadata database to retrieve and display information about DAGs, tasks, and their execution status.

- [ ] **Scheduler**

_**Description**_: The Scheduler is the component responsible for scheduling tasks. It decides when and where to run tasks based on the code definitions of the DAGs, considering dependencies and scheduling parameters.
_**Role**_: Monitors all tasks and DAGs to trigger task instances whose dependencies have been met.
_**Dependencies**_: Depends on the metadata database to track the state and execution of tasks. It also places tasks in the queue to be picked up by workers.

- [ ] **Metadata Database**

_**Description**_: This is the backend that stores state and metadata for Airflow. It includes information about the structure of DAGs, task instance status, execution history, variables, and other crucial data for Airflow's operation. In general for production environments, Postgresql or Mysql are used.
**_Role_**: Acts as the source of truth for the state of all DAGs, tasks, and system metadata.
_**Dependencies**_: Underlies practically all components of Airflow as they interact with the database to retrieve or store information.

- [ ] **Executor**

_**Description**_: Executors are the mechanism through which Airflow decides how and where to run tasks. Executors abstract the execution environment from the scheduler. Different types of executors support different environments and workload distributions.
_**Role**_: Responsible for executing the tasks that the scheduler assigns to them.
_**Dependencies**_: Executors rely on the scheduler to assign tasks and on the metadata database to update the state of task execution. Certain executors (e.g., CeleryExecutor, KubernetesExecutor) also depend on external services (like message brokers or Kubernetes) to manage the task distribution.

- [ ] **Worker**

_**Description**_: Workers are the processes that execute the logic of tasks as defined in the DAGs. In setups using the CeleryExecutor, workers are separate processes running on remote machines.
**_Role_**: Perform the tasks as defined in the DAGs.
**_Dependencies_**: Workers depend on the message queue to receive tasks and on the metadata database to update the state of their execution. Additionally, they need access to the same codebase that defines the DAGs and tasks.

- [ ] **Message Queue (for CeleryExecutor or similar)**

_**Description**_: The message queue is an intermediary for storing messages and is used by distributed executors like CeleryExecutor. It facilitates communication between the scheduler and the workers.
_**Role**_: Temporarily holds messages regarding task execution status and commands between scheduler and workers.
_**Dependencies**_: It is dependent on the scheduler to push task execution commands and on workers to pull these commands for execution.

- [ ] **DAG (Directed Acyclic Graph)**

**_Description_**: A DAG is a Python script that defines a workflow in Airflow with its tasks and dependencies. It's the blueprint of the operations we want to perform.
_**Role**_: Represents workflows by defining a sequence of tasks and dependencies.
**_Dependencies_**: Depends on Airflow's scheduler for execution timing, on operators to define tasks, and potentially on external systems if tasks involve interactions outside of Airflow.


##   <span style="color:blue;font-weight:bold">General Technical choices</span>
  

Here are some key concepts and features that have motivated our choice of Apache Airflow:
1. **Programmatic workflow wreation**: Apache Airflow allows to define workflows as code, which makes them more maintainable, versionable, testable, and collaborative. This enables engineers to manage and track changes using standard version control systems (like Git). [Ref](https://airflow.apache.org/)

1. **Dynamic pipeline generation:** Airflow pipelines are configured as Directed Acyclic Graphs (DAGs), which are flexible and can easily accommodate changes in the workflow as requirements evolve. [Ref](https://airflow.apache.org/docs/apache-airflow/stable/howto/dynamic-dag-generation.html)

1. **Scalability and Extensibility:** Airflow can scale to handle a large number of tasks with varied complexities. It is designed with an extensible architecture, where we can add custom operators, executors, and extend the core library to fit our needs. [Ref](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html)

1. **Rich user interface:** The web-based UI is one of Airflow's strengths, providing insights into the DAG's pipelines, tasks, dependencies, logs, and execution status, which is essential for monitoring and troubleshooting. [Ref](https://airflow.apache.org/docs/apache-airflow/2.0.2/ui.html)

1. **Community and Integrations**: Airflow has a large community that contributes a vast collection of operators for integration with third-party services and systems, such as AWS, GCP, Azure, databases, and more. [Ref](https://airflow.apache.org/docs/apache-airflow/stable/integration.html) 

1. **Scheduling and Triggering:** Airflow has a powerful scheduler that manages the execution of tasks, ensuring they run and retry as configured. It also supports external triggers for workflows. [Ref](https://airflow.apache.org/docs/apache-airflow/1.10.1/scheduler.html) [Ref](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/scheduler.html)

1. **Dynamic parameterization:** Airflow supports parameterization through Jinja templates and can dynamically adapt the workflows based on input parameters. [Ref](https://airflow.apache.org/docs/apache-airflow/stable/howto/dynamic-dag-generation.html)

1. **Role-Based Access Control (RBAC)**: It supports RBAC, enabling organizations to secure access to tasks and workflows. [Ref](https://airflow.apache.org/docs/apache-airflow-providers-fab/stable/auth-manager/access-control.html)

1. **Error handling and retry mechanism**: Airflow provides flexible error handling and automated retry mechanisms, which are crucial for handling failures in any complex workflow.

1. **Support for complex workflows:** Airflow can manage complex dependencies and has the ability to handle conditional task branches, sub-DAGs, and task groups.

1. **Ease of deployment and maintenance:** Airflow’s Docker and Kubernetes Executor support makes it straightforward to deploy and scale in containerized environments.

1. **Workflow parameterization**: With Airflow, we can easily parameterize  workflows, allowing dynamic changes to  DAGs based on external variables or time schedules.


##   <span style="color:blue;font-weight:bold">General Technical Architecture </span>
![Airflow-Technical-Architecture-Diagram5](https://github.com/user-attachments/assets/d9978fad-104d-4226-a309-d023419ed822)

