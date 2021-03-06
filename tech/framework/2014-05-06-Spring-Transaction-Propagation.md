<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. Spring的事务传播机制详解</a>
<ul>
<li><a href="#sec-1-1">1.1. 1. 事务的传播属性（Propagation）</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1. 1) REQUIRED ，这个是默认的属性</a></li>
<li><a href="#sec-1-1-2">1.1.2. 2) MANDATORY</a></li>
<li><a href="#sec-1-1-3">1.1.3. 3) NEVER</a></li>
<li><a href="#sec-1-1-4">1.1.4. 4) NOT<sub>SUPPORTED</sub></a></li>
<li><a href="#sec-1-1-5">1.1.5. 5) REQUIRES<sub>NEW</sub></a></li>
<li><a href="#sec-1-1-6">1.1.6. 6) SUPPORTS</a></li>
<li><a href="#sec-1-1-7">1.1.7. 7) NESTED</a></li>
<li><a href="#sec-1-1-8">1.1.8. 8) PROPAGATION<sub>NESTED</sub> 与PROPAGATION<sub>REQUIRES</sub><sub>NEW的区别</sub></a></li>
</ul>
</li>
<li><a href="#sec-1-2">1.2. 2 事务的隔离级别（Isolation Level）</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. 1) 首先说明一下事务并发引起的三种情况</a></li>
<li><a href="#sec-1-2-2">1.2.2. 2) DEFAULT （默认）</a></li>
<li><a href="#sec-1-2-3">1.2.3. 3) READ<sub>UNCOMMITTED</sub> （读未提交）</a></li>
<li><a href="#sec-1-2-4">1.2.4. 4) READ<sub>COMMITTED</sub> （读已提交）</a></li>
<li><a href="#sec-1-2-5">1.2.5. 5) REPEATABLE<sub>READ</sub> （可重复读）</a></li>
<li><a href="#sec-1-2-6">1.2.6. 6) SERIALIZABLE（串行化）</a></li>
<li><a href="#sec-1-2-7">1.2.7. 7) 隔离级别解决事务并行引起的问题</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# Spring的事务传播机制详解<a id="sec-1" name="sec-1"></a>

## 1. 事务的传播属性（Propagation）<a id="sec-1-1" name="sec-1-1"></a>

### 1) REQUIRED ，这个是默认的属性<a id="sec-1-1-1" name="sec-1-1-1"></a>

Support a current transaction, create a new one if none exists.
如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。
被设置成这个级别时，会为每一个被调用的方法创建一个逻辑事务域。如果前面的方法已经创建了事务，那么后面的方法支持当前的事务，如果当前没有事务会重新建立事务。
如图所示：

### 2) MANDATORY<a id="sec-1-1-2" name="sec-1-1-2"></a>

Support a current transaction, throw an exception if none exists.
支持当前事务，如果当前没有事务，就抛出异常。

### 3) NEVER<a id="sec-1-1-3" name="sec-1-1-3"></a>

Execute non-transactionally, throw an exception if a transaction exists.
以非事务方式执行，如果当前存在事务，则抛出异常。

### 4) NOT<sub>SUPPORTED</sub><a id="sec-1-1-4" name="sec-1-1-4"></a>

Execute non-transactionally, suspend the current transaction if one exists.
以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

### 5) REQUIRES<sub>NEW</sub><a id="sec-1-1-5" name="sec-1-1-5"></a>

Create a new transaction, suspend the current transaction if one exists.
新建事务，如果当前存在事务，把当前事务挂起。
如图所示：

### 6) SUPPORTS<a id="sec-1-1-6" name="sec-1-1-6"></a>

Support a current transaction, execute non-transactionally if none exists.
支持当前事务，如果当前没有事务，就以非事务方式执行。

### 7) NESTED<a id="sec-1-1-7" name="sec-1-1-7"></a>

Execute within a nested transaction if a current transaction exists, behave like PROPAGATION<sub>REQUIRED</sub> else.
支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。
嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。

### 8) PROPAGATION<sub>NESTED</sub> 与PROPAGATION<sub>REQUIRES</sub><sub>NEW的区别</sub><a id="sec-1-1-8" name="sec-1-1-8"></a>

它们非常 类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。
使用PROPAGATION<sub>REQUIRES</sub><sub>NEW时，内层事务与外层事</sub> 务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。
两个事务互不影响。两个事务不是一个真正的嵌套事务。同时它需要JTA 事务管理器的支持。
使用PROPAGATION<sub>NESTED时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，它是一个真正的嵌套事务。</sub>

## 2 事务的隔离级别（Isolation Level）<a id="sec-1-2" name="sec-1-2"></a>

### 1) 首先说明一下事务并发引起的三种情况<a id="sec-1-2-1" name="sec-1-2-1"></a>

1.  i. Dirty Reads 脏读

    一个事务正在对数据进行更新操作，但是更新还未提交，另一个事务这时也来操作这组数据，并且读取了前一个事务还未提交的数据，而前一个事务如果操作失败进行了回滚，后一个事务读取的就是错误数据，这样就造成了脏读。

2.  ii. Non-Repeatable Reads 不可重复读

    一个事务多次读取同一数据，在该事务还未结束时，另一个事务也对该数据进行了操作，而且在第一个事务两次次读取之间，第二个事务对数据进行了更新，那么第一个事务前后两次读取到的数据是不同的，这样就造成了不可重复读。

3.  iii. Phantom Reads 幻像读

    第一个事务正在查询符合某一条件的数据，这时，另一个事务又插入了一条符合条件的数据，第一个事务在第二次查询符合同一条件的数据时，发现多了一条前一次查询时没有的数据，仿佛幻觉一样，这就是幻像读。

4.  iv. 非重复读和幻像读的区别

    非重复读是指同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生非重复读。
    (A transaction rereads data it has previously read and finds that another committed transaction has modified or deleted the data. )
    
    幻像读是指同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不 同的结果集，此时发生幻像读。
    (A transaction reexecutes a query returning a set of rows that satisfies a search condition and finds that another committed transaction has inserted additional rows that satisfy the condition. )
    
    表面上看，区别就在于非重复读能看见其他事务提交的修改和删除，而幻像能看见其他事务提交的插入。

### 2) DEFAULT （默认）<a id="sec-1-2-2" name="sec-1-2-2"></a>

这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应

### 3) READ<sub>UNCOMMITTED</sub> （读未提交）<a id="sec-1-2-3" name="sec-1-2-3"></a>

这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。

### 4) READ<sub>COMMITTED</sub> （读已提交）<a id="sec-1-2-4" name="sec-1-2-4"></a>

保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。

### 5) REPEATABLE<sub>READ</sub> （可重复读）<a id="sec-1-2-5" name="sec-1-2-5"></a>

这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了不可重复读

### 6) SERIALIZABLE（串行化）<a id="sec-1-2-6" name="sec-1-2-6"></a>

这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。

### 7) 隔离级别解决事务并行引起的问题<a id="sec-1-2-7" name="sec-1-2-7"></a>

Dirty reads        non-repeatable reads    phantom reads
Serializable                 不会                          不会                        不会
REPEATABLE READ       不会                          不会                          会
READ COMMITTED        不会                           会                           会
Read Uncommitted       会                              会                           会
