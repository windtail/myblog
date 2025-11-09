---
title: "Web Frontend Learning"
date: 2025-11-09T20:42:06+08:00
draft: false
tags: ["tauri", "frontend"]
---

最近使用 tauri 开发一个小工具，学了一些前端知识，也能用antd开发一个有几页的工具了。
下面的叙述未必正确，毕竟我不是一个前端工程师。

## 工具选择

- React (antd)
- Typescript
- bun + vite

## 一些简单的规则

- interface 和 type 都可以用来定义接口类型，一般用 type
- ?.[0] 可以用来访问可能不存在的index，和?.member类似
- !号可以用来assert一个变量不为空，例如 user!

## react或库的使用

### axios

```typescript
import axios from 'axios'

export const API = axios.create({
    baseURL: import.meta.env.VITE_API_BASE_URL
});

API.interceptors.request.use((cfg) => {
    const t = localStorage.getItem('token');
    if (t && cfg.headers) {
        cfg.headers.Authorization = `Bear ${t}`;
    }
    return cfg
})
```

要想 `import.meta.env.VITE_API_BASE_URL` 编辑时提示，应修改 `vite-env.d.ts`，添加以下内容：

```typescript

interface ImportMetaEnv {
    readonly VITE_API_BASE_URL: string;
}

interface ImportMeta {
    readonly env: ImportMetaEnv;
}
```

### 自定义hook

自定义hook就是一个函数，是为了封装逻辑，复用复杂的逻辑。注意每个使用了这个hook的组件，都会存储一份hook内部的状态。
对于需要全局共享的状态，可以使用context。

```typescript
export function useXXX() {}
```

### 自定义context

```tsx
import {createContext, ReactNode, useContext, useEffect, useState} from 'react';

interface AuthContextValue {
    user: User | null;
    loading: boolean,
    login: (username: string, password: string) => Promise<void>;
    logout: () => void;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export const AuthProvider = ({children}: { children: ReactNode }) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);

    const login = async (username: string, password: string) => {
        const {data} = await API.post<AuthResult>('/api/v1/auth', {username, password, auto_login: true})
        localStorage.setItem('token', data.token);

        try {
            const {data} = await API.get<AuthUser>('/api/v1/user')
            setUser(data)
        } catch (e) {
            localStorage.removeItem('token');
            throw e
        }
    };

    const logout = () => {
        setUser(null);
        localStorage.removeItem('token');
    };

    const check = async () => {
        try {
            const {data} = await API.get<AuthUser>('/api/v1/user')
            setUser(data)
        } catch (e) {
            console.error(e)
            setUser(null)
            localStorage.removeItem('token');
        } finally {
            setLoading(false)
        }
    }

    useEffect(() => {
        const t = localStorage.getItem('token');
        if (t) {
            void check();
        } else {
            setLoading(false);
        }
    }, [])
    
    const value: AuthContextValue = {
        user,
        loading,
        login,
        logout,
    };

    return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
    const context = useContext(AuthContext);
    if (!context) {
        throw new Error('useAuth 必须在 AuthProvider 内使用');
    }
    return context;
};

export default AuthContext;
```

```tsx
import {ReactNode} from "react";
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

interface ProtectedRouteProps {
    children: ReactNode;
}

const ProtectedRoute = ({ children }: ProtectedRouteProps) => {
    const { user, loading } = useAuth();

    if (loading) return <div>Loading auth...</div>;
    return user ? children : <Navigate to="/login" replace />;
}

export default ProtectedRoute;
```

然后就可以这样包装一下组件，使得登录后才能访问。

```tsx
const HomePage = () => {
    return (
        <ProtectedRoute>
            <AuthenticatedHomePage/>
        </ProtectedRoute>
    );
};
```

### [vite-plugin-pages](https://github.com/hannoeru/vite-plugin-pages)

在`src/pages`文件夹下创建文件，就会自动生成路由，然后可以在函数中这样写来实现跳转

```typescript
import { useNavigate } from 'react-router-dom';
const navigate = useNavigate();
navigate('/xxx');
```

也可以使用 `Navigate` 组件来实现跳转，如

```tsx
import { Navigate } from 'react-router-dom';
return <Navigate to="/xxx" replace />;
 ```

其中 replace 表示是否替换当前路由，可以防止浏览器的回退按钮，返回到上一个页面。

vite-plugin-pages 需要按官方文档配置一下，才能正确生成路由。

```tsx
import { StrictMode, Suspense } from 'react'
import { createRoot } from 'react-dom/client'
import {
  BrowserRouter,
  useRoutes,
} from 'react-router-dom'

import routes from '~react-pages'

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      {useRoutes(routes)}
    </Suspense>
  )
}

const app = createRoot(document.getElementById('root')!)

app.render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
)
```

```ts
// vite-env.d.ts
/// <reference types="vite-plugin-pages/client-react" />
```

> 注意：一定要修改 vite-env.d.ts，否则就会 tsc 会报错！！
> vite-plugin-pages 没法自动生成带路由守卫的路由，但可以在生成之后，修改生成的 routes 对象。
