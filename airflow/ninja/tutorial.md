# Airflow-Tutorial

<details>

<summary>Properties</summary>

:pencil:2024.08.03

참고 블로그:
- [Apache Airflow, 어렵지 않게 시작하기](https://gngsn.tistory.com/264)
- [오늘의 집, 에어플로우 도입기](https://www.bucketplace.com/post/2021-04-13-%EB%B2%84%ED%82%B7%ED%94%8C%EB%A0%88%EC%9D%B4%EC%8A%A4-airflow-%EB%8F%84%EC%9E%85%EA%B8%B0/)

</details>

## Easy to start

### Docker-Compose Setting

1. 에어플로우 프로젝트 관리를 위한 디렉터리 생성
2. 도커 컴포즈 파일 다운로드

```
mkdir airflow_project && cd airflow_project

~/airflow-local$ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.3/docker-compose.yaml'
```

> Docker-compose 파일 구성

- airflow-scheduler
    - Task를 일정 주기마다 실행 할 수 있도록 DAG 파일을 모니터링 및 관리

- airflow-worker
    - Scheduler의 Queue에 등록된 Task가 실행 단계로 전달 될 시 실행

- airflow-webserver
    - UI Dashboard, realtime-log, click-to-run 등 웹 서버에서 모니터링 및 단발성 이벤트 실행 또는 삭제 가능

- airflow-trigger
    - 특정 스케줄러에 의해 동작 하지 않고 이벤트를 감지 하여 실행

- airflow-init
    - 에어플로우 프로젝트 초기 설정

- postgres
    - 로컬에서 airflow db는 sqlite3를 사용 하지만 docker 환경에서는 postgres 사용

- redis
    - 스케줄러에서 워커로 메세지 전송 하는 브로커

- common environment
    - airflow.cfg에 설정 하는 내용


### Create airflow refference directory

1. 도커 환경에서 참조 할 추가 디렉터리 생성

```
mkdir -p ./dags ./logs ./plugins ./config
```

- dags : DAG 파일 보관
- logs : Task 실행 시, 혹은 Scheduler의 로그 보관
- config : 커스텀 log parser를 추가하거나 Cluster 정책을 위한 airflow_local_settings.py를 추가할 수 있음
- plugins : 커스텀 Plugin 보관

### Set up .env

1. 환경 변수 관리

```
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

> Airflow docker compose guide to can modify values in the environment

```python
# Basic Airflow cluster configuration for CeleryExecutor with Redis and PostgreSQL.
#
# WARNING: This configuration is for local development. Do not use it in a production deployment.
#
# This configuration supports basic configuration using environment variables or an .env file
# The following variables are supported:
#
# AIRFLOW_IMAGE_NAME           - Docker image name used to run Airflow.
#                                Default: apache/airflow:2.9.3
# AIRFLOW_UID                  - User ID in Airflow containers
#                                Default: 50000
# AIRFLOW_PROJ_DIR             - Base path to which all the files will be volumed.
#                                Default: .
# Those configurations are useful mostly in case of standalone testing/running Airflow in test/try-out mode
#
# _AIRFLOW_WWW_USER_USERNAME   - Username for the administrator account (if requested).
#                                Default: airflow
# _AIRFLOW_WWW_USER_PASSWORD   - Password for the administrator account (if requested).
#                                Default: airflow
# _PIP_ADDITIONAL_REQUIREMENTS - Additional PIP requirements to add when starting all containers.
#                                Use this option ONLY for quick checks. Installing requirements at container
#                                startup is done EVERY TIME the service is started.
#                                A better way is to build a custom image or extend the official image
#                                as described in https://airflow.apache.org/docs/docker-stack/build.html.
#                                Default: ''
#
# Feel free to modify this file to suit your needs.
```

---


### Customizing Options

1. 도커 컴포즈 파일의 옵션 값 설정

- Outbound ports change

    ```yaml
    ports: - "9090:8080"
    ```

- Does not load example cases

    ```yaml
    AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    ```

- Change the task instance log url

    ```yaml
    x-airflow-common:
        AIRFLOW__WEBSERVER__BASE_URL: "http://127.0.0.1:9090"
    ```

### Airflow proejct init

```
docker compose up airflow-init
```

> Troubleshooting:

1. No such Image redis:7.2-bookworm

- docker compose up airflow-init 재실행
    - 도커 허브에서 지원하는 버전이 없었거나 내용이 달라졌을 것이라 생각 할 수 있지만 도커 허브에는 잘 등록 되어 있으니 다시 실행 하면 정상적으로 풀링 가능


### Commands

- DAG list
    ```
    docker compose run airflow-worker airflow dags list
    ```

- Tasks list
    ```
    docker compose run airflow-worker airflow tasks list <dag_id>
    ```

- Test
    ```
    docker compose run airflow-worker airflow dags test "print-context"
    ```

- DAG file refresh
    ```
    docker compose restart airflow-scheduler
    ```

- DAG Trigger
    ```
    docker compose run airflow-worker airflow dags trigger <dag_id>
    ```

### Example usage

"Data Pipelines with Apache Airflow" into hans-on code:

```python
import pendulum
import urllib.request

from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.operators.postgres import PostgresOperator


dag_args = dict(
    dag_id="stocksense",
    start_date=pendulum.now().subtract(days=1),
    template_searchpath="/tmp/",
)


def _get_data(output_path, **context):
    execution_date = context["logical_date"]
    adjusted_date = execution_date.subtract(hours=2)

    year = adjusted_date.year
    month = adjusted_date.month
    day = adjusted_date.day
    hour = adjusted_date.hour

    url = (
        "https://dumps.wikimedia.org/other/pageviews/"
        f"{year}/{year}-{month:0>2}/pageviews-{year}{month:0>2}{day:0>2}-{hour:0>2}0000.gz"
    )

    print(f"url: {url}")
    urllib.request.urlretrieve(url, output_path)
    print("Data fetched successfully")


def _fetch_pageviews(pagenames, execution_date, **_):
    result = dict.fromkeys(pagenames, 0)

    with open("/tmp/wikipageviews", "r") as f:
        for line in f:
            domain_code, page_title, view_counts, _ = line.split(" ")

            if domain_code == "en" and page_title in pagenames:
                result[page_title] = view_counts

    with open("/tmp/postgres_query.sql", "w") as f:
        for pagename, pageviewcount in result.items():
            f.write(
                "INSERT INTO page_view_counts VALUES "
                f"('{pagename}', {pageviewcount}, '{execution_date}')"
            )

    print(result)


with DAG(**dag_args) as dag:
    get_data = PythonOperator(
        task_id="get_data",
        python_callable=_get_data,
        op_kwargs={
            "output_path": "/tmp/wikipageviews.gz",
        }
    )

    extract_gz = BashOperator(
        task_id="extract_gz",
        bash_command="gunzip --force /tmp/wikipageviews.gz",
    )

    fetch_pageviews = PythonOperator(
        task_id="fetch_pageviews",
        python_callable=_fetch_pageviews,
        op_kwargs={
            "pagenames": [
                "Google",
                "Amazon",
                "Apple",
                "Microsoft",
                "Facebook",
            ],
        }
    )

    write_to_postgres = PostgresOperator(
        task_id="write_to_postgres",
        postgres_conn_id="my_postgres",
        sql="/tmp/postgres_query.sql",
    )

    get_data >> extract_gz >> fetch_pageviews >> write_to_postgres

```