# make radio button checked after loading page and make readonly

## 1. call function after finishing loading page by window.onload
## 2. then get element by id
## 3. insert propert using set attribute
```javascript
// (1)
window.onload = function {
    
   //              (2)                   (3)
    document.getElementById("fast").setAttribute("checked",true);

}
```

## blocking check on radio button
```html
<!-- add below for property -->
onclick="return(false);"
```