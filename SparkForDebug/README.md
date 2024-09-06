# Container for Spark Test

## Fast Way to test SparkJobs

- 스파크로 이것저것 실험해보고 싶은데, 굳이 클러스터는 필요없고, 로컬에서 로컬모드로 실행하자니 Java 버전이 안맞는다.

- 굳이 java 버전을 바꾸자니, 원래 있는 패키지들과 의존성이 박살날 수도 있을 것 같아서 제일 빠른 방법을 찾았다 !

### docker-compose.yml

```yml
services:
  spark:
    image: jupyter/pyspark-notebook
    ports:
      - "12345:8888"
      - "4040:4040"
    volumes:
      - ./work:/home/jovyan/work
    environment:
      - SPARK_OPTS=--conf spark.ui.port=4040
      - JUPYTER_TOKEN=pysparkjupyterdlawork9888
```

```sh
docker compose up -d
```

- localhost:12345에서 확인하자