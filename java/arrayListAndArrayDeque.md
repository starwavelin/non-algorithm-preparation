# ArrayList, ArrayDeque and LinkedList

### ArrayList
Resizable Array Implementation  
Uses Array in the underground  

#### Interview Questions
1. How does ArrayList grow to itself?
2. What is the growth rate?
3. Time Complexity of appending? -- Amortized O(1) cuz resizing may happen
4. Time Complexity of removing? -- O(n)
For Question 1 and 2, see the section "Resize"

#### Frequently Used Methods
**add(object)** - which is append, Amortized O(1)   
**add(index, object)** - O(n) per src code, the index should be within the capacity and once insert in that index, we need to shift elements after that index by 1.  
**get(index)** - O(1)   
**set(index, object)** - O(1)  
**remove(index)** - O(n) again cuz array/arrayList doesn't allow empty cell, and due to shifting  
**size()** - O(1)

#### Resize
When starting, an arrayList has size = 0 and elementData.length = 0; then when you add one object into this arrayList, size = 1 and elementData.length = 10. Then when size = 10, arrayList will grow to the 1.5 times of the original size.  
Why 1.5x ?
Answer: Assume we don't know what the latter size should be, and we denote the latter size to be y. And we start the arrayList with size 1. Then we have  
1 y y^2  [when we reached y^2, we have GC collected 1 and y, so when continuous increasing needed, we can fill in the holes occurred in the front and will have ```1 + y = y^3 ==> y = 1.32```].  
Then, ```1 + y + ... + y^(n-2) = y^n``` gives us y = 1.618  
Cuz 1.618 is a float number, 1.5 is a better option.  
Wait? Isn't 1.5 also a float number? Let's see how we can get 1.5 in code:  
(1) len = len * 1.5  
(2) len = len * 3 / 2  
(3) len = len + (len >> 1) // 注意Java中位运算的优先级小于一般四则运算，所以这里要加括号！！  
Option 1 will give us a double result, Option 2, times 3 may cause integer overflow. So, Option 3 is the best alternative in which ```len >> 1``` gives us a half of original length and make final result 1.5 times the original in int value. This is also why we don't use 1.618 cuz we cannot use bit operation to form 1.618.  

#### Core code of Remove
```java
public E remove(int index) {
	...
	int numRemoved = size - index - 1;
	if (numRemoved > 0) {
		System.arraycopy(elementData, index+1, elementData, index, numMoved);
	}
	elementData[--size] = null; // clear to let GC do its work -- this step is necessary otherwise memory overflow..
	return oldValue;
}
```

### ArrayDeque

#### Frequently Used Methods
**addFirst(object)** - Amortized O(1)  
**addLast(object)** - Amortized O(1)  
**getFirst()** - O(1)  
**getLast()** - O(1)  
**removeFirst()** - O(1)  
**removeLast()** - O(1)  
--SetSomething() - ArrayDeque does NOT support set() or access(), if you want
to set some value or search some value in ArrayDeque, only you can do is linear
searching it costing O(N)   

#### How to circular
1. When there are empty spaces in the head, and we append to tail  
```tail = (tail + 1) % length``` OR  
```tail = (tail + 1) & (length - 1)```  
2. When there are empty spaces in the end, and we append to head  
```head = (head - 1 + length) % length``` OR  
```head = (head - 1) & (length - 1)```  
Memorize Tips: View tail and head as a scanner, ```scanner + 1``` means moving to the right and ```scanner - 1``` means moving to the left.


#### Resize
ArrayDeque's initial size is 16 (2^4).  
When ```tail == head```, we resize the originalCapacity to 2x. And set ```head``` to the index ```0``` of the arrayDeque, and set ```tail``` to index ```originalCapacity```  
Think: why 2x?  
Answer: somehow related to the Bitwise AND operation like ```head = (head - 1) & (length - 1)``` and ```tail = (tail + 1) & (length - 1)```. Here I think the reason is if 2x increase, we can always ensure ```length - 1``` is a number of ```2^n - 1``` which is always ```00...01...1``` in binary. The first 0 is + sign and all other 1s form ```2^n - 1``` to support ```&``` operation.  


### Performance Comparisons between ArrayList, ArrayDeque and LinkedList
| Performance   | Add First       | Add Last      | Remove First  | Remove Last | Set | Access |
| ------------- |----------------:| -------------:|--------------:|------------:|----:|-------:|
| ArrayList     |  O(N)           | Amortized O(1)| O(N)          |   O(1)      |O(1) | O(1)   |
| ArrayDeque    |  Amortized O(1) | Amortized O(1)| O(1)          |   O(1)      |N/A  | N/A    |
| LinkedList    |  O(1)           | O(1)          | O(1)          |   O(1)      |O(N) | O(N)   |

Note:  
For ArrayList,  
Add First: ```list.add(0, el);```  
Add Last: ```list.add(el);```  
Remove First: ```list.remove(0, el);```  
Remove Last: ```list.remove(el);```  

Generally speaking, if we need faster head operation, we pick ArrayDeque; if we don't need to use fast head operation using ArrayList is fine.  
And, ArrayList is more efficient in accessing and setting an intermediate element (index based) while ArrayDeque do Not have fast access/set on an intermediate element!  

For LinkedList and ArrayDeque, when doing Set and Access, you always need to find the element first, and the finding costs O(N)  


### LinkedList
Java's own LinkedList is a doubly-linked list.  
```java
public class LinkedList<E> extends ... {
	Node<E> first;
	Node<E> last;
}
```
```java
private static class Node<E> {
	E item;
	Node<E> next;
	Node<E> prev;

	Node(Node<E> prev, E element, Node<E> next) {
		this.item = element;
		this.next = next;
		this.prev = prev;
	}
}
```

#### Why the Node class is static?
Node class is nested within LinkedList class in Java Source Code. Nested "static class" has a special feature: anything within the nested static class cannot access anything outside this nested class but within its wrapper class; anything outside this nested static class but within its wrapper class can access members (even private) within the nested static class.  | "我"不能看外面但外面可以看我使用我

#### Size (if 64-bit machine)
| 8 bytes | x bytes | 8 bytes |
|:-------:|:-------:|:-------:|
| prev    | Element | next    |  

Here, both prev and next are memory addresses.  
