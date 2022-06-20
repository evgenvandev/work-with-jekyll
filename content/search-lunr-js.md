---
nav_order: 7
title: Search with lunr.js
---

Для поиска на сайте используем библиотеку [_lunr.js_](https://lunrjs.com/).
Ссылка на [GitHub source](https://github.com/olivernn/lunr.js).

На данном сайте код инициализации скрипта _`lunr.js`_ находится в файле _`/_includes/template/lunr-js.html`_.
Вот полный код файла _`lunr-js.html`_

```javascript
<script src="{{ '/assets/lib/lunr.min.js' | relative_url }}"></script>
<script src="{{ '/assets/js/lunr-store.js' | relative_url }}"></script>
<script src="{{ '/assets/lib/lunr.stemmer.support.js' | relative_url }}"></script>
<script src="{{ '/assets/lib/lunr.ru.js' | relative_url }}"></script>
<script src="{{ '/assets/lib/lunr.multi.js' | relative_url }}"></script>
<script>
/* initialize lunr */
var idx = lunr(function () {
  // use the language (ru)
  // использовать язык (ru)
  //this.use(lunr.ru)
  // the reason "en" does not appear above is that "en" is built in into lunr js
  // причина, по которой "en" не отображается выше, заключается в том, что "en" встроен в lunr js.
  this.use(lunr.multiLanguage('en', 'ru'))
  // then, the normal lunr index initialization
  // затем, обычная инициализация индекса lunr
  this.ref('id')
  this.field('title')
  this.field('text')

  //this.pipeline.remove(lunr.trimmer)

  for (var item in store) {
    this.add({
      title: store[item].title,
      text: store[item].text,
      id: item
    })
  }
});

/* search function */
function lunr_search () {
  var query = searchbox.value;//.toLowerCase();
  /* basic search that supports operators */
  var result = idx.search(query); 
  /* more fuzzy search, but doesn't support operators:
  var result =
    idx.query(function (q) {
      query.split(lunr.tokenizer.separator).forEach(function (term) {
        q.term(term, { boost: 100 })
        if(query.lastIndexOf(" ") != query.length-1){
          q.term(term, {  usePipeline: false, wildcard: lunr.Query.wildcard.TRAILING, boost: 10 })
        }
        if (term != ""){
          q.term(term, {  usePipeline: false, editDistance: 1, boost: 1 })
        }
      })
    });*/
  var resultslist = "<tr><td><p>" + result.length + " Results(s) found</p></td></tr>";
  //resultsdiv.prepend('<p class="">' + result.length + ' Result(s) found</p>');
  for (var item in result) {
    var ref = result[item].ref;
    var searchitem =
      '<tr>'+
          '<td class="">' +
            '<p class="h4"><a href="' + store[ref].url + '">' + store[ref].title+ '</a></p>' +
            '<p class="ms-3">' +
            store[ref].text.split(" ").splice(0,20).join(" ") + '...<br>' +
            '</p></td>' +
      '</tr>';
    resultslist += searchitem;
  }
  resultsdiv.innerHTML = resultslist;
}
/* set elements */
var resultsdiv = document.getElementById('lunrResults');
var searchbox = document.getElementById('lunrSearchBox');
/* get query string */
document.addEventListener('DOMContentLoaded', function(event) {
  if (window.location.search) {
    var queryString = decodeURIComponent(window.location.search.substring(1).split("=")[1]);
    searchbox.value = queryString;
    lunr_search ();
  }
});

</script>
```

В первых двух строках подключаем стандартные файлы для библиотеки _`lunr.js`_:

```javascript
<script src="/assets/lib/lunr.min.js"></script>
<script src="/assets/js/lunr-store.js"></script>
```

## Поддержка поиска по языкам

[Инструкция поддержки поиска только для русского языка](https://github.com/MihaiValentin/lunr-languages#in-a-web-browser)

[Инструкция поддержки поиска для нескольких языков](https://github.com/MihaiValentin/lunr-languages#indexing-multi-language-content)

Следующие три строки подключают файл поддержки языков _`lunr.stemmer.support.js`_, файл поиска только для русского языка _`lunr.ru.js`_, файл поиска на нескольких языках одновременно _`lunr.multi.js`_, так как английский язык поиска используется по умолчанию:

```javascript
<script src="{{ '/assets/lib/lunr.stemmer.support.js' | relative_url }}"></script>
<script src="{{ '/assets/lib/lunr.ru.js' | relative_url }}"></script>
<script src="{{ '/assets/lib/lunr.multi.js' | relative_url }}"></script>
```

Далее инициализируем объект `lunr` функцией. 
В этой функции полю (функции) `use` передаём аргумент `lunr.ru`, если хотим применять поиск только на русском языке или аргумент `lunr.multiLanguage('en', 'ru')`, если хотим применять поиск по нескольким языкам:

```javascript
/* initialize lunr */
var idx = lunr(function () {
  // use the language (ru)
  // использовать язык (ru)
  //this.use(lunr.ru)
  // the reason "en" does not appear above is that "en" is built in into lunr js
  // причина, по которой "en" не отображается выше, заключается в том, что "en" встроен в lunr js.
  this.use(lunr.multiLanguage('en', 'ru'))
  // then, the normal lunr index initialization
  // затем, обычная инициализация индекса lunr
  this.ref('id')
  this.field('title')
  this.field('text')
...
```
