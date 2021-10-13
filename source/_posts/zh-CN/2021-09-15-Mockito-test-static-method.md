---
layout:       post
title:        "测试 | 使用 Mockito 测试 static 静态方法"
subtitle:     ""
date:         2021-09-15 14:00:00
updated:      2021-09-15 14:00:00
author:       "Quan Qinle"
header-img:   "img/home-bg.webp"
multilingual: false
catalog:      true
categories:
    - Mockito
    - Unit Testing
tags:
    - Mockito
    - Unit Testing

---

从 3.4.0 版本开始，Mockito 已经可以支持测试静态方法了。

Mocking static methods (since 3.4.0)
When using the inline mock maker, it is possible to mock static method invocations within the current thread and a user-defined scope. This way, Mockito assures that concurrently and sequentially running tests do not interfere. To make sure a static mock remains temporary, it is recommended to define the scope within a try-with-resources construct. In the following example, the Foo type's static method would return foo unless mocked:

 assertEquals("foo", Foo.method());
 try (MockedStatic mocked = mockStatic(Foo.class)) {
 mocked.when(Foo::method).thenReturn("bar");
 assertEquals("bar", Foo.method());
 mocked.verify(Foo::method);
 }
 assertEquals("foo", Foo.method());
 
Due to the defined scope of the static mock, it returns to its original behavior once the scope is released. To define mock behavior and to verify static method invocations, use the MockedStatic that is returned.



<!-- more -->

# Datatables 简介
https://datatables.net/

官方自我介绍：
> Add advanced interaction controls to your HTML tables the free & easy way

# 安装
在 HTML 中添加`DataTables`的 js 和 css。我使用的是当前最新版本 1.10.25。
```html
<link rel="stylesheet" href="https://cdn.datatables.net/1.10.25/css/jquery.dataTables.min.css">

<script src="https://cdn.datatables.net/1.10.25/js/jquery.dataTables.min.js"></script>
```

如需`DataTables`提供的其他功能，请查阅：
- https://datatables.net/extensions/index
- https://datatables.net/download/index

# HTML 中添加 `<table>`
在 HTML 中添加表格标签，并指定 id 以便后续的 JS 可以定位到它并完成表格创建。

```html
<table id="myTable">
</table>
```

# 两种处理模式
`DataTables`有两种操作模式（`serverSide='boolean'`）：客户端处理（`false`）和服务端处理（`true`）。
- 客户端处理：适用于数据量小的场景，一次性获取所有数据，在浏览器中执行分页、过滤、排序等操作。这是默认模式。
- 服务端处理：适用于数据量大的场景，分次调用 API 获取部分数据，由服务端执行分页、过滤、排序。官方推荐数据量大于 50000 时使用该模式，但仁者见仁，个人认为这个值太高了，也许参考值可以调整到 1000。

注意：客户端模式时，除了上面描述的从服务器获取数据之外，数据源还可以直接写死在 HTML 中，也可以从本地静态文件中读取，但这些和我的应用场景不符，所有本文不会记述。

## serverSide='false' 客户端处理
```html
<script>
$(function () {
  $("#myTable").DataTable ({
    serverSide: false,
    columns: [
      { title: "#",       data: "id" },
      { title: "Word",    data: "word" },
      { title: "Meaning", data: "meaning" },
      { title: "Example", data: "example" }
    ],
    ajax: {
      url: "/english/all.json",
      type: "GET",
      contentType: "application/json",
      dataSrc: "data"
    }
  } );

} );
```

代码解读：
- `$("#myTable")` 找到 id 是 myTable 元素
- `serverSide: false` 客户端处理模式
- `ajax` 页面加载后，自动发起 GET 请求，得到 JSON 格式的响应，从 JSON 中获取元素 data 的值作为表格数据
- `columns` 表头取自 title，而 data 对应响应 JSON 中 data 元素的属性名

以下是响应 JSON 示例：
```json
{
  "data":[
      {"id":1,"word":"hello","meaning":"","example":"Hello world"},
      {"id":2,"word":"good","meaning":"not bad","example":"Good morning"}
  ]
}
```

## serverSide='true' 服务端处理
```html
<script>
//
// Pipelining function for DataTables. To be used to the `ajax` option of DataTables
//
$.fn.dataTable.pipeline = function ( opts ) {
  // Configuration options
  var conf = $.extend( {
    pages: 5,     // number of pages to cache
    url: '',      // script url
    data: null,   // function or object with parameters to send to the server
                  // matching how `ajax.data` works in DataTables
    method: 'GET' // Ajax HTTP method
  }, opts );

  // Private variables for storing the cache
  var cacheLower = -1;
  var cacheUpper = null;
  var cacheLastRequest = null;
  var cacheLastJson = null;

  return function ( request, drawCallback, settings ) {
    var ajax          = false;
    var requestStart  = request.start;
    var drawStart     = request.start;
    var requestLength = request.length;
    var requestEnd    = requestStart + requestLength;

    if ( settings.clearCache ) {
      // API requested that the cache be cleared
      ajax = true;
      settings.clearCache = false;
    }
    else if ( cacheLower < 0 || requestStart < cacheLower || requestEnd > cacheUpper ) {
      // outside cached data - need to make a request
      ajax = true;
    }
    else if ( JSON.stringify( request.order )   !== JSON.stringify( cacheLastRequest.order ) ||
              JSON.stringify( request.columns ) !== JSON.stringify( cacheLastRequest.columns ) ||
              JSON.stringify( request.search )  !== JSON.stringify( cacheLastRequest.search )
    ) {
      // properties changed (ordering, columns, searching)
      ajax = true;
    }

    // Store the request for checking next time around
    cacheLastRequest = $.extend( true, {}, request );

    if ( ajax ) {
      // Need data from the server
      if ( requestStart < cacheLower ) {
        requestStart = requestStart - (requestLength*(conf.pages-1));

        if ( requestStart < 0 ) {
          requestStart = 0;
        }
      }

      cacheLower = requestStart;
      cacheUpper = requestStart + (requestLength * conf.pages);

      request.start = requestStart;
      request.length = requestLength*conf.pages;

      // Provide the same `data` options as DataTables.
      if ( typeof conf.data === 'function' ) {
        // As a function it is executed with the data object as an arg
        // for manipulation. If an object is returned, it is used as the
        // data object to submit
        var d = conf.data( request );
        if ( d ) {
          $.extend( request, d );
        }
      }
      else if ( $.isPlainObject( conf.data ) ) {
        // As an object, the data given extends the default
        $.extend( request, conf.data );
      }

      return $.ajax( {
        "type":     conf.method,
        "url":      conf.url,
        "data":     request,
        "dataType": "json",
        "cache":    false,
        "success":  function ( json ) {
          cacheLastJson = $.extend(true, {}, json);

          if ( cacheLower != drawStart ) {
            json.data.splice( 0, drawStart-cacheLower );
          }
          if ( requestLength >= -1 ) {
            json.data.splice( requestLength, json.data.length );
          }

          drawCallback( json );
        }
      } );
    }
    else {
      json = $.extend( true, {}, cacheLastJson );
      json.draw = request.draw; // Update the echo for each response
      json.data.splice( 0, requestStart-cacheLower );
      json.data.splice( requestLength, json.data.length );

      drawCallback(json);
    }
  }
};

// Register an API method that will empty the pipelined data, forcing an Ajax
// fetch on the next draw (i.e. `table.clearPipeline().draw()`)
$.fn.dataTable.Api.register( 'clearPipeline()', function () {
  return this.iterator( 'table', function ( settings ) {
    settings.clearCache = true;
  } );
} );

//
// DataTables initialisation
//
$(document).ready(function() {
  $('#myTable').DataTable( {
    processing: true,
    serverSide: true,
    columns: [
      { title: "#",        data: "id" },
      { title: "Word",     data: "word" },
      { title: "Meaning",  data: "meaning" },
      { title: "Example",  data: "example" }
    ],
    ajax: $.fn.dataTable.pipeline( {
      url: '/english/post/list.json',
      method: 'POST',
      pages: 3 // number of pages to cache
    } )
  } );
} );
</script>
```

代码解读：
- `serverSide: true` 服务端处理模式
- `ajax` 页面加载后，自动发起 POST 请求，得到 JSON 格式的响应，从 JSON 中获取元素 data 的值作为表格数据
- `pages: 3` 一次获取 N 页数据（缓存），用以减少请求的次数
- 实际使用中，只需要修改最后一部分的 columns 和 Ajax 的三个参数

关于 API 的 request 和 response 参数，可以查阅：https://datatables.net/manual/server-side

以下是响应 JSON 示例：
```json
{
  "draw":1,
  "recordsTotal":119,
  "recordsFiltered":119,
  "data":[
      {"id":1,"word":"hello","meaning":"","example":"Hello world"},
      {"id":2,"word":"good","meaning":"not bad","example":"Good morning"}
  ]
}
```

另外需要注意的是，此模式下 API 请求的参数是 Form Data 形式，服务端处理时不是接收 json。例如 Spring Boot 可以通过 `@RequestParam` 接收参数：
```java
/**
 * 分页获取数据
 *
 * @param draw Draw counter.
 * @param start Paging first record indicator. This is the start point in the current data set (0 index based - i.e. 0 is the first record).
 * @param length Number of records that the table can display in the current draw.
 * @return - 
 */
@PostMapping("/post/list.json")
@ResponseBody
@ApiOperation(value = "分页获取数据")
public DatatablesResult getList(@RequestParam int draw, @RequestParam int start, @RequestParam int length) {
    int pageNum = start / length;
    Page<EnglishWord> data = englishService.getList(pageNum, length);
    return DatatablesResult.success(draw, 
                                    new PageInfo(data).getTotalElements(),
                                    new PageInfo(data).getTotalElements(),
                                    data.getContent());
}
```

# 表格的样式与功能

开启搜索与排序
```html
$("#myTable").DataTable ({
  searching: true,
  ordering: true
} );
```

开启分页功能，设置分页样式
```html
$("#myTable").DataTable ({
  paging: true,
  pagingType: "full_numbers"
} );
```

开启调整每页数量
```html
$("#myTable").DataTable ({
  lengthChange: true,
  lengthMenu: [
    [ 10, 25, 50, 100, -1],
    [ '10', '25', '50', '100', 'All' ]
  ]
} );
```

开启保存状态（分页位置，每页数量，搜索，排序等）。当用户刷新浏览器时，仍使用之前的状态
```html
$("#myTable").DataTable ({
  stateSave: true
} );
```

开启操作按钮，如，保存、导出 excel、导出 pdf、打印、调整可见列、调整每页数量
```html
$("#myTable").DataTable ({
  buttons: ["copy", "excel", "pdf", "print", "colvis", "pageLength"],
  lengthMenu: [
    [ 10, 25, 50, 100, -1],
    [ '10', '25', '50', '100', 'All' ]
  ]
} );
```

最后一列显示「编辑」按钮
```html
<script>
$(function () {
  var table = $("#myTable").DataTable ({
    columns: [
      { title: "#",        data: "id" },
      { title: "Word",     data: "word" },
      { title: "Meaning",  data: "meaning" },
      { title: "Example",  data: "example" },
      { title: "Operation",data: "", orderable: false, searchable: false }
    ],
    columnDefs: [
      {
        "targets": -1,
        "data": null,
        "defaultContent": "<button>Edit</button>"
      }
    ]
  } );
  
  $('#english-table tbody').on( 'click', 'button', function () {
    var data = table.row( $(this).parents('tr') ).data();
    alert( "id=" + data.id +" is "+ data.word );
  } );

} );
```
