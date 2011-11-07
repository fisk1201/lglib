#Appendix lglib 

##Introduction to lglib
lglib is a cornerstone for the lua-based web framework [bamboo](https://github.com/daogangtang/bamboo). Generally speaking, it provides a high-level API for customized use on the following aspects:   

1. Extension of several lua standard libraries, like string, table and io   
2. Add several data structures for use, i.e., Dict, Set, List, Queue, etc   
3. Implementation of Object-Oriented Programming mechanism     
4. Global helper functions, like http-related methods       

Injecting methods of lglib.string, lglib.table, lglib.io into namespaces of lua standard libraries, string, table and io separately. This means that we extend three lua standard libraries a little bit. 

Object is the final parent for all lua class instances. BY local Dog = Object:extend{....}
List: list API should be splited into list and queue/stack further 
Dict: map API, its sorted version should be also developed
Set: set API, its sorted version should be also constructed
T: config a prototype for given lua instance/object ??????

setProto()
config a prototype for a given object/instance

typename()
returning type name of data structures, like List, Dict, Table, Set, etc

isList()
checking whether an object is an instance of LIST

isDict()
checking whether an object is an instance of DICT

isSet()
checking whether an object is an instance of SET


checkType()
checkType(a, b, c, 'string', 'table', 'number')
works only for basic data type, boolean, number, string, table, function, etc. 
It should be extended to user-defined data type, like Dict, set, list, queue, etc  ?????

checkRange()
checkRange(a, 0, 10, b, 20, 30, c, 10, 100)


isFalse()
nil, false, 0, "", {}, etc
FOR conditional testing????

printf()
printf as an alias of string.format()

ptable()
print one-layer of table information

fptable()
print all details of table in the nested structure

seri(obj)
serialize lua instances/objects into a lua-table format

unseri(obj_str)
deserialization is just loading serialized string into memory
the point is that format of serialzation are just lua code of assignment of lua-tables


deserialization/serialization

API, usage, (implementation)

### Data structure 
####table ----to be reused in terms of implementation rather than interface  
extension of lua-table in the standard library
To add table_length parameter, just wrapping the simple methods of lua-table insert(), remove() and modifying the constructor{}, t["abc"] = "def" and t["abc"] = nil with changing the #len. This extra parameter can accelerate performance of many operations, such as, equal(), merge(), difference(), etc.

BTW, Lua-5.2 add __len metamethod for lua-table
http://www.lua.org/work/doc/manual.html#3.4.6
for now, we can design the good api firstly. The implementation could be improved later on.

tbl:isEmpty(): true or false returned
true for no any element, while false for at least one element.
 

*tbl:equal(another)*: true or false returned
equal by value or reference. Now the method is implemented only in one-layer. Therefore, for a simple plain table, it is equal by value; for table with nesting structures, it is equal by reference for such fields. Be careful that value and reference are mixed in the sense of semantics.


*tbl:copy()*: new copy of table returned, but sharing nested tables with the orginal version. 
only one-layer copying. For nesting structure, nested tables are shared by the origin and copy of tables.


tbl:deepCopy(): new copy of table
make an exact value-copy of one table, and there is no reference shared any more


*tbl:update(source, keys)*:
if keys is nil, update "tbl" table by all keys of "source" table. Otherwise, only keys in a "keys"-list are updated.


private API??
tbl:merge():semi-union and intersection operations for two tables 
tbl:difference():semi-complement and symmetric difference operations
take care of the relative order of two tables, because there are overwritten issues for common keys/indice (usually number or string) between two tables.   

*tbl:takeAparts()*: 
spliting a lua-table into two parts, list-part and dict-part. 
The underlying reason of this spliting is that lua-table is implemented as a combination of array and hash-table internally. On one hand, it is much better to be used as a combination, like queue/stack. On the other hand, using them separately could be also preferred, like keeping the inserted order (list-type) and much faster inserting/deleting operations (dict-type or set). Therefore, depending on your user case, you should choose to use a proper type, set(ordered or unordered), dict(ordered or unordered), list or queue. <ordered by what??? also should be considered much seriously>

C-array---->list and hash table ---->dict.


The next question you may ask follows as. "how does lua store the data of lua-table?" IF table is a sequence or sequence part of lua-table, that is, the set of its positive numeric keys is equal to {1..n} for some integer n, it will store this info into a C-array internally. we call this part as list-type because of its stroage. For all other cases, it is treated as dict-type. For example, any key that is not one of {1..n} will be stored in hash table. Also, if a key with value m (m>n), hole case, will be also treated as dict type. 
t = {1,2,3,4, one = 1, two = 2, three = 3, 5}
t[0] = 0
t[-1] = -1
t[10] = 10

where 1, 2, 3, 4 and 5 are stored as array/list, and others including 0->0, -1->-1 and 10->10 are stored as dict/hash table. 

One more thing that you should keep in mind is the spliting here done only in one layer. The list/dict type that you get can also contains nesting lua-table internally. SOMETIMES I think one-layer is enough. IF we do not use lua-table directly at all???  or try to use it as little as possible.

####list
list-part
initialization
local lista = List()  
local listb = List {1,2,3,4,5,6}  
    
@class method
List.range(start, finish)
start parameter is optional, and the default value is 1. Both sides are inclusive and a sequence of integers from "start" to "finish" are generated.

mapn() and zip() CAN NOT be understood yet. 
transform() and map() 

@instance methods
List:splice(idx, list)
insert another *list* at the location *idx*


List:sliceAssign(i1, i2, seq)
assignment in the style of slicing

__eq()
test whether two lists are equal or not, and can be written as l2 == l1

isEmpty()
whether a list is empty or not 

isList()
checking whether it is a instance of List prototype BY checking the typename = LIST where LIST is a global constant?

List:extend(another) [is a special case of List:splice(idx, list)]
list expansion by another one, appending at the tail of "self" list. it can be written as lnew = l1 + l2.

List:iremove (i)
delete by index

List:remove(x) [is a special csae of List:remove(x, numOfDeletion)]
delete elements that have the value "x"

List:chop(i1,i2)
deleted by indexing interval 

List:find(val, idx)
starting from idx index and trying to find the first element with value=val

List:contains(x)
check whether a list instance has the element "x"

List:count(x)
counting times that element "x" repeats in the list "self"

List:join(sep)
simpe wrapper of table.concat() method

List:sort(cmp)
sort a list w.r.t. an order function "cmp", and it is a simple wrapper of table.sort(orderFunction)

List:reverse()
reversing the element order 

List:slice(start, stop, is_rev)
-- select a piece of list with index from "start" to "stop"
-- start, stop maybe nil, negative integer, or other values. the default value of "start" is 1, while #list for "stop" 
-- if is_rev is "rev", the returned list is in reversing order.


List:clear()
clear all elements of list

List:len() [storing and maintaining an extra parameter is a better choice]
return the size/length of list. 

queue-part [implemented as a lua-table]
List:append(val)
appending an extra element at the tail of list

List:prepend(val)
appending an extra element at the head of list
 
List.push
push a new element into list at the right-hand side

List:pop
pop a element from list at the right-hand side

List:clear()
clear all elements of list

List:len()
return the size/length of list


####dict
Dict is the prototype of all Dict instances. It inherits extended version of lua-table. [remove this inheritance relationship] 

initializing a Dict object 
dicta = {}
dictb = {one = 1, two = 2, three = 3, four = 4}

suggestion: 
call table.takeAparts() method directly, and initialize Dict instance by returned Dict part. At the same time, you can make sure/guarantee that the instance feature can be kept all the way. 
 
delete the is_key() method, 
add insert() for checking the dict feature
add extra length parameter when initialization

insert()/delete()


**relationship between Dict and table**
Try to implement the Dict API by table-extended methods

isDict()
for non-empty dict, true means Dict instance, false as non-Dict instance.

Dict:keys()
return all the keys of Dict instance, but without keeping any order because of the feature of hash table.

Dict:values()
return all the values of Dict instance, still without keeping any order
 
Dict:hasKey()
check whether the specific key exist in the Dict object or not

Dict:size()
return the length of Dict instance 


####set 
Set is the prototype of all Set instances. It inherits the prototype of Dict. [remove the inheritance relationship, now Set is at the same foot of Dict.]

initializing a Set object 
seta = {}
setb = {"one", "two", "2", "three", "four"}
seta:add("12") NOT seta:add(12)

when encounting positive integers as elements, you should first explicitly convert them into string and then do necessary operations. This requirement results from the consideration of performance. We would like to store all elements into hash table internally. The same argument also holds for Dict type.

basic operations
add(key)
seta:add("23") --->{"23" = true}

delete(key)
delete one element "key"

Do not use the table default API, like seta["23"] = true or seta["abc"] = nil. Actually, methods add() and delete() are implemented in this way internally, that can be treated as raw API. If the size of set is stored as a parameter, we should maintain such a parameter in add() and delete() that can not be easily done in raw API.  <this is another way to handle the implementation of extra length parameter>

has(key)
test whether a element "key" exist or not

members()
return all of elements in random order

Five logical operations of set
Set:union(another)
standard union operation, can be written as A + B

Set:intersection(another)
standard intersection operation, can be written as A * B

Set:difference(another)
standard difference operation, can be written as A - B

Set:symmetricDifference(another)
standard symmetric difference operations, can be written as A ^ B

Set:isSub(another)
check whether "self" is a sub-set of "another" or not
can be written as A < B

Q1- use nil instead of false
Q2- how to make sure list type
Q3- how about adding an extra parameter, length/size of set?
Q4- Call isEmpty() from table?? simple wrapper will be much better.



####extra advanced ordered data structure, like self-balanced binary tree
collection(list, queue, set(sorted set))
map(sorted map)---ordered by key or ordered by value
http://loop.luaforge.net/library/collection/PriorityQueue.html

lua-table with extension can be treated as building blocks for interface of [collection(list, queue, set(sorted set))
map(sorted map)].

In addition to single key-value pair, each table treated as one node also holds extra weight parameter! In the constructor, client should choose/provide a weight parameter and related order function. THEN all methods will refer to them when insert/delete/iteration/has/etc. Once implemented, sorted set (ordered by keys or weight parameter) and sort dict/map (ordered by keys, values or weight parameter) are just simple wrappers or special cases of this implementation. In some sense, this implementation is at the same foot of lua-table.

### Object-Oriented Programming
new()
extend()
1. single-rooted structure? From the statement, "p == Object or not p", it seems that the parent of some class is not Object!!
2. __index: override issue, it just searches its metatable in the backward. 
3. the length of inheritance chain should be not too long? ALSO, new() and extend() methods should be treated as moudle methods to be called directly rather than jumping several times.
4. init() process along the inheritance relationship

fields[__tag, __parent, usual ones], special methods[new() and extend()], magic methods and ordinary methods ????

5. clone() only do it in one-layer deep
6. hashCode() is used to be equal().. equal by value or reference ??



### Extended several modules
string
add/modify definitions of magic methods
__mul, __mod, __div, __add

cap(self)
Capitalizing first letter of word "self" 

contains(self, substr)
check whether string "self" contains "substr" or not 

startsWith(self, begin)
check whether string "self" starts with "begin" or not

endsWith(self, tail)
check whether string "self" ends with "tail" or not

split(self, delim, count, no_patterns)
-- spliting a given string by a delimiter
-- @param self 		splited sting
-- @param delim		delimiter
-- @param count	 	how many times that the delimiter could be replaced
-- @param no_patterns   true|false|nil    whether turn off regular expression in delimiter or not 
-- @return rlist 	list of splited pieces

splitOut(self, delim, count, no_patterns)
unpack a list of splited pieces that returned by split()

splitBy(self, ...)
spliting a given string by several delimiters

rfind(self, substr)
find location of the last substring in a given string

ltrim(self)
trim out the blank space at the head of string "self"

rtrim(self)
trim out blank space at the tail of string "self"

trim(self)
trim out blank space at both sides string

replace(self, ori, new, n)
replace substring with new substring

mapreplace (self, mapping, n)
multiple replacements by mapping from old substring to new one


index(self, i)
select a letter from given string 

slice(self, i, j)
select a continous piece of letters from given string


UTF8 encoding and decoding 
every string generated by lua will be encoded in UTF8???

Lua can also handle UTF8 bytes directly


####input/output
loadFile(from_dir, name)
loading files into memory and the basic unit is file

loadLines(source, firstline, lastline)
Loads a source file that indexed from "firstline" to "lastline", and also prefix line number for each line;
the basic unit is line 


####http
escapeHTML(s)
simple HTML escape sequence, like converting "<" into "&lt;"

 
encodeURL(url)
simple URL encoding  


decodeURL(url)
Simplistic URL decoding that can handle "plus" and "space" encoding too. ???


parseURL(url, sep)
parse parameters of URL and store key-value pairs into lua-table


