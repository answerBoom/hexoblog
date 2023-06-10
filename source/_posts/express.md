---
title: express
description: 快递项目笔记
mathjax: true
tags:
  - Java
categories:
  - Java
abbrlink: 2013454d
sticky: 2
swiper_index: 2
date: 2023-06-08
---
```java
package com.fzu.express;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.po.TableFill;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.ArrayList;
import java.util.List;

/**
 * @author
 * @since 2018/12/13
 */
public class CodeGenerator {

    public static void main(String[] args) {

        // 1、创建代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 2、全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir("D:\\java2\\express-back-end\\src\\main\\java");

        gc.setAuthor("liliye");
        gc.setOpen(false); //生成后是否打开资源管理器
        gc.setFileOverride(false); //重新生成时文件是否覆盖

        //UserServie
        gc.setServiceName("%sService");    //去掉Service接口的首字母I

        gc.setIdType(IdType.AUTO); //主键策略
        gc.setDateType(DateType.ONLY_DATE);//定义生成的实体类中日期类型
        gc.setSwagger2(false);//开启Swagger2模式
        mpg.setGlobalConfig(gc);

        // 3、数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://127.0.0.1:3306/express?allowPublicKeyRetrieval=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 4、包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("express"); //模块名
        //包
        pc.setParent("com.fzu");
        //包
        pc.setController("controller");
        pc.setEntity("entity");
        pc.setService("service");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 5、策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setInclude("sys_log");
        strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略
        strategy.setTablePrefix(pc.getModuleName() + "_"); //生成实体时去掉表前缀

        strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略
        strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作
        strategy.setLogicDeleteFieldName("deleted"); //配置逻辑删除字段的名称
//        strategy.setVersionFieldName("version");//配置乐观锁字段

        //配置创建时间 更新时间 字段
        List<TableFill> tableFillList = new ArrayList<>();
        tableFillList.add(new TableFill("create_time", FieldFill.INSERT));
        tableFillList.add(new TableFill("update_time", FieldFill.INSERT_UPDATE));
        strategy.setTableFillList(tableFillList);

        strategy.setRestControllerStyle(true); //restful api风格控制器
        strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符

        mpg.setStrategy(strategy);


        // 6、执行
        mpg.execute();
    }
}
~~~

~~~java
import com.alibaba.fastjson.JSON;
import com.fzu.express.annotation.SystemLog;
import com.fzu.express.entity.SysLog;
import com.fzu.express.entity.WebUser;
import com.fzu.express.service.SysLogService;
import com.fzu.express.utils.JwtUtil;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Enumeration;

/**
 * @description:
 * @create: 2022-11-05 17:14
 **/

@Component
@Aspect
@Slf4j
public class LogAspect {
    @Autowired
    private SysLogService sysLogService;
    @Pointcut("@annotation(com.fzu.express.annotation.SystemLog)")
    public void pt() {
     }

     @Around("pt()")
     public Object printLog(ProceedingJoinPoint joinPoint) throws Throwable {
         Object proceed;
         try {
             handlerBefore(joinPoint);
             proceed = joinPoint.proceed();
             handleAfter(proceed);
         } finally {
             // 结束后换行
             log.info("===========END============" + System.lineSeparator());
         }
         return proceed;
     }

    private void handleAfter(Object proceed) {
        // 打印出参
        log.info("Response       : {}",JSON.toJSONString(proceed));
    }

    private void handlerBefore(ProceedingJoinPoint joinPoint) {
        ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = servletRequestAttributes.getRequest();
        //获取被增强方法的注解对象
        SystemLog systemLog = getSystemLog(joinPoint);
        String token = request.getHeader("token");
        System.out.println(token);
        if (StringUtils.isNotBlank(token) && !"null".equals(token)) {
            WebUser user = JwtUtil.getUser(token);
            SysLog sysLog = new SysLog(null, user.getId(), user.getName() + "执行了" + systemLog.businessName() + "操作", null);
            sysLogService.save(sysLog);
        } else {
            SysLog sysLog = new SysLog(null, null, "执行了" + systemLog.businessName() + "操作", null);
            sysLogService.save(sysLog);
        }
        log.info("=======Start=======");
        // 打印请求 URL
        log.info("URL             : {}",request.getRequestURI());
        HttpServletRequest httpServletRequest = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        // 打印描述信息
        log.info("BusinessName   : {}",systemLog.businessName());
        // 打印 Http method
        log.info("HTTP Method    : {}",request.getMethod());
        // 打印调用 controller 的全路径以及执行方法
        log.info("Class Method   : {}.{}",joinPoint.getSignature().getDeclaringTypeName(),((MethodSignature) joinPoint.getSignature()).getName());
        // 打印请求的 IP
        log.info("IP             : {}",request.getRemoteHost());
        // 打印请求入参
        log.info("Request Args   : {}", JSON.toJSONString(joinPoint.getArgs()));
    }

    private SystemLog getSystemLog(ProceedingJoinPoint joinPoint) {
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        SystemLog systemLog = methodSignature.getMethod().getAnnotation(SystemLog.class);
        return systemLog;
    }
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface SystemLog {
    String businessName();
}
```

