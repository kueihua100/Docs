## Python tips
### Python:
    pip install SomePackage            # latest version
    pip install SomePackage==1.0.4     # specific version
    pip install 'SomePackage>=1.0.4'     # minimum version
    python -m compileall ./ : will compile current folder's python code into byte code.

### 語句和控制流
    1. if語句，當條件成立時執行語句塊。經常與else, elif（相當於else if）配合使用。
    
    2. for語句，遍列列表、字串、字典、集合等迭代器，依次處理迭代器中的每個元素。
    
    3. while語句，當條件為真時，循環執行語句塊。
    
    4. try語句。與except, finally, else配合使用處理在程式執行中出現的異常情況。
    
    5. class語句。用於定義類型。
    
    6. def語句。用於定義函式和類型的方法。
    
    7. pass語句。表示此行為空，不執行任何操作。
    
    8. assert語句。用於程式調適階段時測試執行條件是否滿足。
    
    9. with語句。Python2.6以後定義的語法，在一個場景中執行語句塊。比如，執行語句塊前加鎖，然後在語句塊執行結束後釋放鎖。
    
    10. yield語句。在迭代器函式內使用，用於返回一個元素。自從Python 2.5版本以後。這個語句變成一個運算子。
    
    11. raise語句。丟擲一個異常。
    
    12. import語句。匯入一個模組或包。常用寫法：
      from module import name, 
      import module as name, 
      from module import name as anothername

### 運算式
    Python的運算式寫法與C/C++類似。只是在某些寫法有所差別。

    1. 主要的算術運算子與C/C++類似。+, -, *, /, //, **, ~, %分別表示加法或者取正、減法或者取負、乘法、除法、整除、乘方、取補、取模。
    
    2. >>, <<表示右移和左移。&, |, ^表示二進制的AND, OR, XOR運算。>, <, ==, !=, <=, >=用於比較兩個運算式的值，
       分別表示大於、小於、等於、不等於、小於等於、大於等於。在這些運算子裡面，~, |, ^, &, <<, >>必須應用於整數。
       
    3. Python使用and, or, not表示邏輯運算。is, is not用於比較兩個變數是否是同一個物件。in, not in用於判斷一個物件是否屬於另外一個物件。
    
    4. Python使用'（單引號）和"（雙引號）來表示字串。與Perl、Unix Shell語言或者Ruby、Groovy等語言不一樣，兩種符號作用相同。
      一般地，如果字串中出現了雙引號，就使用單引號來表示字串;反之則使用雙引號。如果都沒有出現，就依個人喜好選擇.
      %r: in format string, means 全印. ex. a='123', %r->'123', %d->123
      print """ .... """: will print all .... between first """ and second """.
      
    5. 運算式前加r指示Python不解釋字串中出現的\。這種寫法通常用於編寫正規表式或者Windows檔案路徑。
    
    6. Python使用y if cond else x表示條件運算式。意思是當cond為真時，運算式的值為y，否則運算式的值為x。
        相當於C++和Java里的cond?y:x。
    
    7. Python區分列表（list）和元組（tuple）兩種型別。list的寫法是[1,2,3]，而tuple的寫法是(1,2,3)。
      可以改變list中的元素，而不能改變tuple。在某些情況下，tuple的括弧可以省略。tuple對於賦值語句有特殊的處理。
      
    8. dict (or hash) info = {'name':'duncan', 'age':42, 'hight':171, 'weight':75}, 
      取用dict by using the strings before ":",
      eg. print info['name'] = > will output duncan

###
    1. lambda的語法是：
        lambda arg1, arg2, ....: expression
        lambda中arg1、arg2等就相當於定義函式時的參數，之後你可以在expression中使用這些參數
        lambda是運算式，不是陳述句，你在:之後的也必須是運算式，lambda中也不能有區塊，
        這表示一些小的運算任務你可以使用lambda，而較複雜的邏輯你可以使用def來定義。    
