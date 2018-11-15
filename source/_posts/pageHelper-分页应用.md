---
title: pageHelper 分页应用
date: 2018-09-14 14:08:26
tags:
    - pageHelper 分页应用
---
![猫咪老师](/images/页面图片/6.jpeg)
<!--more-->
## 1.pagehelper 分页原理
pageHelper 是做什么的呢？他是封装了分页的后台部分，说的简单一点，就是你不需要每个POJO类的增删改查里都包括那两个方法了，他帮你做了。你只需要有一个selectAll的方法，他会根据你使用的数据库将你selectAll的sql改装成一个分页查询的SQL，并顺带生成一个查询总数的SQL。
### 1.1 如何进行拦截的呢？
通常我们使用这个插件的，配置一个拦截器“com.github.pagehelper.PageInterceptor”，让我们看看这个拦截器里面做了什么操作？
```
/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2017 abel533@gmail.com
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */

package com.github.pagehelper;

import com.github.pagehelper.cache.Cache;
import com.github.pagehelper.cache.CacheFactory;
import com.github.pagehelper.util.MSUtils;
import com.github.pagehelper.util.StringUtil;
import org.apache.ibatis.cache.CacheKey;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Properties;

/**
 * Mybatis - 通用分页拦截器<br/>
 * 项目地址 : http://git.oschina.net/free/Mybatis_PageHelper
 *
 * @author liuzh/abel533/isea533
 * @version 5.0.0
 */
@SuppressWarnings({"rawtypes", "unchecked"})
@Intercepts(
    {
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
    }
)
public class PageInterceptor implements Interceptor {
    //缓存count查询的ms
    protected Cache<CacheKey, MappedStatement> msCountMap = null;
    private Dialect dialect;
    private String default_dialect_class = "com.github.pagehelper.PageHelper";
    private Field additionalParametersField;

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        try {
            Object[] args = invocation.getArgs();
            MappedStatement ms = (MappedStatement) args[0];
            Object parameter = args[1];
            RowBounds rowBounds = (RowBounds) args[2];
            ResultHandler resultHandler = (ResultHandler) args[3];
            Executor executor = (Executor) invocation.getTarget();
            CacheKey cacheKey;
            BoundSql boundSql;
            //由于逻辑关系，只会进入一次
            if(args.length == 4){
                //4 个参数时
                boundSql = ms.getBoundSql(parameter);
                cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
            } else {
                //6 个参数时
                cacheKey = (CacheKey) args[4];
                boundSql = (BoundSql) args[5];
            }
            List resultList;
            //调用方法判断是否需要进行分页，如果不需要，直接返回结果
            if (!dialect.skip(ms, parameter, rowBounds)) {
                //反射获取动态参数
                Map<String, Object> additionalParameters = (Map<String, Object>) additionalParametersField.get(boundSql);
                //判断是否需要进行 count 查询
                if (dialect.beforeCount(ms, parameter, rowBounds)) {
                    //创建 count 查询的缓存 key
                    CacheKey countKey = executor.createCacheKey(ms, parameter, RowBounds.DEFAULT, boundSql);
                    countKey.update("_Count");
                    MappedStatement countMs = msCountMap.get(countKey);
                    if (countMs == null) {
                        //根据当前的 ms 创建一个返回值为 Long 类型的 ms
                        countMs = MSUtils.newCountMappedStatement(ms);
                        msCountMap.put(countKey, countMs);
                    }
                    //调用方言获取 count sql
                    String countSql = dialect.getCountSql(ms, boundSql, parameter, rowBounds, countKey);
                    BoundSql countBoundSql = new BoundSql(ms.getConfiguration(), countSql, boundSql.getParameterMappings(), parameter);
                    //当使用动态 SQL 时，可能会产生临时的参数，这些参数需要手动设置到新的 BoundSql 中
                    for (String key : additionalParameters.keySet()) {
                        countBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
                    }
                    //执行 count 查询
                    Object countResultList = executor.query(countMs, parameter, RowBounds.DEFAULT, resultHandler, countKey, countBoundSql);
                    Long count = (Long) ((List) countResultList).get(0);
                    //处理查询总数
                    //返回 true 时继续分页查询，false 时直接返回
                    if (!dialect.afterCount(count, parameter, rowBounds)) {
                        //当查询总数为 0 时，直接返回空的结果
                        return dialect.afterPage(new ArrayList(), parameter, rowBounds);
                    }
                }
                //判断是否需要进行分页查询
                if (dialect.beforePage(ms, parameter, rowBounds)) {
                    //生成分页的缓存 key
                    CacheKey pageKey = cacheKey;
                    //处理参数对象
                    parameter = dialect.processParameterObject(ms, parameter, boundSql, pageKey);
                    //调用方言获取分页 sql
                    String pageSql = dialect.getPageSql(ms, boundSql, parameter, rowBounds, pageKey);
                    BoundSql pageBoundSql = new BoundSql(ms.getConfiguration(), pageSql, boundSql.getParameterMappings(), parameter);
                    //设置动态参数
                    for (String key : additionalParameters.keySet()) {
                        pageBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
                    }
                    //执行分页查询
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, pageKey, pageBoundSql);
                } else {
                    //不执行分页的情况下，也不执行内存分页
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, cacheKey, boundSql);
                }
            } else {
                //rowBounds用参数值，不使用分页插件处理时，仍然支持默认的内存分页
                resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
            }
            return dialect.afterPage(resultList, parameter, rowBounds);
        } finally {
            dialect.afterAll();
        }
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        //缓存 count ms
        msCountMap = CacheFactory.createCache(properties.getProperty("msCountCache"), "ms", properties);
        String dialectClass = properties.getProperty("dialect");
        if (StringUtil.isEmpty(dialectClass)) {
            dialectClass = default_dialect_class;
        }
        try {
            Class<?> aClass = Class.forName(dialectClass);
            dialect = (Dialect) aClass.newInstance();
        } catch (Exception e) {
            throw new PageException(e);
        }
        dialect.setProperties(properties);
        try {
            //反射获取 BoundSql 中的 additionalParameters 属性
            additionalParametersField = BoundSql.class.getDeclaredField("additionalParameters");
            additionalParametersField.setAccessible(true);
        } catch (NoSuchFieldException e) {
            throw new PageException(e);
        }
    }

}

```
通过源代码发现：拦截器会根据参数来判断使用需要进行的物理分页，不使用分页插件处理时，仍支持默认的内存分页。需要分页时候，根据条件判断是否需要count和limit处理。

### 1.2 如何处理分页的逻辑
对应不同的数据库有不同的方言，如常见的Mysql，分页关键字：limit，它是如此组装你的SQL：
```
/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2017 abel533@gmail.com
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */

package com.github.pagehelper.dialect.rowbounds;

import com.github.pagehelper.dialect.AbstractRowBoundsDialect;
import org.apache.ibatis.cache.CacheKey;
import org.apache.ibatis.session.RowBounds;

/**
 * mysql 基于 RowBounds 的分页
 *
 * @author liuzh
 */
public class MySqlRowBoundsDialect extends AbstractRowBoundsDialect {

    @Override
    public String getPageSql(String sql, RowBounds rowBounds, CacheKey pageKey) {
        StringBuilder sqlBuilder = new StringBuilder(sql.length() + 14);
        sqlBuilder.append(sql);
        if (rowBounds.getOffset() == 0) {
            sqlBuilder.append(" LIMIT ");
            sqlBuilder.append(rowBounds.getLimit());
        } else {
            sqlBuilder.append(" LIMIT ");
            sqlBuilder.append(rowBounds.getOffset());
            sqlBuilder.append(",");
            sqlBuilder.append(rowBounds.getLimit());
            pageKey.update(rowBounds.getOffset());
        }
        pageKey.update(rowBounds.getLimit());
        return sqlBuilder.toString();
    }

}
```
封装分页SQL，使我们不需要每个地方都去写分页的查询信息，同时，使我们select的SQL语言向下兼容，换了数据库也不需要更改SQL代码。
## 2. pageHelper 应用举例
```
参数：
package com.diditech.iov.inventory.domain;

import com.alibaba.fastjson.annotation.JSONField;
import com.diditech.iov.core.utils.QueryParam;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * 4S店库存车辆管理请求参数
 * @author wangnn
 * @date 2018/9/4 15:10
 */
public class VehicleManagementQueryParam extends QueryParam{

    //排序
    @JSONField(serialize = false)
    private final static Map<String,String[]> sortMap = new HashMap();

    static {
        sortMap.put("insatlltime",new String[]{"cvi.INSTALL_TIME"});
    }

    @Override
    public String getSort_desc() {
        return parseDESCSortParamMulti(sort_desc, sortMap);
    }

    @Override
    public String getSort_asc() {
        return parseASCSortParamMulti(sort_asc, sortMap);
    }
    //----------------------请求参数--------------------------
    /**
     * 仓库ID
     */
        private Integer wareHouseId;
    /**
     * 入库开始时间 年-月-日
     */
    private Date startTime;
    /**
     * 入库结束时间
     */
    private Date endTime;
    /**
     * 设备编号  返回一样
     */
    private String deviceNum;
    //-----------------------返回参数------------------------------
    /**
     * 车型
     */
    private String vehicleType;
    /**
     * 车系
     */
    private String vehicleSerie;
    /**
     * 车架号
     */
    private String frameNo;
    /**
     * 所属仓库名称
     */
    private String wareHoseName;
    /**
     * 安装时间
     */
    private Date installDate;
//    /**
//     * 安装时间
//     */
//    private String installDateStr;
    /**
     *  入库时间
     */
    private Date wareHousingTime;
//    /**
//     * 返回前端 入库时间
//     */
//    private String wareHousingTimeStr;
    /**
     * sim卡号
     */
    private String simNum;
    /**
     * 车辆管理状态 -- 解押
     */
    private Byte detentionState;
//    /**
//     * 车辆管理状态 -- 解押
//     */
//    private String detentionStateStr;
    /**
     * 解押时间
     */
    private Date  detentionTime;
//    /**
//     * 解押时间
//     */
//    private String detentionTimeStr;
    /**
     * 车辆管理状态 -- 转售
     */
    private Byte resaleState;
//    /**
//     * 车辆管理状态 -- 转售
//     */
//    private String resaleStateStr;
    /**
     * 转售时间
     */
    private Date  resaleTime;
//    /**
//     * 转售时间
//     */
//    private String resaleTimeStr;
    /**
     * 车辆ID
     */
    private Integer vehicleId;

    public static Map<String, String[]> getSortMap() {
        return sortMap;
    }

    public Integer getWareHouseId() {
        return wareHouseId;
    }

    public void setWareHouseId(Integer wareHouseId) {
        this.wareHouseId = wareHouseId;
    }

    public Date getStartTime() {
        return startTime;
    }

    public void setStartTime(Date startTime) {
        this.startTime = startTime;
    }

    public Date getEndTime() {
        return endTime;
    }

    public void setEndTime(Date endTime) {
        this.endTime = endTime;
    }

    public String getDeviceNum() {
        return deviceNum;
    }

    public void setDeviceNum(String deviceNum) {
        this.deviceNum = deviceNum;
    }

    public String getVehicleType() {
        return vehicleType;
    }

    public void setVehicleType(String vehicleType) {
        this.vehicleType = vehicleType;
    }

    public String getVehicleSerie() {
        return vehicleSerie;
    }

    public void setVehicleSerie(String vehicleSerie) {
        this.vehicleSerie = vehicleSerie;
    }

    public String getFrameNo() {
        return frameNo;
    }

    public void setFrameNo(String frameNo) {
        this.frameNo = frameNo;
    }

    public String getWareHoseName() {
        return wareHoseName;
    }

    public void setWareHoseName(String wareHoseName) {
        this.wareHoseName = wareHoseName;
    }

    public Date getInstallDate() {
        return installDate;
    }

    public void setInstallDate(Date installDate) {
        this.installDate = installDate;
    }

    public Date getWareHousingTime() {
        return wareHousingTime;
    }

    public void setWareHousingTime(Date wareHousingTime) {
        this.wareHousingTime = wareHousingTime;
    }

    public String getSimNum() {
        return simNum;
    }

    public void setSimNum(String simNum) {
        this.simNum = simNum;
    }

    public Byte getDetentionState() {
        return detentionState;
    }

    public void setDetentionState(Byte detentionState) {
        this.detentionState = detentionState;
    }

    public Date getDetentionTime() {
        return detentionTime;
    }

    public void setDetentionTime(Date detentionTime) {
        this.detentionTime = detentionTime;
    }

    public Byte getResaleState() {
        return resaleState;
    }

    public void setResaleState(Byte resaleState) {
        this.resaleState = resaleState;
    }

    public Date getResaleTime() {
        return resaleTime;
    }

    public void setResaleTime(Date resaleTime) {
        this.resaleTime = resaleTime;
    }

    public Integer getVehicleId() {
        return vehicleId;
    }

    public void setVehicleId(Integer vehicleId) {
        this.vehicleId = vehicleId;
    }
}

```

```
QueryParam：
package com.diditech.iov.core.utils;

import java.util.Map;

import javax.validation.constraints.Min;

import org.apache.commons.lang3.StringUtils;
import org.hibernate.validator.constraints.Length;
import org.springframework.core.env.Environment;

import com.diditech.iov.core.exception.BusinessException;
/**
 * 分页排序 共通过滤 参数
 * @author Lionel
 *
 */
public class QueryParam {
	//final static String regEX = "[`~!@#$%^&*()+=|{}':;',\\[\\].<>/?~！@#￥%……&*（）——+|{}【】‘；：”“’。，、？]";
	//若支持前天传多列排序 则使用regEX2
	//final static String regEX2 = "[`~!@#$%^&*()+=|{}':;'\\[\\].<>/?~！@#￥%……&*（）——+|{}【】‘；：”“’。，、？]";
	final static String ASC = " ASC ";
	final static String DESC = " DESC ";
	
	private static Environment env;
	{
		env = (Environment)ApplicationContextUtils.getBean("environment");
	}
	
	public QueryParam(){
		page_size = env.getProperty("custome.page.size");
		page_index = "1";//默认第一页
	}
	
	/**
	 * 记录条数 page_size
	 */
	@Length(min=0, max=10)
	@Min(0)
	protected String page_size;

	/**
	 * 记录页数 page_index
	 */
	@Length(min=0, max=10)
	protected String page_index;
	
	/**
	 * 忽略条数 offset
	 */
	@Length(min=0, max=10)
	protected String offset;
	
	/**
	 * 最大条数 limit
	 */
	@Length(min=0, max=10)
	protected String limit;
	
	/**
	 * 降序排序，多个用逗号分隔 sort_desc
	 */
	@Length(min=0, max=30)
	protected String sort_desc;
	
	/**
	 * 降序排序，多个用逗号分隔 sort_asc
	 */
	@Length(min=0, max=30)
	protected String sort_asc;


	public String getPage_size() {
		return page_size;
	}

//	暂时后台统一配置
//	public void setPage_size(String page_size) {
//		this.page_size = page_size;
//	}


	public String getPage_index() {
		return page_index;
	}


	public void setPage_index(String page_index) {
		this.page_index = page_index;
	}


	public String getOffset() {
		try {
			if(StringUtils.isBlank(offset)&&StringUtils.isNotBlank(this.page_index)&&StringUtils.isNotBlank(this.page_size)){
				int pageIndex = Integer.valueOf(page_index);
				int pageSize = Integer.valueOf(page_size);
				int offSetTemp = (pageIndex-1)*pageSize;
				return String.valueOf(offSetTemp);
			}
		} catch (Exception e) {
			throw new BusinessException("查询参数异常");
		}
		return offset;
	}


	public void setOffset(String offset) {
		this.offset = offset;
	}


	public String getLimit() {
		if(StringUtils.isBlank(limit)&&StringUtils.isNotBlank(this.page_size)){
			return page_size;
		}
		return limit;
	}


	public void setLimit(String limit) {
		this.limit = limit;
	}


	//public abstract String getSort_desc();
	public String getSort_desc() {
		return sort_desc;
	}


	public void setSort_desc(String sort_desc) {
		this.sort_desc = sort_desc;
	}


	//public abstract String getSort_asc();
	public String getSort_asc() {
		return sort_asc;
	}


	public void setSort_asc(String sort_asc) {
		this.sort_asc = sort_asc;
	}

	/**
	 * 判断分页排序参数是否为空
	 * @return
	 */
	public boolean isEmply(){
		if(StringUtils.isNotBlank(getPage_index())||StringUtils.isNotBlank(getOffset())||
				StringUtils.isNotBlank(sort_asc)||StringUtils.isNotBlank(sort_desc)){
			return false;
		}
		return true;
	}
	
	/**
	 * 判断排序参数是否为空
	 * @return
	 */
	public boolean sortParamIsEmply(){
		if(StringUtils.isNotBlank(sort_asc)||StringUtils.isNotBlank(sort_desc)){
			return false;
		}
		return true;
	}
	
	/**
	 * 根据排序map创建
	 * @param shortString
	 * @param sortMap
	 * @return
	 * @deprecated 废弃
	 */
	public String parseSortParam(String sortString, Map<String, String> sortMap){
		if(StringUtils.isBlank(sortString)){
			return null;
		}
		String temp = sortString.replaceAll(StringTools.regEX_PUNCTUATION, "");
		String result = "";
		temp = temp.toLowerCase();
		if(StringUtils.isNotBlank(temp)){
			String[] tempArr = temp.split(",");
			for(String str: tempArr){
				String temp2 = sortMap.get(str);
				if(StringUtils.isNoneBlank(temp2)){
					result += temp2;
				}
			}
		}
		if(StringUtils.isEmpty(result)){
			throw new BusinessException("排序参数异常！");
		}
		return result;
	}
	/**
	 * 根据排序map创建正序字符串
	 * @param shortString
	 * @param sortMap
	 * @return
	 */
	public String parseASCSortParamMulti(String sortString, Map<String, String[]> sortMap){
		return parseSortParamMulti(ASC, sortString, sortMap);
	}
	/**
	 * 根据排序map创建倒序字符串
	 * @param shortString
	 * @param sortMap
	 * @return
	 */
	public String parseDESCSortParamMulti(String sortString, Map<String, String[]> sortMap){
		return parseSortParamMulti(DESC, sortString, sortMap);
	}
	private String parseSortParamMulti(String type, String sortString, Map<String, String[]> sortMap){
		if(StringUtils.isBlank(sortString)){
			return null;
		}
		String temp = sortString.replaceAll(StringTools.regEX_PUNCTUATION, "");
		StringBuffer result = new StringBuffer("");
		temp = temp.toLowerCase();
		if(StringUtils.isNotBlank(temp)){
			String[] tempArr = temp.split(",");
			for(String str: tempArr){
				String[] temp2 = sortMap.get(str);
				if(temp2==null){
					throw new BusinessException("排序参数不合法！");
				}
				for(int ii=0;ii<temp2.length;ii++){
					if(StringUtils.isNoneBlank(temp2[ii])){
						result.append(temp2[ii]).append(type).append(",");
					}
				}
			}
		}
		if(StringUtils.isEmpty(result)){
			throw new BusinessException("排序参数异常！");
		}
		result.deleteCharAt(result.length()-1);
		return result.toString();
	}
	
}

```

```
controller:
@AccessLogger(value = "库存车辆管理查询")
    @RequestMapping(value = "/info", method = RequestMethod.GET)
    @ApiOperation(value = "库存车辆管理查询", httpMethod = "GET",
            response = ResponseMessage.class, notes = "库存车辆管理查询")
    public ResponseMessage getVehicleManagementInfo(VehicleManagementQueryParam vehicleManagementQueryParam) {
        //前端 拼接时间
        Page<VehicleManagementQueryParam> vehicleManagementList =
                vehicleManagementService.getVehicleManagementInfo(vehicleManagementQueryParam);
        return ResponseMessage.ok(new PageInfo<VehicleManagementQueryParam>(vehicleManagementList));
    }
```
```
service:
@Override
    public Page<VehicleManagementQueryParam> getVehicleManagementInfo(VehicleManagementQueryParam vehicleManagementQueryParam) {
        AccUser loginUser = ContextUtils.getLoginUser();
        Integer shopId = loginUser.getCustomerId();
        //安装时间倒序
        if (vehicleManagementQueryParam.sortParamIsEmply()) {
            vehicleManagementQueryParam.setSort_desc("insatlltime");
        }
        PageHelper.startPage(Integer.valueOf(vehicleManagementQueryParam.getPage_index()),
                Integer.valueOf(vehicleManagementQueryParam.getPage_size()));
        List<VehicleManagementQueryParam> vehicleManagementList =
                vehicleManagementMapper.getVehicleManagementInfo(vehicleManagementQueryParam, shopId);
        return (Page<VehicleManagementQueryParam>) vehicleManagementList;
    }
```

```
mapper:
<select id="getVehicleManagementInfo"
            resultType="com.diditech.iov.inventory.domain.VehicleManagementQueryParam">
        SELECT
        cvi.VEHICLE_ID vehicleId,
        cvi.VEHICLE_BRAND_NAME vehicleType,
        cvi.VEHICLE_SERIES vehicleSerie,
        cvi.PLATE_NUM frameNo,
        cvi.WAREHOUSE_NAME wareHoseName,
        cvi.INSTALL_TIME installDate,
        cvi.INVENTORY_TIME wareHousingTime,
        cvi.DEVICE_NUM deviceNum,
        cvi.SIM_NUM simNum,
        cvi.ISRELEASED detentionState,
        cvi.RELEASE_TIME detentionTime,
        cvi.ISFORRESALE resaleState,
        cvi.RESALE_TIME resaleTime
        FROM
        dd_new.inventory_rpt_current cvi
        INNER JOIN dd_new.inventory_vehicle_ref vr
        ON vr.VEHICLE_ID = cvi.VEHICLE_ID
        WHERE
        vr.customer_4s_id = #{shopId}
        <if test="vehicleManagementQueryParam.wareHouseId != null" >
            AND vr.WAREHOUSE_ID = #{vehicleManagementQueryParam.wareHouseId}
        </if>
        <if test="vehicleManagementQueryParam.startTime != null" >
            AND cvi.INVENTORY_TIME &gt;= #{vehicleManagementQueryParam.startTime}
        </if>
        <if test="vehicleManagementQueryParam.endTime != null" >
            AND cvi.INVENTORY_TIME &lt;= #{vehicleManagementQueryParam.endTime}
        </if>
        <if test="vehicleManagementQueryParam.deviceNum != null and vehicleManagementQueryParam.deviceNum != ''">
            AND cvi.DEVICE_NUM LIKE CONCAT ('%',#{vehicleManagementQueryParam.deviceNum},'%')
        </if>
        <if test="vehicleManagementQueryParam.sort_desc != null and vehicleManagementQueryParam.sort_desc != ''" >
            ORDER BY ${vehicleManagementQueryParam.sort_desc}
        </if>
    </select>
```






