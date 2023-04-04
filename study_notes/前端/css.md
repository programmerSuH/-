# CSS简介

- CSS（**C**ascading **S**tyle **S**heet）：层叠样式表
- CSS是一门语言，用于控制网页表现



# CSS导入html

1. 内联样式

   ```css
   <div style="color:red">Hello CSS</div>
   ```


2. 内部样式

   ```css
   <style type="text/css">
       <div>{
           color:red;
       }
   </style>
   ```

3. 外部样式

   ```
   <link rel="stylesheet" href="demo.css">
   ```

   demo.css

   ```css
   div{
   	color:red;
   }
   ```

   

# CSS选择器

1. 元素选择器

   ```css
   div{color:red}
   ```

   

2. id选择器

   ```css
   #name{color:red}
   
   <div id="name">hello css</div>
   ```

   

3. 类选择器

   ```css
   .cls{color:red}
   
   <div class="cls">hello css</div>
   ```

   