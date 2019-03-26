# Mybatis处理流程

1. 解析Xml文件
2. 将Xml中内容保存到Configuration中
3. 将Configuration交给SqlSessionFactory管理
4. 由SqlSessionFactory创建SqlSession对象
5. 由SqlSession对象完成对数据库操作
6. SqlSession对象，通过Mapper的id与Xml中的配置进行关联
7. Executor : 一个在SqlSession内部使用接口负责对数据的具体操作
8. mapped statement(底层封装工具类) ：负责生成具体的SQL命令以及对查询结果 集的二次封装

9. ObjectFactory 将查询结果转换成实体， 创建实体对象