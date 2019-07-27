本文的分析，基于jdk1.8的源码
TreeMap实现了红黑树
红黑树的五个性质：  
1、一个节点的颜色只能是红色或者黑色  
2、根节点必须是黑色  
3、叶子节点是空的黑色节点 
4、红色节点的两个自及诶单都是黑色的，也就是不能有两个相邻的红色节点  
5、一个节点到一个叶子节点，中间的多条路径包含个黑色节点个数是相同的。 
插入操作函数分析
```
public V put(K key, V value) {
    //暂时保存root的值。判断，如果根节点为空，就新建一个元素，默认节点是黑色
    //如果根节点不为空，可以两种情况来实现。一种是通过传进来的Comparator比较key的值。
    //另一种是通过key的比较器来比较。具体实现思路基本一致。
    //先从根节点开始比较。如果key的值和比较节点的值相同，直接把value赋给节点，然后返回。
    //如果连key.value 小于root.value.接下来比较左子树的值。
    //如果key.value大于root.value，接下来比较key和右子树的值。
    //

    //暂时保存root的值。判断，如果根节点为空，就新建一个元素，默认节点是黑色
    Entry<K, V> t = root;

    if(root == null) {
        compare(key, key);
        root = new Entry(key, value, null);
        modCount++;
        size = 1;
        return null;
    }

    //如果根节点不为空，可以两种情况来实现。一种是通过传进来的Comparator比较key的值。
    int cmp;
    Entry<K, V> parent;
    Comparator<? super K> cpr = comparator;
    //初始化制定了comparator比较器
    if(cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if(cmp < 0) {
                t = t.left;
            } else if(cmp > 0) {
                t = t.right;
            }
            else {
                return t.setValue(value);
            }

        } while(t != null);
    }
    else {//另一种是通过key的比较器来比较。具体实现思路基本一致。
         if(key == null) {
            return NullPointerException;
         }
         @SupressWarnings("unchecked")
             Comparable<? super K> k = (Comparable<? super K>) key;
         do{
            parent = t;
            cmp = cpr.compare(key, t.key);
            if(cmp < 0) {
                t = t.left;
            }
            else if(cmp > 0) {
                t = t.right;
            }
            else {
                return t.setValue(value);
            }
         } while(t != null);
    }
    
    //先从根节点开始比较。如果key的值和比较节点的值相同，直接把value赋给节点，然后返回。
    //如果连key.value 小于root.value.接下来比较左子树的值。
    //如果key.value大于root.value，接下来比较key和右子树的值。

    //把数据插入
    Entry<K, V> e = new Entry<>(key, value, parent);
    if(cmp < 0) {
        parent.left = e;
    }
    else {
        parent.right = e;
    }

    //新插入元素会破坏红黑树规则，需要调整
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;

    
 }
```
调整函数是实现的重难点
