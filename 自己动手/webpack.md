## webpack 把JS 文件编译成了什么



目标文件1： 

``` javascript
const aaa = 5 + 5

export default aaa
```

目标文件2： 

``` javascript
import aa from './sub'

const result = 1 + aa

export default result
```

然后 执行 webpack

编译后的文件如下： 

``` javascript
(() => { // webpackBootstrap
  "use strict";
  var __webpack_modules__ = ({

/***/ "./sub.js":
      /*!****************!*\
        !*** ./sub.js ***!
        \****************/
      /***/
      ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {

        __webpack_require__.r(__webpack_exports__);
        /* harmony export */ __webpack_require__.d(__webpack_exports__, {
        /* harmony export */   "default": () => (__WEBPACK_DEFAULT_EXPORT__)
          /* harmony export */
        });
        const aaa = 5 + 5

        /* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = (aaa);

        /***/
      })

    /******/
  });
  /************************************************************************/
  // The module cache
  var __webpack_module_cache__ = {};

  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
      return cachedModule.exports;

    }
    // Create a new module (and put it into the cache)
    var module = __webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {}
      /******/
    };
    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;
    /******/
  }
  /******/
  /************************************************************************/
  /******/ 	/* webpack/runtime/define property getters */
  (() => {
    // define getter functions for harmony exports
    __webpack_require__.d = (exports, definition) => {
      for (var key in definition) {
        if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
          Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
          /******/
        }
        /******/
      }
      /******/
    };
    /******/
  })();
  /******/
  /******/ 	/* webpack/runtime/hasOwnProperty shorthand */
  (() => {
    __webpack_require__.o = (obj, prop) => (Object.prototype.hasOwnProperty.call(obj, prop))
    /******/
  })();
  /******/
  /******/ 	/* webpack/runtime/make namespace object */
  (() => {
    /******/ 		// define __esModule on exports
    __webpack_require__.r = (exports) => {
      if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        /******/
      }
      Object.defineProperty(exports, '__esModule', { value: true });
      /******/
    };
    /******/
  })();
  /******/
  /************************************************************************/
  var __webpack_exports__ = {};
  // This entry need to be wrapped in an IIFE because it need to be isolated against other modules in the chunk.
  (() => {
    /*!******************!*\
      !*** ./index.js ***!
      \******************/
    __webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "default": () => (__WEBPACK_DEFAULT_EXPORT__)
      /* harmony export */
    });
/* harmony import */ var _sub__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./sub */ "./sub.js");



    const result = 1 + _sub__WEBPACK_IMPORTED_MODULE_0__.default

/* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = (result);
  })();

  /******/
})()
  ;
//# sourceMappingURL=main.js.map
```

编译后的文件  分成了几个部分： 

1. `__webpack_modules__`

2. `__webpack_module_cache__`

3. `function __webpack_require__(moduleId)`

4. `__webpack_require__.d`

5. ` __webpack_require__.o`

6. ` __webpack_require__.r`

   

#### `__webpack_modules__`

`__webpack_modules__` 是一个对象 ， 对象的 `key` 是 我们的 js 文件路径 ， 比如 `"./sub.js"`,   对象的 `value` 就是 我们对应的 函数体

``` javascript
var __webpack_modules__ = ({

	 /***/ "./sub.js":
      /*!****************!*\
        !*** ./sub.js ***!
        \****************/
      /***/
      ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {

        __webpack_require__.r(__webpack_exports__);
        /* harmony export */ __webpack_require__.d(__webpack_exports__, {
        /* harmony export */   "default": () => (__WEBPACK_DEFAULT_EXPORT__)
          /* harmony export */
        });
        const aaa = 5 + 5

        /* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = (aaa);

        /***/
      })

    /******/
  })
```





#### `__webpack_module_cache__`

`__webpack_module_cache__` 也是一个对象， 根据名字 ， 就知道 这个对象 用于 缓存模块代码



#### `__webpack_require__`

这个方法，  其实就是 我们 在  `js` 文件中的 `import` , 在 这个 方法中，  首先 检查 ``__webpack_module_cache__` 这个缓存中 有没有 将要   导入 的缓存 ， 如果有 ， 则从  `__webpack_module_cache__` 对象中 获取 ， 如果 没有  则从 `__webpack_modules__`  对象中获取 ，  并且  缓存到 `__webpack_module_cache__`对象中

``` javascript
function __webpack_require__(moduleId) {
    // Check if module is in cache
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
      return cachedModule.exports;

    }
    // Create a new module (and put it into the cache)
    var module = __webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {}
      /******/
    };
    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;
    /******/
  }
```





#### `__webpack_require__.d`

对于 `js`代码中的 `export default` 导出的模块， 就是在 `__webpack_require__.d`方法 中， 利用 `Object.defineProperty` 把 导出的模块  ，挂在到 `export `对象上的

``` javascript
 __webpack_require__.d = (exports, definition) => {
      for (var key in definition) {
        if (
            __webpack_require__.o(definition, key) && 
            !__webpack_require__.o(exports,key)
        ) 
        {
          Object.defineProperty(exports, key, { enumerable: true, get: definition[key] 
        }
       }
  }
```







#### `__webpack_require__.o`

就是 `Object.prototype.hasOwnProperty()`

``` javascript
(
    () => {
    __webpack_require__.o = (obj, prop) => (Object.prototype.hasOwnProperty.call(obj, prop)
)
```







#### `__webpack_require__.r`

 直接看源码上的注释就是 `define __esModule on exports`

``` javascript
__webpack_require__.r = (exports) => {
      if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        /******/
      }
      Object.defineProperty(exports, '__esModule', { value: true });
      /******/
    }
```

