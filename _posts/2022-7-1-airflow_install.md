---
layout: post
title: airflow 실습
---
Airflow는 데이터 처리 일련의 과정을 스케쥴링하고 모니터링 및 관리를 편리하게 도와주는 프레임웍이다.
데이터수집 중 완료, 장애 발생 후 처리 과정을 DAG를 통해 분리하고 flow로 통해서 각 상황에 대처하도록 프그래밍이 가능하다.
기존의 젠킨스잡에 스케쥴로 실행되는 job을 DAG설계를 통해 견고하게 처리되도록 교체하였다.

설치

```
export AIRFLOW_VERSION=2.3.2

export PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"

export CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

설정 파일 생성 

```python
airflow info -> airflow.cfg 파일이 생성됨 
```

database 설정 

```python
#airflow.cfg 
sql_alchemy_conn = mysql+mysqldb://[아이디]:[비밀번호]@#@[아이피]:[포트]/airflow_db?charset=utf8

# SequentialExecutor은 sqllite 환경에서만 사용 가능 wjs
executor = LocalExecutor

#timezone 
default_ui_timezone = Asia/Seoul
```

airflow dag 파일 위치 지정

```python
#airflow.cfg 
[core]
dags_folder = ~/airflow/dags

#example 로딩 안함 
load_examples = False 
```

db 설정 

```jsx
# 데이터베이스 생성 
create database airflow 

# 디비 유저 생성 
create user airflow_user with password 'airflow_user_password'
grant all privileges on database airflow to airflow_user 
```

airflow 실행 

```python
export AIRFLOW_HOME=./airflow
export PYTHONPATH= 'dag source 위치'

# airflow_home을 설정 해야 airflow.cfg파일을 읽어서 처리함 
airflow db init 

# web ui 유저 생성 
airflow users create --username airflow --firstname airflow --lastname airflow --role Admin --email test@test.com

airflow scheduler 
airflow webserver --port 8080 
```

airflow 종료

```jsx
cat $AIRFLOW_HOME/airflow-webserver.pid | xargs kill -9
cat /dev/null >  $AIRFLOW_HOME/airflow-webserver.pid
```

pythonOperatror 작성 

```python
def job1():
        print('start job')
        return 'job return value'

t1 = PythonOperator(
        task_id='preprocess',
        python_callable=job1,
        dag=dag,
    )

def job2(**context):
	param = context["params"]["key"] 

# python 함수 호출 파라메터 전달 
t1 = PythonOperator(
        task_id='function',
				provide_context=True,
				params={'key': 'test''}, 
        python_callable=job2,
        dag=dag,
    )
```

전체코드

```jsx
from airflow import DAG
from airflow.operators.python import PythonOperator
import pendulum

#airflow dags가 pathonpath로 잡히기 떄문에 해당 디렉토리 기준으로 패키지명시 해야됨 
from job.user.connect_user_report import UserReport
from job.user.join_user_report import JoinUserReport

args = {
        'depends_on_past': False,
        }

with DAG(
    dag_id='main',
    default_args=args,
    description='Daily Report Create DAG',
    schedule_interval="@daily",
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=['daily-report'],
) as dag:

    def start():
        print("start job")

    def end():
        print("end job")

    def user_connect_count_reporter():
        user_report = UserReport()
        user_report.run()
        print("end job1")

    def user_join_count_reporter():
        join_user_report = JoinUserReport()
        join_user_report.run()
        print("end job1")

    start_job = PythonOperator(
        task_id='start',
        provide_context=True,
        python_callable=start,
        dag=dag,
    )

    connect_user_job = PythonOperator(
        task_id='connect_user_job',
        python_callable=user_connect_count_reporter,
        dag=dag,
    )

    join_user_job = PythonOperator(
        task_id='join_user_job',
        python_callable=user_join_count_reporter,
        dag=dag,
    )

    end_job = PythonOperator(
        task_id='end',
        provide_context=True,
        python_callable=end,
        dag=dag,
    )

    start_job >> connect_user_job >> join_user_job >> end_job
```

설치중 문제점 

- mariadb연동 실패 airflow db init 중 오류 발생 
마리아 디비는 공식적으로 지원하지 않는다 
[https://github.com/apache/airflow/discussions/15758](https://github.com/apache/airflow/discussions/15758) 관련 이슈
- sqlite C library version too old (< 3.15.0).
    
    ```
    # sqlite 버전 업데이트 
    wget https://www.sqlite.org/2021/sqlite-autoconf-3340100.tar.gz
    tar xvf sqlite-autoconf-3340100.tar.gz
    cd sqlite-autoconf-3340100
    ./configure
    make
    sudo make install
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
    ```
    

airflow 유저  관리

```jsx
airflow user list 
airflow user create
airflow user delete --username XXX 
```

API 사용하기 

```glsl
# 현재 상태 조회 
$ airflow config get-value api auth_backend
airflow.api.auth.backend.deny_all

$ airflow.cfg 설정 변경
auth_backend = airflow.api.auth.backend.basic_auth

$ curl -X GET --user "test:yrdy" http://127.0.1:8080/api/v1/dags/dagRuns

```
