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

##   <span style="color:blue;font-weight:bold">Technical infrastructure</span>

### Environments 

An environment is composed of 2 servers.
Servers must be deployed in different racks, it's preferable to deploy them in different server rooms.
Servers should be deployed in same network as workers.

_**Table: Example of environment**_
| Node | IP Address | Memory |CPU  |  |
|--|--|--|--|--|
| AIRFLOW-SERVER1 | 10.11.12.10  |16  | 2 |  |
| AIRFLOW-SERVER2 | 10.11.12.20 |16  |  2|  |


##   <span style="color:blue;font-weight:bold">Software components</span>
**Linux** : Any Linux 8.x 
**Python** : Airflow is written in Python language. All these Python version are compatible with Airflow 2.9.2 : Python: 3.8, 3.9, 3.10, 3.11, 3.12 [(Ref)](https://airflow.apache.org/docs/apache-airflow/2.9.1/installation/prerequisites.html)
**Apache Airflow** : Installed with pip Python tool.  Installed components : Webserver, Scheduler, Celery Executor, Flower UI
**HAproxy** : As load balancer to Webserver and Flower UI (Uses of HAproxy provided by CLoud Services)
**Postgresql** : As as metadata database
**Redis** : As message broker


|Software  | Version |Comment |
|--|--|--|
|Linux | Linux 8|
| Python | 3.9 | Python is installed in a vitual environment |
| Airflow | 2.10.1 |The latest release so far (3<sup>rd</sup> june'24) [Ref](https://airflow.apache.org/announcements/)|
|HAproxy | 2.9.x |  |
| Postgresql | 14.x | Seems the most stable release |
| Redis community | 7.4-rc |  |




##  <span style="color:blue;font-weight:bold">Robustness of the solution</span>

###  <span style="color:blue;font-weight:bold">High availability and fault tolerance</span>

#### **Database Backend** 
---
Use of highly available database for the Airflow metadata could be the best choice, it can provide rplication and automatic failover.

There are several solutions and architectures designed to achieve HA for PostgreSQL, each with its own set of features, complexities, and trade-offs.
> We chose  Postgresql **"Streaming Replication"** for its relative simplicity of deployment. **Warm standby mode** and **asynchronous** replication are selected (Implementation document coming soon). **Failover and failback are operated manually.**

**_Diagram: Postgresql Streaming Replication_**
![Postgresql-Streaming-Replication4.jpg](/.attachments/Postgresql-Streaming-Replication4-6e618a8c-0332-40ed-a96e-a0cd8cf270e9.jpg =420x)


 PostgreSQL uses a mechanism called **Write-Ahead Logging** (**WAL**) to ensure data integrity. Whenever changes are made to the data, entries are first written to a WAL file before the changes are applied to the actual database. This ensures that in case of a crash, the database can recover by replaying the WAL entries.

In the streaming replication, the **primary server outputs** its WAL records continuously to a WAL archive, which standby server(s) can then access. 

The standby server(s) is set up in either **hot standby mode**, which allows it to also handle read-only queries, or **warm standby mode**, which doesn't allow access until the primary server fails.

Once the standby server(s) is configured and the base backup is in place, **WAL records are streamed from the primary to standby(s) in real-time**. The standby server(s) continuously apply these WAL records as soon as they arrive to keep up with the changes happening on the primary server.

PostgreSQL supports both **synchronous** and **asynchronous** replication. In synchronous replication, a transaction is not considered complete on the primary until it has been confirmed written to the WAL on both the primary and standby server(s). This provides strong guarantees of data consistency but can affect performance. In asynchronous replication, there is a potential for data loss if the primary fails before the standby has received all changes, but it provides better performance since transactions are considered complete as soon as the WAL is written on the primary.


#### **Webserver Redundancy** 
---
We install multiple (at least 2) webservers on different physical servers, and we load balance the these nodes with HAproxy. This ensure that the UI and scheduling capabilities are always available.

####**Scheduler Redundancy**
> We deploy multiple schedulers (at least 2)([Ref](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/scheduler.html#running-more-than-one-scheduler)). Running Apache Airflow with multiple schedulers can help improve the overall performance and resilience of Airflow setup, especially as the number of tasks and workflows (DAGs) grows.

To run Airflow with multiple schedulers, we need to use Airflow 2.x and  a database backend that supports multiple schedulers, such as PostgreSQL( or MySQL).


#### **Redis replication**
---
The replication method used in Redis is the basic Primary-Replica or Leader-Follower replication. This setup consists of a single master Redis instance and one or more follower instances. 

> The chosen architecture is : one master and one follower, with asynchronous replication. Failover and failback are operated manually.

**_Diagram: Redis Replication_**
![Redis-Cluster.jpg](/.attachments/Redis-Cluster-fbd8b15f-5e24-430a-a060-7b71ab8cc030.jpg)


###   <span style="color:blue;font-weight:bold">Scalability and Performance</span>
1. **Executor Choice:** Choose the right executor (e.g., CeleryExecutor, KubernetesExecutor) based on the workload characteristics and scalability needs.
1. **Resource Optimization:** Use resource management features to allocate CPU and memory resources based on the task requirements.
1. **Distribute Workloads:** Ensure that the workload is evenly distributed across the available workers to prevent bottlenecks and improve execution time.
##   <span style="color:blue;font-weight:bold">Disaster recovery plan (DRP)</span>
Include it in the global DRP if it exists.

# <span style="color:blue;font-weight:bold">Security</span>

##   <span style="color:blue;font-weight:bold">Authentication</span>

### Local authentication
Airflow implement a default local authentication. At the installation phase, a superuser is created and some default roles :
Admin
Viewer
User
Op
Public

Superuser can create users and roles, and can also assign roles to users.
### OpenID Connect authentication
Coming in the second release of the project.
### RBAC / Multi-tenancy
![AIrflow-work6.jpg](/.attachments/AIrflow-work6-89ebf440-e795-4b0e-8741-78d3a848445b.jpg)



##   <span style="color:blue;font-weight:bold">Service accounts</span>

##   <span style="color:blue;font-weight:bold">API Access</span>
### Apache Airflow API overview
Since Apache Airflow 2.0 (December 2020), a significant overhaul of the platform of the API was realized. One of the standout features of Airflow 2.0 was the introduction of a Stable REST API (also known as the Stable API or Airflow 2.0 API).
**Stable REST API**: This new API was designed to be comprehensive, covering a wide range of functionalities needed to interact with Airflow programmatically. It adhered to REST principles and included endpoints for DAG management, task instances, variable management, connection management, and more. The API also included improved authentication and authorization mechanisms.

**OpenAPI Specification**: The Stable API followed the OpenAPI Specification (OAS), making it easier for users to understand and integrate with Airflow programmatically. This made it easier to generate client libraries in various programming languages.


![Airflow-API2.jpg](/.attachments/Airflow-API2-0111fb1a-2447-4fee-b02f-49c9a3027514.jpg)

##   <span style="color:blue;font-weight:bold">Secrets Backend</span> 
Second phase of the project
Open source Hashicorp Vault to store passwords ?? 

##   <span style="color:blue;font-weight:bold">Technical flows matrix</span>

| Flow ID |Flow status  |Source block  |Source IP  |Target block  |Target IP  |Target port  | Protocol |  |  |  |
|--|--|--|--|--|--|--|--|--|--|--|
|  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |



# <span style="color:blue;font-weight:bold">Deployment and operation</span>

##   <span style="color:blue;font-weight:bold">Build</span>
TBD
##   <span style="color:blue;font-weight:bold">Monitoring</span>
- Monitor URLs : 
1. Airflow Webserver UI 
1. Airflow Flower UI 
- Monitor Postgresql server on the primary node                
- Monitor Postgresql streaming replication
- Regularly monitor your Redis instances to check their health and replication status.
- Monitor Airflow scheduler process on both nodes of the server platform
- Monitor Airflow webserver process on both nodes of the server platform
- Monitor Airflow celery worker process on all Airflow workers



