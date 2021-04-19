# nexus

## docker run
```bash
docker run -d -p 8081:8081 --name nexus --restart always -v /data/nexus:/nexus-data sonatype/nexus3
```

## nexus3x版本上传第三方jar包
因为nexus3x版本没有2x版本中内置的3rd_part，所以不能在界面中上传jar包，必须使用maven的命令行。
-Dfile指定上传jar包  -Durl指定上传nexus私服地址 -DrepositoryId指定settings.xml中server段中相应id(登陆信息)

```bash
mvn deploy:deploy-file -DgroupId=com.fruitday -DartifactId=test01 -Dversion=0.0.1 -Dpackaging=jar -Dfile=test01.jar -Durl=http://10.28.50.6:8081/repository/3rd-party -DrepositoryId=3rd-party
```