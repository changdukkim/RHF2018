# RHF 2018 Seoul - OCP Demo



## 1. Image & Git S2I Build



> Front-end <-> Back-end 로 구성된 2-Tier 웹앱을 각각의 빌드 방식으로 배포함



Front end 어플리케이션은 Image build 방식으로 배포합니다.



1. Admin Console의 `Deploy Image` 혹은 Project 를 선택하여 우측 상단의 `Add to Project`의 `Deploy Image`를 선택



2. `Image Name`에 `openshiftroadshow/parksmap-katacoda:1.0.0` 입력 후 찾기버튼 클릭



3. 이미지가 정상적으로 찾아지면 이미지 정보가 표기됨



   > ### openshiftroadshow/parksmap-katacoda:1.0.0 2 years ago, 194.3 MiB, 12 layers

   >

   > - Image Stream **parksmap-katacoda:1.0.0** will track this image.

   > - This image will be deployed in Deployment Config **parksmap-katacoda**.

   > - Port 8080/TCP will be load balanced by Service **parksmap-katacoda**. Other containers can access this service through the hostname **parksmap-katacoda**.



4. 이미지 정보를 확인 한 뒤 'Deploy' 버튼 클릭



5. 생성 확인. 동작하는 작업을 확인하기 위해서는 `Continue to the project overvie` 링크 클릭하여 확인 가능 



6. Project Overview화면에서 `parksmap-katacoda` 어플리케이션을 확장하여 `Create Route` 클릭



   - 기본 설정된 값으로 생성



Front end 어플리케이션만 동작하고 있을때는 지도상에 표기할 데이터를 Back end로 부터 받아오지 못하여 데이터 기반 정보는 표기되지 않습니다.



Back end 어플리케이션은 Git의 소스를 S2I(Source to Image)를 사용하여 배포합니다. 해당 데모에서는 Github에 있는 소스를 Openshift에서 제공하는 Python 빌더 이미지와 결합하여 배포합니다.



1. Admin Console의 `Search Catalog` 에서 `Python` 을 찾거나, Project를 선택하여 우측 상단의 `Add to Project`의 `Browse Catalog`를 선택하여 `Python` 검색

2. `Python`을 선택 후 `Next`

3. `Configuration`

   - `Version`은 `3.6`을 선택

   - Application Name: nationalparks-katacoda

   - Git repository: https://github.com/changdukkim/nationalparks-katacoda

   - `Create` 버튼 클릭

4. 생성 확인. 동작하는 작업을 확인하기 위해서는 `Continue to the project overvie` 링크 클릭하여 확인 가능



생성된 Route로 요청하여 Front end 만 실행되었을 때와 비교 하면 Back end에서 데이터를 받아와서 화면에 표기되는 것이 확인됩니다.



Github의 어플리케이션에 변화가 있는 경우 재배포되는 것을 확인 할 수 있습니다.







## 2. Local S2I build



환경



- JDK 8

- Maven

- Project 소스코드 참고 (https://drive.google.com/open?id=1aSey1qiCF2lfsgKYDjmB1L4cWMZZkE_4)

- 출처: https://learn.openshift.com/middleware/courses/middleware-spring-boot/spring-getting-started







1. Project folder로 이동

```shell

cd ~/demo

```

2. mvn build 수행



```shell

mvn spring-boot:run

```



2. web browser에서 localhost:8080 > 확인 



3. Openshift Login

Openshift console에서 사용자 이름 밑에 ```Copy Login Command``` 클릭

```shell

oc login https://master.na39.openshift.opentlc.com --token=znVtaSAW-fST4moAFMzxX2Y8QDQE8r-n8TcAORdi6Y0

```



4. New Project만들기



```shell

oc new-project redhatforumkorea --display-name="Demo - Spring Boot App"

```



5. H2 database를 PostgreSQL로 대체



```shell
oc new-app -e POSTGRESQL_USER=luke \
-e POSTGRESQL_PASSWORD=secret \
-e POSTGRESQL_DATABASE=my_data \
openshift/postgresql-92-centos7 \
--name=my-database
```



6. fabric plug-in으로 openshift에 deploy



```shell

mvn package fabric8:deploy -Popenshift -DskipTests

```



7. 상태 확인



```shell

oc rollout status dc/fruits

```



8. 코드 수정 (/demo/src/main/resources/import.sql)

```shell

insert into fruit (name) values ('Cherry');

insert into fruit (name) values ('Apple');

insert into fruit (name) values ('Banana');

# add new sql

insert into fruit (name) values ('Mango');

```

9. Re-deploy (자동 Rolling-Update)



```shell

mvn package fabric8:deploy -Popenshift -DskipTests

```









## 3. Binary Source Deployment on JBOSS EAP



Source Link : https://github.com/RHsyseng/MSA-EAP7-OSE
Blog Link : https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/build_and_deployment_of_java_applications_on_openshift_container_platform_3/index


Maven으로 빌드를 수행하여 war 파일을 생성한다.



`mvn install`



1. `mkdir C:\tmp\nocontent`

2. `oc new-app jboss-eap71-openshift:1.1~c:\\tmp\\nocontent --name=billing-service`

3. `oc start-build billing-service --from-file=billing.war`







어플리케이션에서 사용할 MySQL Pod을 생성한다.



```

oc new-app -e MYSQL_USER=product -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=product -e MYSQL_ROOT_PASSWORD=passwd mysql --name=product-db

```



```

oc new-app -e MYSQL_USER=sales -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=sales -e MYSQL_ROOT_PASSWORD=passwd mysql --name=sales-db

```







1. Product service

   ![](https://access.redhat.com/webassets/avalon/d/Reference_Architectures-2017-Build_and_Deployment_of_Java_Applications_on_OpenShift_Container_Platform_3-en-US/images/2e8dc106ba494f620c21ff2f7ed22efa/product_deploy.png)

2. `oc new-app jboss-eap71-openshift:1.1~c:\\tmp\\nocontent --name=product-service`

3. ``oc start-build product-service --from-dir=deploy`







1. Sales service

   ![](C:\Users\gyulee\Dropbox\Markup\images\sales_deploy.png)

2. `oc new-app jboss-eap71-openshift:1.1~c:\\tmp\\nocontent --name=sales-service`

3. ``oc start-build product-service --from-dir=deploy`







1. `oc new-app jboss-eap71-openshift:1.1~c:\\tmp\\nocontent --name=presentation`

2. `oc start-build presentation --from-file=ROOT.war`

3. `oc expose service presentation`



```shell

C:\workspace\MSA-EAP7-OSE-master\Billing\target>mkdir C:\tmp\nocontent

C:\workspace\MSA-EAP7-OSE-master\Billing\target>oc new-app jboss-eap71-openshift:1.1~c:\\tmp\\nocontent --name=billing-service

--> Found image c326e30 (8 months old) in image stream "openshift/jboss-eap71-openshift" under tag "1.1" for "jboss-eap71-openshift:1.1"



    JBoss EAP 7.1

    -------------

    Platform for building and running JavaEE applications on JBoss EAP 7.1



    Tags: builder, javaee, eap, eap7



    * A source build using binary input will be created

      * The resulting image will be pushed to image stream "billing-service:latest"

      * A binary build was created, use 'start-build --from-dir' to trigger a new build

    * This image will be deployed in deployment config "billing-service"

    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "billing-service"

      * Other containers can access this service through the hostname "billing-service"



--> Creating resources ...

    imagestream "billing-service" created

    buildconfig "billing-service" created

    deploymentconfig "billing-service" created

    service "billing-service" created

--> Success

    Build scheduled, use 'oc logs -f bc/billing-service' to track its progress.

    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:

     'oc expose svc/billing-service'

    Run 'oc status' to view your app.

    

C:\workspace\MSA-EAP7-OSE-master\Billing\target>oc start-build billing-service --from-file=billing.war

Uploading file "billing.war" as binary input for the build ...

build "billing-service-2" started

```







