---
author: "doreentseng"
title: "大型 React 專案中 API 層的演進與分層設計"
date: "2026-03-18"
description: "探討在 React 專案規模逐漸擴大時，API 呼叫分散於 Component 層所帶來的維護與架構問題，並分享透過 API、Service、React Query Hooks 與 Component 分層設計重構資料流的實務經驗。"

tags: ["react", "api design", "frontend architecture", "refactoring"]
categories: ["frontend", "react", "architecture"]
series: ["Frontend Architecture Notes"]
ShowToc: true
TocOpen: true
---

🤞 完整範例：[Github Repo](https://github.com/doreentseng/react-data-fetching-architecture-example)
&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;
👻 Live Demo：[點我看 demo](https://react-data-fetching-architecture-ex.vercel.app/)

---

專案被擴充得越來越大時，發現自己埋的坑越來越多。

> 為了讓示範更清晰，本文先略過 TypeScript。

## 前期 API 的呼叫方式

前期開發 React 專案時，我習慣將 API 請求統一集中在資料夾 `/api` 下，由每個 component 單獨呼叫使用。

## 專案規模擴大後出現的架構問題

隨著專案的規模愈來愈大，功能愈來愈複雜時，同一個 API 在多處被呼叫，我開始碰到以下問題：
1. 每個 component 需各自維護 loading / error / data 等狀態，導致狀態管理分散、代碼冗餘
2. 同一支 API 被多處重複呼叫，導致請求重複，造成不必要的 Network、後端壓力及 UX 延遲
3. 資料 cache 的缺失，不同 component 中拿到的資料內容可能出現不一致，需額外處理
4. 資料轉換邏輯分散，難以維護
5. 錯誤處理容易不一致，難以維護，使用者體驗不穩定
6. 跨 component 的資料共用困難，狀態同步成本高，可能需要不斷地通過 props 傳遞或使用 store

⋯⋯

## 架構的重構：API / Service / Hook / Component


因此，我決定重新設計資料存取的架構。將原本由 component 直接呼叫 API 並處理所有相關邏輯的做法，依責任拆分為 API 層、Service 層、Hooks 層與 Component 層。透過職責分離，讓每一層專注於單一責任，進而提升整體的可讀性、可維護性與可擴充性。


- **API 層** → 專注於處理實際的 HTTP 請求與共用設定，例如 base URL、headers、token、以及通用的錯誤攔截等。
- **Service 層** → 負責封裝業務邏輯與資料轉換，將 API 回傳的資料整理為前端實際需要的通用的資料結構。
- **Hook 層**（我使用 TanStack Query 的 useQuery / useMutation）→ 負責資料的獲取與 server state 的管理，包含快取、重新請求、錯誤處理與狀態管理。
- **Component 層** → 只關注 UI 呈現與互動邏輯，不再直接處理 API 呼叫與資料轉換。

## API 層
API 層的核心原則：
1. **專注於資料請求**：所有 HTTP 請求與共用設定集中在這裡，不做任何 UI 或業務邏輯處理。
2. **統一設定**：baseURL、headers、timeout 與 interceptor 全部集中，方便修改與管理。
3. **可測試性與可替換性**：上層 Service / Hook 可以獨立測試，未來若要替換 axios 或換 HTTP client，只需修改這一層。

我將所有 API 的共用設定與具體 API function 分開。使用 `axios` 進行示範。先建立一個共用的 API 配置，集中管理所有 HTTP 請求的基礎設定 `/lib/axiosClient.js`：

```typescript
import axios from 'axios';

const axiosClient = axios.create({
  baseURL: 'https://api.example.com/v1',
  headers: {
    'Content-Type': 'application/json',
  },
  timeout: 10 * 1000,
});

axiosClient.interceptors.request.use(
  async (config) => {
    return config;
  },
  (err) => {
    return Promise.reject(err);
  }
);

export default axiosClient;
```

具體的 API function `/apis/classApi.js`：
```js
import axiosClient from '@/lib/axiosClient';

// 取得班級列表
export const fetchClasses = async () => {
  const { data } = await axiosClient.get('/classes');
  return data;
};
```

`/apis/studentApi.js`：

```js
import axiosClient from '@/lib/axiosClient';

// 取得學生列表
export const fetchStudents = async () => {
  const { data } = await axiosClient.get('/students');
  return data;
};

// 刪除學生
export const deleteStudent = async (studentId) => {
  const { data } = await axiosClient.delete(`/students/${studentId}`);
  return data;
};
```

## Service 層
Service 層介於 API 層與 Hook 層／Component 層之間：
1. **封裝業務邏輯**：把原始 API 資料轉換成符合需求的格式，例如過濾、排序、計算等。
2. **集中錯誤處理**：可在此統一處理 API 錯誤、網路異常或回傳格式問題。
3. **提高可維護性**：當 API 回傳結構改變時，只需修改 Service 層，Component / Hook 不受影響。
4. **整合多個 API**：可在 Service 層將多個 API 結果組合、計算或轉換成複合資料供上層使用。

`/services/classService.js`
```javascript
import { fetchClasses } from '@/apis/classApi';

export const getClassMap = async () => {
  try {
    const classesData = await fetchClasses();

    if (!classesData || !Array.isArray(classesData.classes)) {
      console.warn('班級資料格式錯誤，返回空對照表');
      return {};
    }

    return Object.fromEntries(
      classesData.classes.map(c => [c.id, `${c.grade}年${c.class}班`])
    );
  } catch (error) {
    return {};
  }
};
```

`/services/studentService.js`
```javascript
import { fetchStudents, deleteStudent as apiDeleteStudent } from '@/apis/studentApi';
import { fetchClasses } from '@/apis/classApi';
import { getClassMap } from './classService';
import { BusinessError } from '@/lib/errors';

// 一個 service 呼叫 API 並使用另一個 service
export const getStudents = async () => {
  try {
    const [studentsData, classMap] = await Promise.all([
      fetchStudents(),
      getClassMap(),
    ]);

    return studentsData.students.map(s => ({
      ...s,
      classFullName: classMap[s.classId] || 'Unknown',
    }));
  } catch (error) {
    // 在取得學生列表有問題時，可在這裡統一降級處理
    throw error;
  }
};

// 刪除學生
export const deleteStudent = async (studentId) => {
  try {
    // 驗證 studentId，統一在 studentService 中拋出刪除時找不到 studentId 的錯誤
    if (!studentId) {
      throw new BusinessError('學生 ID 不可為空', 'INVALID_STUDENT_ID');
    }

    await apiDeleteStudent(studentId);
  } catch (error) {
    throw error;
  }
};
```

## Hook 層

Hook 層介於 Service 層與 Component 層之間，它的核心目的就是：把資料獲取與狀態管理邏輯封裝起來，讓 Component 可以專注於 UI：

1. **狀態自動化**：管理 loading / error / stale 等的狀態，例如 isLoading、isError、data、refetch，component 不需要自己寫 useEffect + useState
2. **快取與重新發送**：透過成熟的套件（如 TanStack Query），可以實現資料快取，減少重複請求。
3. **封裝資料獲取**：直接呼叫 Service 層的 function，並返回 ready-to-use 的資料，可以把多個 service 組合的結果封裝成單一 hook，component 只需要拿資料就好。

`/hooks/useUsers.js`
```javascript
import { useQuery } from '@tanstack/react-query';
import { getStudents } from '@/services/studentService';

export const useStudents = () => {
  return useQuery({
    queryKey: ['students'], // Query 的唯一鍵值，用來辨識和快取這筆 'students' 資料
    queryFn: getStudents,
    staleTime: 1000 * 60,
  });
};
```

`/hooks/useDeleteUser.js`
```javascript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { deleteStudent } from '@/services/studentService';

export const useDeleteStudent = (options = {}) => {
  const {
    onSuccess: customOnSuccess,
  } = options;

  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteStudent,

    // 刪除成功後重新獲取學生列表
    onSuccess: (data, studentId, context) => {
      queryClient.invalidateQueries(['students']);
      customOnSuccess?.(data, studentId, context);
    },
  });
};
```

## Component 層
Component 層是最接近 UI 的一層，它的核心職責就是：呈現使用者介面，而不需要知道資料從哪裡來，也不處理複雜的資料邏輯。

簡單說，Component 層專注於「畫面」，把資料邏輯交給 Hook / Service 層處理：
1. **只負責 UI 的呈現**：不直接呼叫 API，也不做資料過濾或組合，所有資料都透過 Hook 拿到 ready-to-use 的狀態。
2. **使用 Hook 層取得資料**：React Query hooks、custom hooks 等都在這裡使用，包含取得的資料和取得資料的狀態等。
3. **回傳使用者互動事件**：例如按鈕觸發 mutation（新增、刪除、更新）；component 不處理資料細節，只呼叫 hook 提供的 mutation。

`/components/StudentComponent.jsx`
```jsx
import { useState, useEffect } from 'react';
import { useStudents } from '@/hooks/useStudents';
import { useDeleteStudent } from '@/hooks/useDeleteStudent';
import { getUserFriendlyMessage } from '@/lib/errors';

function StudentDashboard() {
  const [successMessage, setSuccessMessage] = useState('');

  // 使用 hook 並取得錯誤狀態
  const { data: students, isLoading, isError, error, refetch } = useStudents();
  const {
    mutate: deleteStudent,
    isPending: isDeleting,
    isError: isDeleteError,
    error: deleteError,
  } = useDeleteStudent({
    onSuccess: () => {
      setSuccessMessage('刪除成功！');
    }
  });

  
  // 載入中狀態
  if (isLoading) {
    return <div>載入中...</div>;
  }

  // 錯誤狀態 - 在 Component 層處理錯誤顯示
  if (isError) {
    return (
      <div>
        <div>
          <h3>無法載入學生資料</h3>
          <p>{getUserFriendlyMessage(error)}</p>
          <button onClick={() => refetch()}>
            重試
          </button>
        </div>
      </div>
    );
  }

  return (
    <div>
      <h2>學生列表</h2>

      {/* 刪除成功提示 */}
      {successMessage && (
        <div>{successMessage}</div>
      )}

      {/* 刪除錯誤提示 - inline 顯示 */}
      {isDeleteError && (
        <div>{getUserFriendlyMessage(deleteError)}</div>
      )}

      <ul>
        <li>
          <span>姓名</span>
          <span>性別</span>
          <span>班級</span>
          <span>生日</span>
          <span></span>
        </li>

        {students.length === 0 ? (
          <li>目前沒有學生資料</li>
        ) : (
          students.map((s) => (
            <li key={s.id}>
              <span>{s.name}</span>
              <span>{s.gender}</span>
              <span>{s.classFullName}</span>
              <span>{s.birthday}</span>
              <button
                onClick={() => deleteStudent(s.id)}
                disabled={isDeleting}
              >
                {isDeleting ? '刪除中...' : '刪除'}
              </button>
            </li>
          ))
        )}
      </ul>
    </div>
  );
}

export default StudentDashboard;
```

## 🚀 完整範例 & Demo

本文的所有概念、API / Service / Hook / Component 層分層策略，都已在範例專案中經過補充，完整呈現（但還是沒有加入 `typescript` 喔嘿）。

🤞 完整範例：[Github Repo](https://github.com/doreentseng/react-data-fetching-architecture-example)
&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;
👻 Live Demo：[點我看 demo](https://react-data-fetching-architecture-ex.vercel.app/)
