---
title: 深入 UmiJS：构建业务权限控制
date: 2025-05-26 09:58:12
tags:
  - React
  - UmiJS
categories:
  - React
---

# 背景

> 项目环境： React + UmiJS

常见的权限控制方式有如下：

- 接口权限控制
- 页面权限控制
- 组件权限控制

当前业务中暂未实现页面和组件的权限控制，本次基于 umijs 提供的 access 来帮助实现权限控制

# 调研

**权限数据来源**：由接口提供页面权限和功能权限

**现成可选的权限控制方案：**

1. [UmiJS 中 Access 的使用](https://umijs.org/docs/max/access)
2. [在 Route 层做权限拦截](https://umijs.org/docs/guides/routes#wrappers)

# 设计与实现

## access 设计

**思路**

1. 在页面初始化阶段将权限数据存放在 initialState 中
2. 对页面权限和组件权限分别做判断

**实现代码**
`src/access.ts`

```tsx
/**
 * @see https://umijs.org/zh-CN/plugins/plugin-access
 * */
export default function access(initialState: Record<string, any>) {
  const { accessInfo } = initialState;

  /**
   * 检查当前用户是否拥有权限： 包含了操作权限和页面权限
   */
  const checkAccess = (access: string | number | (string | number)[]) => {
    const { list: accessList = [], isGlobalRight } = accessInfo ?? {};

    // 有全部的权限
    if (isGlobalRight) return true;

    if (!access) return false;

    const accessArray = Array.isArray(access) ? access : [access];
    // (包含角色管理-权限列表)
    return accessArray.some((item) => accessList.includes(item));
  };

  /**
   * 检查当前用户是否拥有页面权限
   */
  const checkPageAccess = (access: string | number | (string | number)[]) => {
    const { pageAccessKeys = [] } = accessInfo ?? {};

    if (!access) return true;
    const accessArray = Array.isArray(access) ? access : [access];
    // (包含首页、角色管理-权限列表)
    return accessArray.some((item) => pageAccessKeys.includes(item));
  };

  return {
    /**
     * 检查当前用户是否拥有权限
     * number类型: 角色管理-权限列表，0表示都有
     */
    checkAccess,
    /**
     * 检查当前用户是否拥有页面权限
     */
    checkPageAccess,
  };
}
```

## 页面权限

**思路**

1. 通过 wrappers 包一层路由判断
2. router 加上 accessKey，通过 useRouteProps 取
3. 校验方式：校验是否存在当前 code

**实现代码**

`routes.ts`

```tsx
{
  path: '/game',
  name: 'game',
  routes: [
    {
      path: 'sport',
      component: './Game/Sport',
       wrappers: [
        '@/wrappers/auth',
      ],
      accessKey: 1010095000, // 加上权限表的key
    },
    ...
  ],
  ...
}
```

`src/wrappers/auth.tsx`

```tsx
import { useKeepAliveContext } from "@/layout/keepalive";
import NoPermissionPage from "@/pages/403";
import {
  useAccess,
  useRouteProps,
  useIntl,
  useLocation,
  Outlet,
} from "@umijs/max";
import { useEffect, useRef } from "react";

// 解决hooks链路异常问题
const NoPermission = () => {
  const intl = useIntl();
  const { updateTab } = useKeepAliveContext();

  const { pathname } = useLocation();

  useEffect(() => {
    updateTab(pathname, {
      name: intl.formatMessage({ defaultMessage: "无权限" }),
    });
  }, []);

  return <NoPermissionPage />;
};

export default () => {
  // 缓存路由，避免多个tab的情况下，只拿到了当前页面的路由，引起频繁刷新
  const routeRef = useRef(useRouteProps());
  const { accessKey, commonAccessKey } = routeRef.current;

  const { checkPageAccess, checkAccess } = useAccess();

  // 通用页面：首页
  if (commonAccessKey && checkAccess(commonAccessKey)) {
    return <Outlet />;
  }
  // 页面权限：其他页面
  if (!commonAccessKey && checkPageAccess(accessKey)) {
    return <Outlet />;
  }

  return <NoPermission />;
};
```

## 组件权限

**思路**：通过 access 提供的函数来查询当前 code 是否有权限

**实现代码**

```tsx
const { checkAccess } = useAccess()
...
 <Button hidden={!checkAccess('Home')}>
 ...
 </Button>
```

## 受影响功能

1. 菜单搜索功能：无权限的菜单不可检索，主要针对不在菜单内的三四级页面
   1. checkAccess(code) 是否满足，不满足时移除数据
2. 收藏夹：不做处理，页面访问时可以显示 403 即可

# 总结

1. 了解 RBAC 的管理下，权限需要前后台共同来配合
2. 借助现有轮子来实现需求
