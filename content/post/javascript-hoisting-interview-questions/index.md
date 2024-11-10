+++
title = '與 Hoisting 技術相關的知識'
date = 2024-11-10T19:40:00+08:00
draft = false
featured_image = 'featured_image.png'
tags = ['Frontend', 'JavaScript', 'Interview']
+++

## Overview

- 什麼是提升 (Hoisting)
- 提升 (Hoisting) 的運作方式
- 文章中沒解釋的進階知識

## 什麼是提升 (Hoisting)

- 變數與函式在程式碼執行之前被『Hoisting』到作用域的頂部，這樣可以確保在執行程式時，所有變數與函式宣告都已經準備好可以使用
- Hoisting 的設計主要是為了讓開發者在撰寫程式碼時更有彈性，最常見的是函式之間可以互相呼叫而不會導致錯誤

## 提升 (Hoisting) 的運作方式

- JavaScript 是一個解釋型語言，解釋型語言指的是透過直譯器 (Interpreter) 逐行讀取並執行程式碼的語言，不過現今的 JavaScript 是透過 V8 引擎進行處理，V8 引擎提供『即時編譯(Just-In-Time Compilation, JIT)』，過程並不是逐行解釋程式碼而是先掃描整個程式碼，進行語法解析並生成機器碼(Bytecode)，在程式碼執行過程 (runtime) 會監控執行比較頻繁的程式碼將其編譯成機器碼以提升運行效能
    - 現行的框架多數提供『提前編譯 (Ahead-Of-Time Compilation, AOT)』讓程式碼在部署前就提前編譯成機器碼，透過機器碼提升程式碼”啟動”的速度
    - JIT 側重於運行時的性能提升：因為會在執行過程中動態針對頻繁執行的程式碼進行深度優化，使長時間運行的應用程式 (Node.js Server) 效能更高
    - AOT 側重於啟動時的速度提升：因為提前將程式碼轉為機器碼，適合在意載入時間的應用程式 (網頁與行動裝置)
- **Hoisting 發生於 JavaScript 編譯階段，而不是執行階段**
    - 當 JavaScript 編譯器掃描程式碼時，會將所有變數宣告和函式宣告『Hoisting』至作用域的頂部，這些宣告都會被記錄到記憶體中，此過程即為『Hoisting』，需注意的是，Hoisting 只 Hoisting 變數的宣告不 Hoisting 賦值，因此程式碼執行前
        - `var` 宣告的變數會被初始化為 `undefined`
        - `let` 和 `const` 則會因『暫時性死區 (Temporal Dead Zone, TDZ)』的原因，若在宣告前使用該變數，會觸發 `ReferenceError`
- 參考以下實際例子的 Hoisting，主要是傳達概念，實際運作會更複雜
    - 使用 var 的變數在宣告前就呼叫
        - 編譯前
            
            ```jsx
            console.log(a);
            var a = 10;
            console.log(a);
            ```
            
        - 編譯後並執行
            
            ```jsx
            var a;
            console.log(a); // undefined
            a = 10;
            console.log(a); // 10
            ```
            
    - 使用 let, const 遇到 TDZ
        - 編譯前
            
            ```jsx
            console.log(a);
            let a = 10;
            ```
            
        - 編譯後並執行
            
            ```jsx
            console.log(a); // ReferenceError: Cannot access 'a' before initialization
            let a = 10;
            ```
            
    - 使用 Function Declaration
        - 編譯前
            
            ```jsx
            greet();
            function greet() {
            	console.log("Hi")
            }
            ```
            
        - 編譯後並執行
            
            ```jsx
            function greet() {
            	console.log("Hi")
            }
            greet(); // Hi
            ```
            
    - 使用 Function Expression
        - 編譯前
            
            ```jsx
            hello();
            greet();
            
            var hello = function () {
            	console.log("Hello!");
            }
            const greet = function() {
                console.log("Hello!");
            };
            ```
            
        - 編譯後並執行
            
            ```jsx
            var hello
            hello(); // TypeError: hello is not a function
            greet(); // ReferenceError: greet is not defined
            
            hello = function () {
            	console.log("Hello!");
            }
            const greet = function() {
                console.log("Hello!");
            };
            
            ```
            

## 文章中沒解釋的進階知識

- 作用域 (Scope)：JavaScript 中的作用域包括全域作用域、函式作用域和區塊作用域