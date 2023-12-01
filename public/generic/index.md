# Java特性-泛型

  
**泛型，即参数化类型。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。**
**那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（类型形参），然后在使用/调用时传入具体的类型（类型实参）。**

*Java 语言中引入泛型是一个较大的功能增强。不仅语言、类型系统和编译器有了较大的变化，而且类库也进行了大翻修，所以许多重要的类，比如集合框架，都已经成为泛型化的了。这带来了很多好处：*
* 类型安全。 泛型的主要目标是提高 Java 程序的类型安全。通过知道使用泛型定义的变量的类型限制，编译器可以在一个高得多的程度上验证类型假设。
* 消除强制类型转换。 泛型的一个附带好处是，消除源代码中的许多强制类型转换。这使得代码更加可读，并且减少了出错机会。
* 潜在的性能收益。 泛型为较大的优化带来可能。在泛型的初始实现中，编译器将强制类型转换（没有泛型的话，程序员会指定这些强制类型转换）插入生成的字节码中。
* 注意泛型的类型参数只能是类类型（包括自定义类），不能是简单类型。

### 常用命名类型参数
* K：键，比如映射的键
* V：值，比如 List 和 Set 的内容，或者 Map 中的值
* E：元素
* T：泛型
* ?：表示不确定的 java 类型


### 通配符
Ingeter 是 Number 的一个子类，同时 Generic<Ingeter> 与 Generic<Number> 实际上是相同的一种基本类型。那么问题来了，在使用 Generic<Number> 作为形参的方法中，能否使用Generic<Ingeter> 的实例传入呢？在逻辑上类似于 Generic<Number> 和 Generic<Ingeter> 是否可以看成具有父子关系的泛型类型呢？下面我们通过定义一个方法来验证。
```java
    public void show(Generic<Number> obj) {
        System.out.println("key value is " + obj.getKey());
    }
```
进行如下的调用：
```java
    Generic<Integer> genericInteger = new Generic<Integer>(123);

    show(genericInteger);  //error Generic<java.lang.Integer>  cannot be applied to Generic<java.lang.Number>
```
通过提示信息我们可以看到 Generic<Integer> 不能被看作为 Generic<Number> 的子类。由此可以看出：同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

我们不能因此定义一个 show(Generic<Integer> obj)来处理，因此我们需要一个在逻辑上可以表示同时是Generic和Generic父类的引用类型。由此类型通配符应运而生。

T、K、V、E 等泛型字母为有类型，类型参数赋予具体的值。除了有类型，还可以用通配符来表述类型，？ 未知类型，类型参数赋予不确定值，任意类型只能用在声明类型、方法参数上，不能用在定义泛型类上。将方法改写成如下：
```java
    public void show(Generic<?> obj) {
        System.out.println("key value is " + obj.getKey());
    }
```
此处 ? 是类型实参，而不是类型形参。即和 Number、String、Integer 一样都是实际的类型，可以把 ？ 看成所有类型的父类，是一种真实的类型。可以解决当具体类型不确定的时候，这个通配符就是 ?；当操作类型时，不需要使用类型的具体功能时，只使用 Object 类中的功能。那么可以用 ? 通配符来表未知类型。

### 泛型上下边界
* 通配符上限为：Generic<? extends Number>
* 通配符下限为：Generic<? super Number>

在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。为泛型添加上边界，即传入的类型实参必须是指定类型的子类型。
```java
    public void show(Generic<? extends Number> obj) {
        System.out.println("key value is " + obj.getKey());
    }
```
我们在泛型方法的入参限定参数类型为 Number 的子类。
```java
    Generic<String> genericString = new Generic<String>("11111");
    Generic<Integer> genericInteger = new Generic<Integer>(2222);
  
    
    showKeyValue1(genericString); // error
    showKeyValue1(genericInteger);
```

当我们的入参为 String 类型时，编译报错，因为 String 类型并不是 Number 类型的子类。

类型通配符上限通过形如 Generic<? extends Number> 形式定义；相对应的，类型通配符下限为Generic<? super Number>形式，其含义与类型通配符上限正好相反，在此不作过多阐述。

### 一个泛型的增删改查Service
```java
    @Transactional(readOnly = true)
    public abstract class CrudService<D extends CrudDao<T>, T extends DataEntity<T>> extends BaseService {
        /**
        * 持久层对象
        */
        @Autowired
        protected D dao;
        
        /**
        * 获取单条数据
        * @param entity
        * @return
        */
        public T get(T entity) {
            return dao.get(entity);
        }
        
        /**
        * 查询列表数据
        * @param entity
        * @return
        */
        public List<T> findList(T entity) {
            return dao.findList(entity);
        }
        
        /**
        * 查询分页数据
        * @param page 分页对象
        * @param entity
        * @return
        */
        public Page<T> findPage(Page<T> page, T entity) {
            entity.setPage(page);
            page.setList(dao.findList(entity));
            return page;
        }

        /**
        * 保存数据（插入或更新）
        * @param entity
        */
        @Transactional(readOnly = false)
        public int save(T entity) {
            if (entity.getIsNewRecord()){
                entity.preInsert();
                return dao.insert(entity);
            }else{
                entity.preUpdate();
                return dao.update(entity);
            }
        }
        
        /**
        * 删除数据
        * @param entity
        */
        @Transactional(readOnly = false)
        public int delete(T entity) {
            return dao.delete(entity);
        }
    }
```
