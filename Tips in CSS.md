CSS常用小技巧



1. 去除input 搜索框选中时的边框

   ```css
   outline: medium;
   ```

2. 背景阴影效果

   ```css
   box-shadow: 
   ```


​        http://blog.csdn.net/freshlover/article/details/7610269

3. 使用 path ：fill 来动态改变填充 svg 的背景颜色；

   ```javascript
   css:

   .tag-icon {
       height: 14px;
       width: 14px;
       position: relative;
       top: 2px;
       background-color: unset;
   }

       .tag-icon path {
           fill: #2f9659;
       }
   ```


   <img src="~/Images/newSite/tag_icon.svg" alt="" class="tag-icon" id="tag_back_color_@i" />


```javascript
  <script>
 $(document).ready(function () {
       $('img[src$=".svg"]').each(function () {
           var randomColor = Math.floor(Math.random() * 16777215).toString(16);
           var $img = jQuery(this);
           var imgURL = $img.attr('src');
           var attributes = $img.prop("attributes");
           $.get(imgURL, function (data) {
               var $svg = jQuery(data).find('svg');
               $svg = $svg.removeAttr('xmlns:a');
               $.each(attributes, function () {
                   $svg.attr(this.name, this.value);
                   $svg.find('path').attr('style', 'fill:#' + randomColor);
               });
               $img.replaceWith($svg);
           }, 'xml');
       });
   });
     </script>
```
4. 去除 list 中前面的圆点

   ```css
   .idName ul li{
     list-style:none
   }
   ```

5. css实现圆形边框

   ```css
   border-radius: 50% 50% 50% 50%;
   ```

6. 将textarea 中的 \n 转换成 br 

   ```javascript
   $('.'+contentAttr+'').html(value.replace(/\r?\n/g,'<br/>'));
   ```

   ​

7. ​

   ```javascript
   $(function() { ... });

   is just jQuery short-hand for 

   $(document).ready(function() { ... });
   ```



8. 在页面中直接使用Css

   >```css
   ><style media="screen" type="text/css">
   >
   >Add style rules here
   >
   ></style>
   >```

   ​

9. 在Razor page 中将时间格式转换

   > ```html
   > <span class="repu-time">@repu.DateAdded.ToString("yyyy/MM/dd")</span>
   > ```



10.

> ```javascript
> var pattern = /([(](\d+)[)])/gm;
> var match = $(document).attr("title").match(pattern);
> if (match === null) {
>     $("title").html(`(${count})${$(document).attr("title")}`);
> }
> else if (match[0] !== `(${count})`) {
>     $("title").html($(document).attr("title").replace(pattern, "(" + count + ")"));
> }
> ```
>
> 