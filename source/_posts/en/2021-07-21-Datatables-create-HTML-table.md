---
layout:       post
title:        "Front-end | Use DataTables to create HTML table"
subtitle:     "DataTables provide easy and strong way to create HTTP Table component"
date:         2021-07-21 14:00:00
updated:      2021-07-29 17:00:00
author:       "Quan Qinle"
header-img:   "img/home-bg.webp"
lang:         en
catalog:      true
categories:
    - Front-end
    - JavaScript
tags:
    - Front-end
    - JavaScript

---

A complete HTML Table should at least have pagination, sorting, filtering, exporting and other functions, a third-party library can help simplify the implementation steps.

So let's look at this kind of library, Datatables.
<!-- more -->

# Datatables Introduction
https://datatables.net/

Introduction from offical website:
> Add advanced interaction controls to your HTML tables the free & easy way

# Installment
In HTML, add js and css of `DataTables`. I used the latest version 1.10.25.
```html
<link rel="stylesheet" href="https://cdn.datatables.net/1.10.25/css/jquery.dataTables.min.css">

<script src="https://cdn.datatables.net/1.10.25/js/jquery.dataTables.min.js"></script>
```

If you need other functions provided by `DataTables`, check these links:
- https://datatables.net/extensions/index
- https://datatables.net/download/index

# Add `<table>` in HTML
In HTML, add the `<table>` tag, and give it an id so that the subsequent JS can locate it and complete the table creation.

```html
<table id="myTable">
</table>
```

# Two mode
`DataTables` has two processing mode (`serverSide='boolean'`): Client-side processing(`false`) and Server-side processing(`true`).
- Client-side processing: useful when working with small data sets, obtain all the data at once, and page, filter, sort and so on in the browser. This is the default mode.
- Server-side processing: useful when working with large data sets, obtain part of the data everytime the API is called, and page, filter, sort and so on in the server. The official recommendation is to use this mode when the data is greater than 50000, but personally, I think this value is too high, maybe the reference value can be 1000.

Note: When we use client-side mode, in addition to getting data from the server as above, the data source can also be written directly in HTML or read from a local static file, but these doesn't fit my scenario, so will not be mentioned in this article.

## serverSide='false' Client-side processing
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

Code explanation: 
- `$("#myTable")`: query the element whose id is myTable
- `serverSide: false`: Client-side processing mode
- `ajax`: after the page is loaded, send GET request automatically, and gain the response with JSON format, parse the JSON and get `data` from it, then fill them in table body
- `columns`: table header comes from the `title`, and `data` corresponds to the attribute name of the `data` in response JSON

Response JSON demo:
```json
{
  "data":[
      {"id":1,"word":"hello","meaning":"","example":"Hello world"},
      {"id":2,"word":"good","meaning":"not bad","example":"Good morning"}
  ]
}
```

## serverSide='true' Server-side processing
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

Code explanation: 
- `serverSide: true`: Server-side processing mode
- `ajax`: after the page is loaded, send POST request automatically, and gain the response with JSON format, parse the JSON and get `data` from it, then fill them in table body
- `pages: 3`: gain more than one page at once to cache, to reduce the times of sending requests
- In practice, need only to change the columns in the last part and the three parameters of Ajax

About API's Request and Response parameters, you can check: https://datatables.net/manual/server-side

Response JSON demo:
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

It is also important to note that the parameters of the API request in this mode are `Form Data` form, so what the server receives is not JSON. For example, Spring Boot can receive parameters via `@ RequestParam`
```java
/**
 * Gain paged data
 *
 * @param draw Draw counter.
 * @param start Paging first record indicator. This is the start point in the current data set (0 index based - i.e. 0 is the first record).
 * @param length Number of records that the table can display in the current draw.
 * @return - 
 */
@PostMapping("/post/list.json")
@ResponseBody
@ApiOperation(value = "Gain paged data")
public DatatablesResult getList(@RequestParam int draw, @RequestParam int start, @RequestParam int length) {
    int pageNum = start / length;
    Page<EnglishWord> data = englishService.getList(pageNum, length);
    return DatatablesResult.success(draw, 
                                    new PageInfo(data).getTotalElements(),
                                    new PageInfo(data).getTotalElements(),
                                    data.getContent());
}
```

# Table's style and function

Enable filtering and sorting
```html
$("#myTable").DataTable ({
  searching: true,
  ordering: true
} );
```

Enable paging, set paging type
```html
$("#myTable").DataTable ({
  paging: true,
  pagingType: "full_numbers"
} );
```

Enable length of one page
```html
$("#myTable").DataTable ({
  lengthChange: true,
  lengthMenu: [
    [ 10, 25, 50, 100, -1],
    [ '10', '25', '50', '100', 'All' ]
  ]
} );
```

Enable state save (pagination position, display length, filtering and sorting). When the end user reloads the page the table's state will be altered to match what they had previously set up.
```html
$("#myTable").DataTable ({
  stateSave: true
} );
```

Enable option buttons, such as save, export excel/pdf, print, adjust columns visible, adjust the length of one page
```html
$("#myTable").DataTable ({
  buttons: ["copy", "excel", "pdf", "print", "colvis", "pageLength"],
  lengthMenu: [
    [ 10, 25, 50, 100, -1],
    [ '10', '25', '50', '100', 'All' ]
  ]
} );
```

Add a `Edit` button at the last column
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
