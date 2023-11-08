+++
title = 'Typescript 疑難雜症大補帖'
date = 2023-11-08T10:40:00+08:00
draft = true
featured_image = 'featured_image.png'
tags = ['Typescript', 'Working Notes']
+++
## 前言
---
這篇主要會紀錄 Typescript 遇到錯誤時，要怎麼做 troubleshooting，會想紀錄是因為踩坑時會一直想要解決問題，但解決後就會忘記把錯誤記錄下來，然後就會再次踩到坑，所以會持續更新。

## Troubleshooting
---
### Cannot find module './xxxx.json'. Consider using '--resolveJsonModule' to import module with '.json' extension.

我們如果在 import JSON 檔時，如果沒有設定的話，基本上 typescript 是不知道怎麼解析的，所以我們可以到 tsconfig.json 做設定，如下

```
{
  "compilerOptions": {
    "resolveJsonModule": true,
  }
}
```

這樣應該就可以解決問題惹

可參考： https://www.typescriptlang.org/tsconfig#resolveJsonModule
---
