---
layout: post
title:  在 JavaScript 中使用 Fetch API 和 FormData 進行檔案上傳
date:   2021-01-24
excerpt: "使用 Fatch API 和 FormData"
category: JavaScript
tag:
- JavaScript
- upload
- file
- fetch
- FormData
comments: true
---

## 使用 FormData

對於 FormData，MDN Web Docs 有一段定義：

> The FormData object lets you compile a set of key/value pairs to send using XMLHttpRequest. It is primarily intended for use in sending form data, but can be used independently from forms in order to transmit keyed data. The transmitted data is in the same format that the form's submit() method would use to send the data if the form's encoding type were set to multipart/form-data.

FormData 實際上是物件（Object），使用鍵值對（key and value）的構造方式來生成表單，並通過 XMLHttpRequest 傳送表單資料。

js
```js
const uploadFile = async(data) => {
    let formData = new FormData();
    formData.append("name", data.name);
    formData.append("file", data.file);

    try {
        const resource = "http://[server-ip]/file/upload"; // use your own URL
        const requestInfo = {
            headers: {
                "Accept": "application/json"
            },
            method: "POST",
            body: formData
        }
        const resp = await fetch(resource, requestInfo);

        if (resp.ok) {
            // your code...
        } else {
            // code when ok is false...
        }
    } catch(e) {
        throw e;
    }
}
```

---

## Reference
* [FormData](<https://developer.mozilla.org/en-US/docs/Web/API/FormData> "Click")