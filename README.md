Little Smallscript
==================
jsを楽に書きたい。   
Little Smalltalk楽しい。   
というわけでLittle Smalltalk -> Javascriptコンパイラ作る。   
Little Smalltalkの処理系が手に入らなかったので本を見ながら作る。   

基本方針:
---------
* 最初は基本型はjsの基本型のみをサポートする。(ListやらBagやらは作らない)
* 出発点がjsの作成支援なので、最初のバージョンは最適化等は考えず、単純な翻訳機に徹する。
  * function () {} が [] になるだけでも相当恩恵はある。
* メッセージ式はそのままjsのメソッド呼び出しに変換する。
  * 参照: https://gist.github.com/3654115
  * オブジェクトobjへの単項メッセージobj unaryはobj.unary()、キーワード引数array at: 1 put: 2はarray.atPut(1, 2)、二項メッセージは後回し
  * このときunaryやkeyword1_keyword2はjsでobjのコンストラクタのprototypeに定義することとする。
  * 予約語やメソッド名に使えない記号のときは他の名前に変換できる仕組みを用意する。　(辞書?)

目標:
---------
docs/from.stをdocs/to.jsにコンパイルできるようにする。   
to.jsはCoffeeScriptが吐き出すコードを真似たい。


---------


Little Smallscript
==================
Little Smalltalk to Javascript translator.

Direction:
----------
* Support only the Javascript's primary classes like Array and String. 
  * We will not build-in Little Smalltalk unique classes such as Bag, List, Bytearray, etc.
* The first version will not perform any optimization.
* Message expressions are translated into method calling expression.
  * reference: https://gist.github.com/3654115
  * "obj unary" becomes "obj.unary()". "array at: 1 put: 2" becomes "array.atPut(1, 2)". binary messages will not be supported.
  
Goal:
----------
Create a translator that can translate docs/from.st to docs/to.js.
Make resulting javascript code similar to that of CoffeeScript.

Milestones:
----------
5am 10 Sep 2012

from:

```smalltalk
[
  Person <- [ :name | 
    this at: 'name' put: name
  ]. 
  Person at: 'prototype'; at: 'answer' put: [this at: 'name'].
  p <- Person new: 'Yendor'. 
  p answer. 
  p
] value
```

to:

```javascript
(function () {
  Person =  function (name) {
    return  (this).atPut("name", name); 
  };
  (function () { 
    var _receiver = (Person).at("prototype");
    _receiver.atPut('answer', function () {
     return  (this).at("name"); 
    }); 
    return _receiver;  
  })(); 
  p =  (Person).new("Yendor");  
  (p).answer();  
  return  p; 
}).value()
```

!! actual output doesn't have newlines !!

7pm 10 Sep 2012

new LittleSmallscript("
```smalltalk
| Map |
Map <- Object subclass.
Map method: [self at: _map put: (Array new)] at: 'new'.
Map method: [:w :h |
  h timesRepeat: [| row |
    row <- Array new.
    w timesRepeat: [row addLast: (MapCell new)].
    _map addLast: row
  ].
  ^ _map
] at: 'init'.
Map method: [^ _map] at: 'show'.

(Map new: init) show
```
").toJS();

//=>
```javascript
(function () {
  var Map;
  Map =  Object.subclass();  
  Map.methodAt(function () {  
    return  self.atPut(_map, Array.new()); 
  }, "new");  
  Map.methodAt(function (w, h) {  
    h.timesRepeat(function () { 
      var row;
      row =  Array.new();  
      w.timesRepeat(function () {  
        return  row.addLast(MapCell.new()); 
      });  
      return  _map.addLast(row); 
    });  
    return  _map; 
  }, "init");  
  Map.methodAt(function () { 
    return  _map; 
  }, "show"); 
  return  Map.new(init).show(); 
}).call(this);
```
