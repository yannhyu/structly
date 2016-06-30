# Exercises from Advanced Python Mastery

This repo contains primarily exercises worked on while taking Dave 
Beazley's excellent course--Advanced Python Mastery. The class focused 
on a review of list and dictionary comprehensions, generators, decorators,
extensive use of the collections and itertools libraries, as well as a 
deep dive into Python's builtin functions relative to class manipulation,
and, finally, also metaclasses and concurrency.

My primary takeaways from this course...

* Python's object manipulation via classes is primarily about getting, 
setting, and deleting methods in builtin dictionaries, and overriding 
those dictionaries.

* Python's object instantiation can be hijacked / overridden from object 
through the type class.


####WHICH CONTAINER/PRIMITIVE SHOULD I USE?

Some heuristics...

* If you need things in order, use a list

* If you need uniqueness, use a set

* If you need a lookup table or an indexed data structure, use a dictionary


####DATA MANIPULATION

#####FILTERING AND QUERYING DATA...

If you need to filter or query a data, use a list or dictionary _comprehension_...

#####EXAMPLES

* List comprehension syntax...

    result = [ <expression> for <name> in <sequence> if <filter/conditional> ]
    
Same as...
    
    result = []
    for name in sequence:
        if condition:
            result.append(expression)

'''
>>>data = (1,2,3,4,5)
>>>result = [ x for x in data if x > 3 ]
>>>print result
[4,5]
'''

* Dictionary comprehension syntax...

    result = { <key:value> for <key, value> in <sequence> if <filter/conditional> }
    
Same as...

    result = {}
    for item in dictionary:
        if condition:
            result.update(item)

'''
>>>dataset = {
...'1':'A', 
...'2':'B', 
...'3':'C', 
...'4':'D', 
...'5':'E'
...}
>>>
>>>result = {k:v for k,v in dataset.items() if k == '4'}
>>>print(result)
{'4': 'D'}
'''

* Filtering data from two sets... (Use zip to create a dictionary!)
'''
>>>keys = [1, 2, 3, 4, 5]
>>>values = ['A', 'B', 'C', 'D', 'E']
>>>result = { k:v for (k,v) in zip(keys, values) if k > 3}
>>>print result
{4: 'D', 5: 'E'}
'''

* Filtering a record from a pre-existing _list_ of dictionaries...
'''
>>>dataset = [
...{'id':1, 'data':'Foo'},
...{'id':2, 'data':'Bar'},
...{'id':3, 'data':'Spam'}
...]
>>>
>>>#assign the 'id' as the dictionary's key and attach whole record to it.
>>>results = {record['id']:record for record in dataset}
>>>print results[1]
{'data': 'Foo', 'id': 1}
'''

* Filter unique values by using curly braces in a list comprehension to create a set...
'''
>>>data = (1,2,3,4,4,5,5)
>>>>>>result = { x for x in data if x > 3 }
>>>print result
set([4,5])
'''

####COUNTING

* If you need to count items in a list, use defaultdict Counter
'''
>>>words = ["Foo", "Bar", "Spam", "Ham", "Foo", "Spam", "Foo"]
>>>
>>>from collections import Counter
>>>word_count = Counter(words)
>>>popular_words = Counter.most_common(2)
>>>print(popular_words)
[('Foo', 3), ('Spam', 2)]
'''

* To sum one kind of element in a list of similar tuples...
'''
>>>portfolio = [
...('AA', 100, 32.20),
...('IBM', 50, 91.10),
...('CAT', 150, 83.44),
...('MSFT', 200, 51.23),
...('GE', 95, 40.37),
...('MSFT', 50, 65.10),
...('IBM', 100, 70.44)
...]
>>>
>>> from collections import Counter
>>> total_shares = Counter()
>>> for name, shares, price in portfolio:
...     total_shares[name] += shares
... 
>>> total_shares['IBM']
150
'''

* Or--for an index or running count--just use enumerate...
'''
>>> for lineno, stock in enumerate(portfolio):
...     print(lineno, stock[0])
... 
0 AA
1 IBM
2 CAT
3 MSFT
4 GE
5 MSFT
6 IBM
'''

####CLUSTERING

* If you need to gather data by similar kind, use defaultdict(list)...
'''
rows = [
{'id':1, 'data':'Foo'},
{'id':2, 'data':'Bar'},
{'id':3, 'data':'Spam'},
{'id':4, 'data':'Bar'}
]
>>>
>>>from collections import defaultdict
>>> data_by_rows = defaultdict(list)
>>> for row in rows:
...     data_by_rows[row['data']].append(row)
... 
>>> for r in data_by_rows['Bar']:
...     print(r)
... 
{'id': 2, 'data': 'Bar'}
{'id': 4, 'data': 'Bar'}
...
'''

...or...

* Use a list comprehension, like so...
'''
>>>portfolio = [
...{'name':'AA', 'shares':100, 'price':32.20},
...{'name':'IBM', 'shares':50, 'price':91.10},
...{'name':'CAT', 'shares':150, 'price':83.44},
...{'name':'MSFT', 'shares':200, 'price':51.23},
...{'name':'GE', 'shares':95, 'price':40.37},
...{'name':'MSFT', 'shares':50, 'price':65.10},
...{'name':'IBM', 'shares':100, 'price':70.44}
...]
>>>
>>>#initialize a dictionary where each item is set to zero...
>>>shares = { s['name']:0 for s in portfolio }
>>>shares
{'IBM': 0, 'CAT': 0, 'GE': 0, 'AA': 0, 'MSFT': 0}
>>>for s in portfolio:
...    shares[s['name']] += s['shares']
>>> shares
{'IBM': 150, 'CAT': 150, 'GE': 95, 'AA': 100, 'MSFT': 250}
'''

...or...

* If you don't need to keep the data (more like a generator), you can use itertools.groupby...
'''
>>>portfolio = [
...{'name':'AA', 'shares':100, 'price':32.20},
...{'name':'IBM', 'shares':50, 'price':91.10},
...{'name':'CAT', 'shares':150, 'price':83.44},
...{'name':'MSFT', 'shares':200, 'price':51.23},
...{'name':'GE', 'shares':95, 'price':40.37},
...{'name':'MSFT', 'shares':50, 'price':65.10},
...{'name':'IBM', 'shares':100, 'price':70.44}
...]
>>>
>>>from operator import itemgetter
>>>from itertools import groupby
>>> 
>>>#sort by desired field first because
>>>#groupby only sorts consecutive items
...portfolio.sort(key=itemgetter('name'))
>>> 
>>>#iterate in groups
...for name, items in groupby(portfolio, key=itemgetter('name')):
...    print(name)
...    for i in items:
...        print('    ', i)
... 
AA
     {'name': 'AA', 'shares': 100, 'price': 32.2}
CAT
     {'name': 'CAT', 'shares': 150, 'price': 83.44}
GE
     {'name': 'GE', 'shares': 95, 'price': 40.37}
IBM
     {'name': 'IBM', 'shares': 100, 'price': 70.44}
     {'name': 'IBM', 'shares': 50, 'price': 91.1}
MSFT
     {'name': 'MSFT', 'shares': 200, 'price': 51.23}
     {'name': 'MSFT', 'shares': 50, 'price': 65.1}
'''
        
#####LIST/DICT CONVERSIONS

dict(pair) --> converts list pairs into dictionary
'''
>>>data = [('GOOG',490.1), ('AA', 23.15), ('IBM', 91.5)]
>>>dict(data)
{ 'AA': 23.15, 'IBM': 91.5, 'GOOG': 490.1 }
'''

list(dict) --> converts a dictionary to a list of key names
'''
>>>data = { 'AA': 23.15, 'IBM': 91.5, 'GOOG': 490.1 }
>>>list(data)
['AA', 'IBM', 'GOOG']
'''

list(dict.items()) --> creates a list of key/value pairs
'''
>>>data = { 'AA': 23.15, 'IBM': 91.5, 'GOOG': 490.1 }
>>>list(data.items())
[('GOOG',490.1), ('AA', 23.15), ('IBM', 91.5)]
'''

#####DICTIONARY VIEWS

Dictionaries can be introspected and teased apart with...

* .keys()
* .values()
* .items() ...i.e., the key/value pairs

Note: these functions do not produce copies of the dictionary, but
overlays to the view. Updating the original dictionary will update
the view.


##CODE EXAMPLES: CONTENTS OF THIS REPO

#### structly

Fetches live streaming data from a stockticker. To create the data stream,
requires running the stocksim.py file in the Data directory in a 
separate process / terminal.

#### ticker_follower

Formative files related to the above.

#### bus_data_analysis

Various Pyhonic manipulations of bus route and schedule info.

#### logging

Logging of a function via a decorator.
