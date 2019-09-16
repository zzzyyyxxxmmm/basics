# mvn生命周期
1. Clean
2. Default: compile -> test-compile -> test -> package -> install 执行test会自动执行前面两个
3. site

正常来说就是```mvn clean install```就行了

# Scope 周期

| scope    | compile | test | package | run | example     |
|----------|---------|------|---------|-----|-------------|
| compile  | ×       | √    | √       | √   | spring-core |
| provided | √       | √    | ×       | ×   | servlet-api |
| test     | ×       | √    | ×       | ×   | junit       |
| runtime  | ×       | √    | √       | √   | JDBCDriver  |


## 运行spring boot
```mvn tomcat:run```
