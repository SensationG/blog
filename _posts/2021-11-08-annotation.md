---
layout: post
title: 自定义注解使用
date: 2021-11-08
Author: hhw
comments: true
toc: true
tags:  [java]

---

## 1、自定义表单操作日志注解

### 1.定义注解

```java
/**
 * @Author: huanghwh
 * @Date: 2021/08/20 19:49
 * @Description: 表单操作日志记录
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BusinessFormOperateLog {

    /**
     * 实体类
     */
    Class entityClass();

    /**
     * 操作类型 (新增；修改；删除) 见 Globals
     */
    String operateType();

}
```

### 2.定义切面

定义切面，拦截所有使用了该注解的方法

```java
import com.alibaba.fastjson.JSONObject;
import com.ursful.framework.orm.IBaseService;
import com.ursful.framework.orm.IMultiQuery;
import com.ursful.framework.orm.query.MultiQueryImpl;
import com.ursful.framework.orm.support.AliasTable;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Objects;

/**
 * @Author: huanghwh
 * @Date: 2021/08 20/15:25
 * @Description: 表单操作日志记录切面
 */
@Slf4j
@Component
@Aspect
public class FormOperateAspect {

    private static final String EXCLUDE_COLUMN = "id,projectId,templateId,parentId,showLog,sno,level,sort";

    @Autowired
    IFormOperateLogService formOperateLogService;

    // 拦截了使用该注解的方法
    @Around("execution(* *.*(..)) && @annotation(com.hymake.jupcc.common.annotation.BusinessFormOperateLog)")
    public Object validateCustomPoint(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("@FormOperateLog开始记录表单操作日志");
      
        // 通过切面获得注解上的值、方法名称
        Object proceed = null;
        IBaseService baseService = (IBaseService) joinPoint.getTarget();
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = null;
        method = methodSignature.getMethod();
        BusinessFormOperateLog annotation = method.getAnnotation(BusinessFormOperateLog.class);

        // 获得注解上的值
        String operateType = annotation.operateType();

        if (StringUtils.equals(operateType, Globals.MODIFY)) {
            // 获取参数
            JSONObject newJsonValue = new JSONObject();
            Object[] args = joinPoint.getArgs();
            for (Object arg : args) {
                if (arg instanceof JSONObject) {
                    newJsonValue = (JSONObject) arg;
                }
            }

            // 从数据库查询旧参数
            When.emptyReturnErrorCode(newJsonValue.get("id"), SystemErrorCode.DO_ERROR_CUSTOM, "记录日志失败：表单id不能为空");
            Long id = Long.valueOf(String.valueOf(newJsonValue.get("id")));
            IMultiQuery query = new MultiQueryImpl();
            AliasTable table = query.table(annotation.entityClass());
            query.whereEqual(table.c("ID"), id);
            LongIdEntity entity = (LongIdEntity) baseService.get(query);
            JSONObject oldJsonObject = BeanUtils.entity2Json(entity);

            // 新旧比较 使用keySet将两个实体进行逐字段比较
            List<FormOperateLog> insertList = new ArrayList<>();
            for (Map.Entry entry : newJsonValue.entrySet()) {
                String key = String.valueOf(entry.getKey());
                Object newValue = entry.getValue();
                Object oldValue = oldJsonObject.get(entry.getKey());
                // 排除掉字段中文名的变更
                if (StringUtils.isNotEmpty(key) && key.length() >= 5){
                    String columnName = key.substring(key.length() - 5);
                    if (StringUtils.equals(columnName, "Title")) {
                        continue;
                    }
                }
                if (EXCLUDE_COLUMN.contains(key)) {
                    continue;
                }
                // 前置处理，防止类型不同比较失败
                if (oldValue instanceof BigDecimal) {
                    oldValue = String.valueOf(oldValue);
                }
                if (Objects.nonNull(newValue) && (Boolean.FALSE.equals(Objects.equals(oldValue, newValue)) || Objects.isNull(oldValue))) {
                    log.info("++++=================变动项--> key：" + key + "--> oldValue：" + oldValue + "---> newValue：" + newValue);
                    FormOperateLog formOperateLog = new FormOperateLog();
                    formOperateLog.setFormId(id);
                    formOperateLog.setName(newJsonValue.getString(entry.getKey() + "Title"));
                    if (Objects.nonNull(oldValue)) {
                        formOperateLog.setOldValue(String.valueOf(oldValue));
                    }
                    formOperateLog.setNewValue(String.valueOf(newValue));
                    formOperateLog.setOperateType(annotation.operateType());
                    formOperateLog.setType(annotation.entityClass().getSimpleName());
                    formOperateLog.setOperateInfo();
                    insertList.add(formOperateLog);
                }
            }
            // 执行方法
            proceed = joinPoint.proceed();
            formOperateLogService.batchSaves(insertList, true);
        } else if (StringUtils.equals(operateType, Globals.ADD)) {
            // 新增
            JSONObject jsonObject = new JSONObject();
            Object[] args = joinPoint.getArgs();
            for (Object arg : args) {
                if (arg.getClass().equals(annotation.entityClass())) {
                    jsonObject = BeanUtils.entity2Json(arg);
                }
            }
            Long id = Long.valueOf(String.valueOf(jsonObject.get("id")));
            When.nullReturnErrorCode(id, SystemErrorCode.DO_ERROR_CUSTOM, "记录日志失败：新增时请提前生成主键");
            FormOperateLog formOperateLog = new FormOperateLog();
            formOperateLog.setFormId(id);
            formOperateLog.setOperateType(annotation.operateType());
            formOperateLog.setType(annotation.entityClass().getSimpleName());
            formOperateLog.setOperateInfo();
            proceed = joinPoint.proceed();
            formOperateLogService.save(formOperateLog);
        } else if (StringUtils.equals(operateType, Globals.DELETE)) {
            // 删除
            List<JSONObject> deleteJsonList = new ArrayList<>();
            Object[] args = joinPoint.getArgs();
            for (Object arg : args) {
                if (arg instanceof List) {
                    deleteJsonList = (List<JSONObject>) arg;
                }
            }
            List<FormOperateLog> insertList = new ArrayList<>();
            for (JSONObject jsonObject : deleteJsonList) {
                FormOperateLog formOperateLog = new FormOperateLog();
                formOperateLog.setFormId(Long.valueOf(String.valueOf(jsonObject.get("id"))));
                formOperateLog.setOperateType(annotation.operateType());
                formOperateLog.setType(annotation.entityClass().getSimpleName());
                formOperateLog.setOperateInfo();
                insertList.add(formOperateLog);
            }
            proceed = joinPoint.proceed();
            formOperateLogService.batchSaves(insertList, true);
        } else {
            When.returnErrorCode(SystemErrorCode.DO_ERROR_CUSTOM, "记录日志切面发送异常：操作类型不存在");
        }
        log.info("@FormOperateLog结束记录");
        return proceed;
    }

}
```

### 3.使用

```java
@BusinessFormOperateLog(entityClass = ProjectTargetAnalysis.class, operateType = Globals.ADD)
@Transactional(rollbackFor = Exception.class)
@Override
public boolean add(ProjectTargetAnalysis projectTargetAnalysis) { ... }
```

## 2、自定义权限切面注解

### 1.定义注解

```java
package com.hymake.bugattizz.common.annotation;

import java.lang.annotation.*;

/**
 * @Author: huanghwh
 * @Date: 2021/08/20 19:49
 * @Description: 表单操作日志记录
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Permit {

    /**
     * 允许访问的角色名称，多个逗号隔开    todo: 这边用数组更为合适
     */
    String allowRoleName() default "";

    /**
     * 不允许访问的角色名称，多个逗号隔开
     */
    String forbiddenRoleName() default "";

    /**
     * 允许访问的机构类型，多个逗号隔开
     */
    String allowOrganizationType() default "";

    /**
     * 不允许访问的机构类型，多个逗号隔开
     */
    String forbiddenOrganizationType() default "";
}
```

### 2.定义切面

```java

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;


/**
 * @Author: huanghwh
 * @Date: 2021/10/25 14:22
 * @Description: 接口角色控制切面
 */
@Slf4j
@Component
@Aspect
public class PermitAspect {


    @Before("execution(* *.*(..)) && @annotation(com.hymake.bugattizz.common.annotation.Permit)")
    public void permitPoint(JoinPoint joinPoint) {
        log.info("@Permit判断接口权限");
      
        // 获取注解上传入的参数
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = methodSignature.getMethod();
        Permit permit = method.getAnnotation(com.hymake.bugattizz.common.annotation.Permit.class);

        String allowRoleName = permit.allowRoleName();
        String forbiddenRoleName = permit.forbiddenRoleName();
        String allowOrganizationType = permit.allowOrganizationType();
        String forbiddenOrganizationType = permit.forbiddenOrganizationType();
        String sysRoleName = BugattiZz.getCurrentUser().getSysRoleName();
        String organizationType = BugattiZz.getCurrentUser().getOrganizationType();
        if (StringUtils.isBlank(sysRoleName)) {
            throw new CommonException("无权限：未查询到当前用户角色信息");
        }
        // 校验允许的角色范围
        if (StringUtils.isNotBlank(allowRoleName) && Boolean.FALSE.equals(allowRoleName.contains(sysRoleName))) {
            throw new CommonException("无访问权限");
        }
        // 校验不允许的角色范围
        if (StringUtils.isNotBlank(forbiddenRoleName) && Boolean.TRUE.equals(forbiddenRoleName.contains(sysRoleName))) {
            throw new CommonException("无访问权限");
        }
        // 校验允许的机构类型
        if (StringUtils.isNotBlank(allowOrganizationType) && Boolean.FALSE.equals(allowOrganizationType.contains(organizationType))) {
            throw new CommonException("无访问权限");
        }
        // 校验不允许的机构类型
        if (StringUtils.isNotBlank(forbiddenOrganizationType) && Boolean.TRUE.equals(forbiddenOrganizationType.contains(organizationType))) {
            throw new CommonException("无访问权限");
        }
    }

}
```

### 3.使用

```java
@Permit(allowRoleName = RoleNameConstant.CITY_ROLE) // 使用
@RsMethod(name = "删除", url = "/delete", security = SecurityType.LOGINED)
@RsDocument(errors = {NoticeErrorCode.class})
public void delete(@RsInput(description = "主键") String id){
    When.emptyReturnErrorCode(id, NoticeErrorCode.NOTICE_ID_IS_EMPTY);
    Express express = new Express(Notice.T_ID, id, ExpressionType.CDT_EQUAL);
    noticeService.deletes(express);
}
```