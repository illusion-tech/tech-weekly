# 前端如何优雅地中断网络请求

> 在前端开发中，网络请求无处不在。但有时候，由于各种原因（如用户取消操作、页面跳转等），我们需要中断正在进行的网络请求（本文只针对前端实现）。

## 一、为什么要中断网络请求？

在前端开发中，我们经常需要发送各种网络请求以获取数据、上传文件或执行其他操作。然而，在请求过程中，可能会遇到以下情况：

- 用户取消了操作，此时不需要继续等待请求的响应。
- 页面进行了跳转，原来的请求已经不再需要。
- 网络请求超时，需要重试或取消。

在这些情况下，如果我们不中断网络请求，可能会导致以下问题：

- 浪费网络资源和服务器资源。
- 可能导致页面卡顿或性能下降。
- 用户体验不佳，如页面跳转后仍在等待旧请求的响应。

因此，学会中断网络请求是前端开发中的一项重要技能。

## 二、如何中断网络请求？

在 JavaScript 中，我们可以使用 **XMLHttpRequest** 或 **Fetch API** 来发送网络请求。对于这两种方式，中断请求的方法略有不同。

---

### XMLHttpRequest

对于 **XMLHttpRequest** 对象，我们可以使用 `abort()` 方法来中断请求。示例代码如下：

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState == 4) {
    if (xhr.status == 200) {
      // 处理响应数据
    } else {
      // 处理错误
    }
  }
};

// 假设在某个时刻，我们需要中断请求
xhr.abort(); // 调用abort方法中断请求
```

---

### Fetch API

对于 **Fetch API**，由于其返回的是一个 `Promise` 对象，我们无法直接调用类似于 `abort()` 的方法来中断请求。但我们可以使用一些技巧来实现类似的功能。

一种常用的方法是使用 `AbortController` 和 `AbortSignal`。`AbortController` 提供了一个 `signal` 属性，该属性是一个 `AbortSignal` 对象。我们可以将 `signal` 对象作为参数传递给 fetch 请求的 `options` 对象。当需要中断请求时，我们可以调用 `AbortController` 的 `abort()` 方法。示例代码如下：

```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch('https://api.example.com/data', { signal })
  .then((response) => {
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json();
  })
  .then((data) => {
    // 处理响应数据
  })
  .catch((error) => {
    if (error.name === 'AbortError') {
      // 请求被中断
    } else {
      // 处理其他错误
    }
  });

// 假设在某个时刻，我们需要中断请求
controller.abort(); // 调用abort方法中断请求
```

---

### axios

在使用 **axios** 进行网络请求时，中断请求的需求同样存在。**axios** 提供了几种方法来优雅地中断正在进行的网络请求。以下是具体的方法：

1. 使用 `CancelToken`

在 **axios** 的早期版本中（如v0.22.0之前），我们可以使用 `CancelToken` 来中断请求。这包括使用 `CancelToken.source()` 方法创建一个取消源，然后将该源的 `token` 作为请求配置的一部分传递给 **axios**。

```javascript
// 创建一个取消源
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

// 创建请求时传入取消令牌
axios.get('/data', {
  cancelToken: source.token
}).catch((thrown) => {
  if (axios.isCancel(thrown)) {
    console.log('请求被取消:', thrown.message);
  } else {
    // 处理错误
  }
});

// 当需要取消请求时，调用 source.cancel()
source.cancel('用户取消了请求');
```

除了使用 `CancelToken.source()` 方法外，我们还可以通过构造函数形式来创建取消令牌和取消函数。

```javascript
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/data', {
  cancelToken: new CancelToken(function executor(c) {
    // 取消函数作为参数传入
    cancel = c;
  })
});

// 当需要取消请求时，直接调用 cancel()
cancel();
```

2. 使用 `AbortController`

从 **axios** 的 v0.22.0 版本开始，支持了使用 `AbortController` 来中断请求。

```javascript
// 创建一个 AbortController 实例：
const controller = new AbortController();

// 发送请求时传入 signal 属性
axios.get('/data', {
  signal: controller.signal
}).then((response) => {
  // 处理响应
}).catch((error) => {
  if (error.name === 'AbortError') {
    console.log('请求被中止');
  } else {
    // 处理其他错误
  }
});

// 当需要取消请求时，调用 controller.abort()
controller.abort();
```
---

> 如果你喜欢用 **Angular**，那么你可以使用 **rxjs** 更加优雅地实现这个功能，示例代码如下：

```javascript
import { Component, OnDestroy } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subject, takeUntil } from 'rxjs';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html'
})
export class ExampleComponent implements OnDestroy {
  // 用于取消订阅地 Subject
  private destroy$ = new Subject<void>();

  constructor(private http: HttpClient) {}

  makeRequest() {
    this.http.get('https://api.example.com/data')
      .pipe(takeUntil(this.destroy$)) // 当 destroy$ 发出信号时，取消订阅
      .subscribe(
        ...
      );
  }

  // 在需要取消订阅时给 destroy$ 发出信号
  cancelRequest() {
    this.destroy$.next();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## 三、总结

中断网络请求是前端开发中的一项重要技能。通过合理地中断请求，我们可以避免浪费资源、提高页面性能和用户体验。
