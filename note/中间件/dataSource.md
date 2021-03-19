1. spring 使用datasource内容

   默认的datasource选择

   ```java
    { "com.zaxxer.hikari.HikariDataSource",
         "org.apache.tomcat.jdbc.pool.DataSource", "org.apache.commons.dbcp2.BasicDataSource" }
   ```

```java
@Bean
@ConfigurationProperties("foo.datasource")
public DataSourceProperties fooDataSourceProperties(){
    logger.info("foo dataSource properties{}");
    return new DataSourceProperties();
}

@Bean
public DataSource fooDataSource(){
    DataSourceProperties dataSourceProperties = fooDataSourceProperties();
    logger.info("foo dataSource {}",dataSourceProperties.getUrl());
    return dataSourceProperties.initializeDataSourceBuilder().build();
}
```