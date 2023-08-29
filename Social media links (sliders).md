
## 問題：為何載入Google Sheet 資料庫後，無法執行`showSlides()`圖片轉輪 動畫的JS？


```js
function showSlides() {

let i;

let slides = document.getElementsByClassName("slider");

let dots = document.getElementsByClassName("dot");

for (i = 0; i < slides.length; i++) {

slides[i].style.display = "none";

}

slideIndex++;

if (slideIndex > slides.length) { slideIndex = 1 }

for (i = 0; i < dots.length; i++) {

dots[i].className = dots[i].className.replace(" active", "");

}

slides[slideIndex - 1].style.display = "block";

dots[slideIndex - 1].className += " active";

setTimeout(showSlides, 5000); // Change image every 2 seconds

  

}
```



```js
const url = 'https://sheets.googleapis.com/v4/spreadsheets/1du7xd_7BgoYUBRIGG7QVlnSLC3NKZk6s4BBjtNwNK_I/values/sheet1?alt=json&key=AIzaSyDXVtiX6u99hVg1KH3XSFmYR-EuTNJXUgU';


fetch(url)

.then(res => res.json())

.then(res => {

const data = res.values;

console.log(data);

  

// 刪除第一個陣列

data.shift();

  

Array.prototype.forEach.call(data, d => {

  
  

let slider = `<div class="sliders mt-5 rounded-full">

<a class="slider fade my-3" href="${d[1]}"><img src="${d[0]}"

class="rounded-3xl hover:shadow-lg shadow-md w-96 sm:w-full"></a>

</div>`;

  

document.querySelector('.sliders').insertAdjacentHTML('beforeend', slider);

  
  
  

})

  

})
```


**Bing (GPT 4):**

由於 `fetch()` 是<u>非同步</u>的，所以在資料完全載入之前，第二段程式碼可能無法找到任何 `.slider` HTML 元素(The DOM)。這可能會導致 `showSlides()` 函數在輪播元素還未完全載入時就被調用，從而導致問題。

為了解決這個問題，您可以將第二段程式碼的調用放在第一段程式碼的 `fetch()` 回調函數中，


```js
fetch(url)
    .then(res => res.json())
    .then(res => {
        const data = res.values;
        console.log(data);

        // 刪除第一個陣列
        data.shift();

        Array.prototype.forEach.call(data, d => {
            let slider = `<div class="sliders  mt-5 rounded-full">
      <a class="slider fade my-3" href="${d[1]}"><img src="${d[0]}"
              class="rounded-3xl hover:shadow-lg shadow-md w-96 sm:w-full"></a>
  </div>`;

            document.querySelector('.sliders').insertAdjacentHTML('beforeend', slider);
        })

        // 在此處調用第二段程式碼
        showSlides();
    })

```




## 觀念：Javascript 的 同步 & 非同步


### 🖼️ 圖解


##### 情境舉例


**非同步 (Asynchronous):** 假如`A`跟`B`要煮菜，既然有兩個人，當然是一起「同時」備料囉～一個負責切蔬菜，另一個人負責切肉，兩人<u>同時</u>開始，即是「非同步」 (Asynchronous)。



![](https://i.imgur.com/zwNOjq3.png)




**同步 (Synchronous):** 如果只有`A`一個煮菜呢？那他當然<u>不可能同時</u>切菜有切肉，所以`A`只能先切菜再切肉（和順序沒有關係）。




![](https://i.imgur.com/TENNGK2.png)



**👩🏻‍💻 筆者碎碎念：** 建議不要中文的名稱來理解，JS 的 同步 (Synchronous) & 非同步(Asynchronous)，否則容易搞混。


##### Bug

我是使用`fetch()`，這個Web API來存取資料庫並回傳資料。`fetch()`是非同步。也就是說，當我的網頁載入時，`fetch()`和`showSliders()`函式的程式碼，會「同時執行」。


![](https://i.imgur.com/5IUc7Ip.png)




```js
// fetch（）
fetch(url)

.then(res => res.json())

.then(res => {

const data = res.values;

console.log(data);

  

// 刪除第一個陣列

data.shift();

  

Array.prototype.forEach.call(data, d => {

let slider = `<div class="sliders mt-5 rounded-full">

<a class="slider fade my-3 w-1" href="${d[1]}"><img src="${d[0]}"

class="rounded-3xl hover:shadow-lg shadow-md w-96 sm:w-full" alt="${d[2]}" srcset="${d[3]}"></a>

</div>`;

  

document.querySelector('.sliders').insertAdjacentHTML('beforeend', slider);

})

})

// 函式
showSlides();
```


結果，載入不了圖片轉輪的幻燈片（HTML 元素/圖片檔案路徑/連結）。


![](https://i.imgur.com/HjPlT1A.png)


為什麼呢？還記得「非同步」的特性嗎？「同時執行」，而不是先執行A，再執行B。當`showSlides()`函式（圖片轉輪的動畫）要執行時，它抓不到幻燈片`.sliders`的HTML元素，因為它和`fetch()`的程式碼是同時、一起執行的。


![](https://i.imgur.com/SJydW9D.png)



關於bug的解釋太複雜？我們煮湯的步驟為例，把`fetch()`想像是切菜（備料）的步驟，把煮湯的步驟想像成是`showSlides()`函式，我們不可能沒有先切菜（備料）再煮湯，對於這個bug也是一樣，`showSlides()`函式抓不到幻燈片的HTML元素，所以當然圖片轉輪無法順利載入在網頁上。



```js
function showSlides() {

let i;

let slides = document.getElementsByClassName("slider"); // The DOM (HTMl 元素)

let dots = document.getElementsByClassName("dot"); // The DOM (HTMl 元素)

for (i = 0; i < slides.length; i++) {

slides[i].style.display = "none";

}

slideIndex++;

if (slideIndex > slides.length) { slideIndex = 1 }

for (i = 0; i < dots.length; i++) {

dots[i].className = dots[i].className.replace(" active", "");

}

slides[slideIndex - 1].style.display = "block";

dots[slideIndex - 1].className += " active";

setTimeout(showSlides, 5000); // Change image every 2 seconds

  

}
```



![](https://i.imgur.com/EzzBHpC.png)



要解決這個bug，需要修改成，先執行`fetch()`的程式碼，在執行`showSlides()`函式（圖片轉輪的動畫），這樣`showSlides()`函式（圖片轉輪的動畫）才抓的到HTML元素。



![](https://i.imgur.com/InTi4OZ.png)




![](https://i.imgur.com/CXmnHs5.png)




##### [MDN 例子](https://developer.mozilla.org/zh-TW/docs/Learn/JavaScript/Asynchronous/Introducing)



其實在MDN也有說明到，「[現在許多 Web API 功能都是使用非同步程式碼的做法，特別是那些要從外部裝置存取或抓取某些種類資源的功能，像是從網路抓取檔案、存取資料庫並回傳資料](https://tinyurl.com/2cqjddcr)」。`fetch()`這個Web API 也是如此。


這個例子，一樣使用`fetch()`這個Web API 。`blob`變數執行時，沒有`response`變數的資料（圖片），所以圖片並不會成功載入到網頁上。



```js
let response = fetch("myImage.png"); // fetch is asynchronous
let blob = response.blob();
// display your image blob in the UI somehow
```



![](https://i.imgur.com/81juGdI.png)


>[MDN | | Fetch API](https://developer.mozilla.org/zh-TW/docs/Web/API/Fetch_API)




**🗂️ Reference**


>[重新認識 JavaScript: Day 26 同步與非同步](https://ithelp.ithome.com.tw/articles/10194569        )


>[MDN | | 非同步的 JavaScript 介紹](https://developer.mozilla.org/zh-TW/docs/Learn/JavaScript/Asynchronous/Introducing)


